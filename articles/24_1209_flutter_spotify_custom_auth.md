---
title: "【Flutter】Spotify API × Firebase Authでログイン機能を実装する方法"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  [
    "flutter",
    "spotify",
    "firebase",
    "firebaseAuth",
    "cloudFunctions",
  ]
published: true
---

# 概要

この記事では、Spotify API を用いた OAuth 2.0 認証と Firebase Authentication の連携方法を学びます。Spotify API を通じてユーザー情報を取得し、Firebase を用いて認証状態を管理します。

## 使用する技術スタック

- フロントエンド: Flutter
- バックエンド: Firebase Cloud Functions
- 認証基盤: Firebase Authentication
- 外部 API: Spotify API

# 前提情報

## Firebase Authentication のカスタムトークン認証について

Firebase Authentication は様々な認証プロバイダをネイティブにサポートしています。下記はその一例です。Firebase コンソールの Authentication > Sign-in method からその一覧を確認することができます。

- Email/Password
- Anonymous
- Google
- Apple

Firebase Authentication のカスタムトークン認証は、Firebase Authentication がネイティブにサポートしていない認証プロバイダと Firebase Authentication を連携するために使用することができます。

https://firebase.google.com/docs/auth/admin/create-custom-tokens?hl=ja

# 実装

## 1. Spotify Developer アカウントのセットアップ

1. Spotify Developer Dashboard にアクセスし、新しいアプリケーションを作成します。
   [Spotify Developer Dashboard](https://developer.spotify.com/dashboard/)
2. クライアント ID とクライアントシークレットをコピーしてメモしておきます。
3. Redirect URI を設定します。
   ※ フロントエンドが fllutter を使用するので下記のように自身のアプリにて設定
   例:<com.example.project_name>://callback

## 2. Firebase プロジェクトの設定

1. Firebase Console で新しいプロジェクトを作成します。
2. Firebase Authentication を有効化します。
3. Cloud Functions を有効化します。

## 3. Cloud Functions の実装

Spotify のアクセストークンと Firebase カスタムトークンの生成を管理する Cloud Function を実装します。

```js: functions/src/index.js
import * as admin from "firebase-admin";

import { onRequest } from "firebase-functions/v2/https";
import logger = require("firebase-functions/logger");

const CLIENT_ID = "<your-client-id>";

export const spotifySignIn = onRequest(async (req, res) => {
  const { code, redirectUri, codeVerifier } = req.body.data;

  try {
    const payload = {
      method: "POST",
      headers: {
        "Content-Type": "application/x-www-form-urlencoded",
      },
      body: new URLSearchParams({
        client_id: CLIENT_ID,
        grant_type: "authorization_code",
        code,
        redirect_uri: redirectUri,
        code_verifier: codeVerifier,
      }),
    };

    // Spotify APIでアクセストークンを取得
    fetch("https://accounts.spotify.com/api/token", payload)
      .then(async (response) => {
        if (!response.ok) {
          logger.error(`HTTP error! status: ${response.status}`);
          res.send(`HTTP error! status: ${response.status}`)
        }
        const data = await response.json();
        const accessToken = data.access_token;
        logger.log("Received Access Token:", accessToken);
        logger.log("Received Refresh Token:", data.refresh_token);

        // アクセストークンを使ってSpotify APIにリクエスト
        const userInfoResponse = await fetch("https://api.spotify.com/v1/me", {
          headers: { Authorization: `Bearer ${accessToken}` },
        });
        if (!userInfoResponse.ok) {
          logger.error(`HTTP error! status: ${userInfoResponse.status}`);
          res.send(`HTTP error! status: ${userInfoResponse.status}`)
        }
        const userResults = await userInfoResponse.json();
        logger.log("Auth code exchange result received:", userResults);
        const spotifyUserID = userResults["id"];
        const profilePic = userResults["images"][0]["url"];
        const userName = userResults["display_name"];
        const email = userResults["email"];

        // Firebaseアカウントの作成とFirebaseカスタムトークンの生成
        const firebaseToken = await createFirebaseAccount(
          spotifyUserID,
          userName,
          profilePic,
          email,
          accessToken
        );
        res.jsonp({ data: firebaseToken });
      })
      .catch((err) => {
        logger.error("Error fetching access token:", err);
      });
  } catch (error: any) {
    logger.error("Error during Spotify token exchange:", error);
    res.status(500).jsonp({ data: error.toString() });
  }
  return;
});


// Firebaseアカウントの作成とFirebaseカスタムトークンの生成
const createFirebaseAccount = async (
  spotifyID: string,
  displayName: string,
  photoURL: string,
  email: string,
  accessToken: string
) => {
  const uid = `spotify:${spotifyID}`;

  // Firebase Realtime Databaseにアクセストークンを保存する.
  const databaseTask = admin
    .database()
    .ref(`/spotifyAccessToken/${uid}`)
    .set(accessToken);

  // ユーザーアカウントを作成または更新する
  const userCreationTask = admin
    .auth()
    .updateUser(uid, {
      displayName: displayName,
      photoURL: photoURL,
      email: email,
      emailVerified: true,
    })
    .catch((error) => {
      // もしユーザーが存在しなかった場合作成する
      if (error.code === "auth/user-not-found") {
        return admin.auth().createUser({
          uid: uid,
          displayName: displayName,
          photoURL: photoURL,
          email: email,
          emailVerified: true,
        });
      }
      throw error;
    });

  await Promise.all([userCreationTask, databaseTask]);
  // Firebase custom auth tokenを作成する.
  const token = await admin.auth().createCustomToken(uid);
  logger.log('Created Custom token for UID "', uid, '" Token:', token);
  return token;
};

```

## 4. Flutter アプリで認証フローを構築

Flutter アプリで Spotify のログイン画面を表示し、認証コードを取得します。
※flutter と firebase の接続設定はできている前提とします。

### 必要なライブラリ

以下のライブラリをインストールします。

```
flutter pub add crypto
flutter pub add firebase_auth
flutter pub add flutter_web_auth
flutter pub add cloud_functions
```

### 認証フローのコード

```dart: sign_in_page.dart
class SignInPage extends StatelessWidget {
  const SignInPage({super.key});

  @override
  Widget build(BuildContext context) {
    const clientId = 'your_client_id';
    const redirectUri = '<com.example.project_name>://callback';
    const scopes = 'user-read-private user-read-email';

    Future<void> handleSignIn() async {

      /// PKCE 認証フローは、コード検証子の作成から始まります。PKCE標準によると、コード検証子は、長さが 43 ～ 128 文字 (長いほど良い) の高エントロピー暗号化ランダム文字列です。文字、数字、アンダースコア、ピリオド、ハイフン、チルダを含めることができます。
      String generateRandomString(int length) {
        const possible ='ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
        final random = Random.secure();
        final values = List<int>.generate(length, (i) => random.nextInt(256));
        return values.map((x) => possible[x % possible.length]).join();
      }

      ///  SHA256 アルゴリズムを使用して変換 (ハッシュ) する
      Future<String> generateCodeChallenge(String plain) async {
        final bytes = utf8.encode(plain);
        final digest = sha256.convert(bytes);
        return base64UrlEncode(digest.bytes)
            .replaceAll('=', '')
            .replaceAll('+', '-')
            .replaceAll('/', '_');
      }

      try {
        final codeVerifier = generateRandomString(64);
        final codeChallenge = await generateCodeChallenge(codeVerifier);

        final authUrl = Uri.https('accounts.spotify.com', '/authorize', {
          'client_id': clientId,
          'response_type': 'code',
          'redirect_uri': redirectUri,
          'scope': scopes,
          'code_challenge_method': 'S256',
          'code_challenge': codeChallenge,
        });

        /// Spotify認証サーバーのログインページにリダイレクトします。
        final result = await FlutterWebAuth.authenticate(
          url: authUrl.toString(),
          callbackUrlScheme: '<com.example.project_name>',
        );

        /// URLを解析してcodeパラメータを取得する
        final code = Uri.parse(result).queryParameters['code'];
        print('Authorization code: $code');

        /// cloud_functionsの関数の呼び出し
        final resultSpotifySignIn = await FirebaseFunctions.instance
            .httpsCallable('spotifyAuth')
            .call<String>({
          'code': code,
          'codeVerifier': codeVerifier,
          'redirectUri': redirectUri,
        });
        final firebaseToken = resultSpotifySignIn.data;
        await FirebaseAuth.instance.signInWithCustomToken(firebaseToken);

          // サインインが成功した場合のリダイレクト処理
          ...
      } on Exception catch (_) {
        openSnackBar(context: context, message: 'サインインに失敗しました');
      }
    }

    /// UIの記述
    return (
      /// ....
    );
  }
```

# まとめ

今回は、Spotify API を用いた OAuth 認証を Flutter で実装し、Firebase Authentication を利用してログインを管理する方法を紹介しました。

参考リンク:
https://developer.spotify.com/documentation/web-api/
https://github.com/firebase/functions-samples/tree/main/Node-1st-gen/spotify-auth