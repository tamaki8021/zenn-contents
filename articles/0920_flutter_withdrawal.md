---
title: "Flutterアプリに退会機能を追加したい"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter", "firebase", "CloudFunctions", "node.js"]
published: true
---

# 概要

下記のようなツイートを拝見したので今作っているFlutter ✕ Firebaseのアプリに退会機能を追加しようということで実際に試してできたやり方や参考になった記事について書いていきたいと思います。

https://twitter.com/_mono/status/1404977392641200128

こちらの記事の手順を参考に実装した方法をまとめました。

https://qiita.com/takashikatt/items/e17401ff301b71e821cd


# はじめに

退会機能の処理としては下記の3つの処理をすることで実装していきます。

- Authenticationで管理しているユーザーの情報を削除
- そのユーザーと紐付いているFirestore(DB)に保存しているユーザデータも削除
- どのユーザーが削除されたのかという履歴を残す

Firebaseでは上記の実装の方法が２つあります。

- クライアント側から削除する
- Cloud Functionsで削時間が

しかしクライアント側での削除は前回のアカウント連携から時間が経過しているとエラーになるため今回は**Cloud Functions**の方でやっていきたいと思います。

[参考](https://zenn.dev/hirokt/scraps/830b9c8d2caf78)

# 退会機能実装の流れ

1. 削除するユーザーのUIDをFirestoreに登録
2. Firestoreに登録時をトリガーとしてCloud Functionsを経由してAuthenticationのユーザーを削除する
3. Authenticationのユーザー削除をトリガーとしてCloud Functions経由でFirestoreに登録されているユーザーのデータを削除する

# 1. 削除するユーザーのUIDをFirestoreに登録

**deleted_users**というコレクションにデータを登録していきます。

:::message
今回は登録するデータのフィールドとしては ```uid``` と ```createdAt``` しか登録していませんが、フィールドの中身はご自身のプロダクトにあったものに変えていただけても大丈です。
:::

```dart
ElevatedButton(
            onPressed: () async {
              final data = {
                "uid": user.uid,
                "createdAt": Timestamp.now(),
              };
              await FirebaseFirestore.instance
                  .collection('delete_users')
                  .add(data)
                  .then((value) async => {
                        await FirebaseAuth.instance.signOut(),
                        Navigator.of(context).pushReplacement(
                            MaterialPageRoute(builder: (context) {
                          return LoginPage();
                        }))
                      })
                  .catchError((e) => print("Failed to add user: $e"));
            },
            child: Text('退会する'),
          )
```

# 2.Firestoreに登録時をトリガーとしてCloud Functionsを経由してAuthenticationのユーザーを削除する

Cloud Functionsの方にAuthenticationのユーザーを削除する関数をデプロイしていきます。
僕自身今回始めてCloud Functionsを使うということで[この記事](https://www.boel.co.jp/tips/vol124/)を参考に進めました。

[こちらの記事](https://qiita.com/tdkn/items/2ed2b01f2656fc50da8c)も参考にさせてもらいました。

コードの内容としてはとても単純でドキュメント追加時のonCreateの実行時にAuthenticationのユーザーを削除する感じです。

```js:index.js
import functions = require("firebase-functions");
import admin = require("firebase-admin");

// firebase-adminの初期化
admin.initializeApp();

exports.deleteUser = functions
    .region("asia-northeast1")
    .firestore.document("deleted_users/{docId}")
    .onCreate(async (snap, context) => {
      const deleteDocument = snap.data();
      const uid = deleteDocument.uid;

      // Authenticationのユーザーを削除する
      await admin.auth().deleteUser(uid);
    });
```

# 3.Authenticationのユーザー削除をトリガーとしてCloud Functions経由でFirestoreに登録されているユーザーのデータを削除する

Firestoreに保存されているデータを削除するのにはFirebase側が提供している **Delete User Data Extensions**を使います。

https://firebase.google.com/products/extensions/delete-user-data

コンソールでインストールするをクリック後、入力項目があるのでご自身のプロダクトを見てUIDに紐づくユーザーコレクションのパスを入力するなどして登録します。

画像は登録後の画面です。参考程度に、、
![](https://storage.googleapis.com/zenn-user-upload/4ca48dc15fe61da2a2c2dcf3.png)

# まとめ
Firebase Authentication、Firestoreのユーザデータを削除する退会機能をまとめてみました。
最初はCloud functionsを使ったことがなかったので不安だったんですけど、案外あっという間に実装することができたので、是非試してみてください。

またこの実装で上手くいかなかったり、間違っている箇所があればコメント等で教えて下さい!

# 参考

https://qiita.com/takashikatt/items/e17401ff301b71e821cd

https://firebase.google.com/docs/functions/get-started?hl=ja

https://firebase.google.com/products/extensions/delete-user-data