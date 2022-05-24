---
title: "expoでfirebase-authのgoogleログインを実装する"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["expo", "ReactNative", "firebase", "google", "firebaseAuth"]
published: false
---

# 初めに

expo のドキュメントには色々書いてあるんですけど、検索したりしても古い情報も混じってたりで、実装に苦労したの firebase-auth を使った google 認証の実装をやっていきたいと思います。

事前に下記 2 点の環境はできている前提で進みたいと思います。

- expo が利用できる環境である
- firebase のプロジェクトを作成済みである

# firebase との接続

[ここはドキュメントの手順に従って行います。](https://docs.expo.dev/guides/using-firebase/#firebase-sdk-setup)

firebase が提供する JavaScript SDK を使用するので firebase パッケージをインストールします。

```
$ expo install firebase
```

firebase.ts(js)ファイルを作成して firebase の管理画面上の Firebase SDK の追加で表示されていた接続情報をコピー&ペースとします。

```ts:firebase.ts
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';

// Optionally import the services that you want to use
//import {...} from "firebase/database";
//import {...} from "firebase/firestore";
//import {...} from "firebase/functions";
//import {...} from "firebase/storage";

// Initialize Firebase
const firebaseConfig = {
  apiKey: 'api-key',
  authDomain: 'project-id.firebaseapp.com',
  databaseURL: 'https://project-id.firebaseio.com',
  projectId: 'project-id',
  storageBucket: 'project-id.appspot.com',
  messagingSenderId: 'sender-id',
  appId: 'app-id',
  measurementId: 'G-measurement-id',
};

export const firebaseApp = initializeApp(firebaseConfig);
export const firebaseAuth = getAuth(firebaseApp)
```

# google ログイン用のパッケージの追加・設定

最初は色々な記事でも使われていた[expo-google-sign-in](https://github.com/expo/expo-google-sign-in)というパッケージを使おうとしたのですが、非推奨になっていました）

なので[@react-native-google-signin/google-signin](https://github.com/react-native-google-signin/google-signin#expo-installation)のパッケージを使って実装していきます。

> このパッケージは"Expo Go"アプリでは使用できないのでまだカスタムネイティブコードを追加していない方はカスタムネイティブコードの追加が必要となります。

## カスタムネイティブコードの追加（まだ行っていない方）

追加は簡単で下記のコマンドを追加したい環境ごとに実行するだけです。
[カスタムネイティブコードの追加](https://docs.expo.dev/workflow/customizing/)

:::message
iOS は Xcode ののインストールが必要であり、Android は Android Studio と Android SDK のインストールが必要になります。
:::

```
# Android
$ expo run:android

# iOS
$ expo run:ios
```

## インストール

```
$ expo install @react-native-google-signin/google-signin
```

## iOS/Android のセットアップ

ドキュメントに従って使う環境別にセットアップを行います。
[iOS の手順](https://github.com/react-native-google-signin/google-signin/blob/master/docs/ios-guide.md)
[Android の手順](https://github.com/react-native-google-signin/google-signin/blob/master/docs/android-guide.md)

## config の設定

`app.json`または`app.config.js`に下記の内容を追加

:::message
`app.config.js`ファイルはデフォルトで作成される`app.json`ファイルの拡張用に作成されるファイルというイメージです。
:::

```json:app.json
{
  "expo": {
    "android": {
      "googleServicesFile": "./google-services.json"
    },
    "ios": {
      "googleServicesFile": "./GoogleService-Info.plist"
    },
    "plugins": ["@react-native-google-signin/google-signin"]
  }
}
```

# ログイン画面の作成

```tsx: loginScreen.tsx
import React from 'react';
import {
  View,
  TextInput,
  Text,
  TouchableOpacity,
  KeyboardAvoidingView,
} from 'react-native';
import { useAuthContext } from '../hooks/auth';

const LoginScreen = () => {
  const { googleSignIn } = useAuthContext();

  return (
     <SafeAreaView>
      <Text>ログイン画面</Text>
      <Button onPress={googleSignIn} title={'googleログイン'} />
    </SafeAreaView>
  );
};

export default LoginScreen;
```

## カスタムフックとしてログイン部分を実装

(createContext)[https://ja.reactjs.org/docs/context.html]を使って state を管理していきます。

```tsx:src/hooks/auth.tsx
import { GoogleSignin } from '@react-native-google-signin/google-signin';
import {
  GoogleAuthProvider,
  onAuthStateChanged,
  signInWithCredential,
  User,
  signOut as firebaseSignOut,
} from 'firebase/auth';
import { createContext, useContext, useEffect, useState } from 'react';
import { firebaseAuth } from '../firebase';

type Auth = {
  user: User | null;
  loading: boolean;
  googleSignIn: () => Promise<boolean>;
};

const AuthContext = createContext<Auth>({} as Auth);

export const useAuthContext = () => {
  return useContext(AuthContext);
};

const useAuthProvider = () => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // googlesigninを行う場合に必須で呼び出すもの
    GoogleSignin.configure({ webClientId: /* GoogleService-Info.plist内にあるCLIENT_ID> */  });
  }, []);

  // Google の認証応答からの ID トークンを Firebase 認証情報と交換し、それを使用して Firebase での認証を行う
  const handleCredentialResponse = async (googleIdToken: string) => {
    const credential = GoogleAuthProvider.credential(googleIdToken);

    const result = await signInWithCredential(firebaseAuth, credential);
    if (result) {
      setUser(currentUser);
    }
  };

  const googleSignIn = async (): Promise<boolean> => {
    setLoading(true);
    try {
      await GoogleSignin.hasPlayServices();
      const userInfo = await GoogleSignin.signIn();
      if (userInfo) {
        await handleCredentialResponse(userInfo.idToken!);
      }

      setLoading(false);
      return true;
    } catch {
      setLoading(false);
      return false;
    }
  };

  return {
    user,
    loading,
    googleSignIn,
  };
}

export const AuthProvider: React.FC = ({ children }) => {
  const authContext = useAuthProvider();

  return <AuthContext.Provider value={authContext}>{children}</AuthContext.Provider>;
};
```

AuthProvider で wrap する

```tsx: App.tsx
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { AuthProvider } from './src/hooks/auth';

export default function App() {
  return (
    <NavigationContainer>
      <AuthProvider>
        <RootStack>
      </AuthProvider>
    </NavigationContainer>
  );
}
```

# ログイン状態の監視

前述で作ったカスタムフックスに onAuthStateChanged を使ってログインしている場合に setUser を利用してユーザ情報を保存します。ログインしていない場合には user 情報を空にします。

```tsx:src/hooks/auth.tsx
...
  const useAuthProvider = () => {
  ...

  const handleRedirect = async () => {
    // google側にログインしているユーザーの情報を取得する
    const userInfo = await GoogleSignin.signInSilently();

    if (userInfo) {
      await handleCredentialResponse(userInfo.idToken!);
    }
  };

  useEffect(() => {
    handleRedirect();

    const unsubscribe = onAuthStateChanged(firebaseAuth, (currentUser) => {
      if (currentUser) {
        onSuccess(currentUser);
      } else {
        setUser(null);
      }
      setLoading(false);
      unsubscribe();
    });
  }, [setUser]);
}
...
```

カスタムフックスの`auth.tsx`の完全版

```tsx:src/hooks/auth.tsx
import { GoogleSignin } from '@react-native-google-signin/google-signin';
import {
  GoogleAuthProvider,
  onAuthStateChanged,
  signInWithCredential,
  User,
  signOut as firebaseSignOut,
} from 'firebase/auth';
import { createContext, useContext, useEffect, useState } from 'react';
import { firebaseAuth } from '../firebase';

type Auth = {
  user: User | null;
  loading: boolean;
  googleSignIn: () => Promise<boolean>;
};

const AuthContext = createContext<Auth>({} as Auth);

export const useAuthContext = () => {
  return useContext(AuthContext);
};

const useAuthProvider = () => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // googlesigninを行う場合に必須で呼び出すもの
    GoogleSignin.configure({ webClientId: /* GoogleService-Info.plist内にあるCLIENT_ID> */  });
  }, []);

  // Google の認証応答からの ID トークンを Firebase 認証情報と交換し、それを使用して Firebase での認証を行う
  const handleCredentialResponse = async (googleIdToken: string) => {
    const credential = GoogleAuthProvider.credential(googleIdToken);

    const result = await signInWithCredential(firebaseAuth, credential);
    if (result) {
      setUser(currentUser);
    }
  };

  const handleRedirect = async () => {
    // google側にログインしているユーザーの情報を取得する
    const userInfo = await GoogleSignin.signInSilently();

    if (userInfo) {
      await handleCredentialResponse(userInfo.idToken!);
    }
  };

  useEffect(() => {
    handleRedirect();

    const unsubscribe = onAuthStateChanged(firebaseAuth, (currentUser) => {
        if (currentUser) {
          onSuccess(currentUser);
        } else {
          setUser(null);
        }
        setLoading(false);
        unsubscribe();
      });
    }, [setUser]);
  }

  const googleSignIn = async (): Promise<boolean> => {
    setLoading(true);
    try {
      await GoogleSignin.hasPlayServices();
      const userInfo = await GoogleSignin.signIn();
      if (userInfo) {
        await handleCredentialResponse(userInfo.idToken!);
      }

      setLoading(false);
      return true;
    } catch {
      setLoading(false);
      return false;
    }
  };

  return {
    user,
    loading,
    googleSignIn,
  };
  }

export const AuthProvider: React.FC = ({ children }) => {
  const authContext = useAuthProvider();

  return <AuthContext.Provider value={authContext}>{children}</AuthContext.Provider>;
};
```

user にユーザ情報が保存されているかどうかの条件分岐を利用してログインしている場合は Home 画面、ログインしていない場合はログイン画面が表示されるようにします。

```tsx: RootStack.tsx
import React, { useEffect } from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import HomeScreen from './screens/HomeScreen';
import RegisterScreen from './screens/RegisterScreen';
import { onAuthStateChanged } from 'firebase/auth';
import { auth } from './firebase';
import { useAuthContext } from '../hooks/auth';

const Stack = createNativeStackNavigator();

export const RootStack = () => {
  const { user } = useAuthContext();

  return (
    <Stack.Navigator>
      {user ? (
        <Stack.Screen name="Home" component={HomeScreen} />
      ) : (
        <Stack.Screen name="login" component={LoginScreen} />
      )}
    </Stack.Navigator>
  );
}
```

# 最後に
プロジェクトで実装したコードの一部を抜粋して紹介しているので、一部typoであったり、うまく動かないところがあるかもしれませんが、少しでも参考になれたらありがたいです。

# 参考にした記事

https://docs.expo.dev/versions/latest/sdk/google-sign-in/

https://github.com/react-native-google-signin/google-signin#expo-installation

https://ja.reactjs.org/docs/context.html