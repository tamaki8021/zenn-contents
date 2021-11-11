---
title: "【Flutter】pushNamedで行う固定ボトムナビゲーションバーを作る"
emoji: "🕌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter", "BottomNavigationBar", "dart", "CupertinoTabBar"]
published: true
---

# 概要

固定ボトムナビゲーションバー作成の記事は色々と探したらあるんですが、複雑で簡単に実装したいけど、なんかよくわからん。。みたいなことがあったので、色々と試行錯誤した結果 CupertinoTabBar で結構簡単に思った通りの実装ができた記事を書いていこうかなと思います。

# 実装

```dart: main.dart
import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      routes: routes,
      home: const BottomNavigation(),
    );
  }
}
```

## routes

```dart: route.dart
final Map<String, WidgetBuilder> routes = {
  TopPage.path: (context) => const TopPage(),
  DammyPage.path: (context) => const DammyPage(),
  LoginPage.path: (contex) => const LoginPage(),
};
```

## ボトムナビゲーションバー

```dart: bottom_navigation.dart

import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';

import '/pages/top_page.dart';
import '/pages/login_page.dart';

import 'util/routes.dart';

class BottomNavigation extends StatelessWidget {
  const BottomNavigation({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return CupertinoTabScaffold(
      tabBar: CupertinoTabBar(
        items: const <BottomNavigationBarItem>[
          BottomNavigationBarItem(
            icon: Icon(Icons.home),
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.login),
          ),
        ],
      ),
      tabBuilder: (BuildContext context, int index) {
        switch (index) {
          case 0:
            return CupertinoTabView(
              routes: routes,
              builder: (context) {
                return const CupertinoPageScaffold(
                  child: TopPage(),
                );
              },
            );
          case 1:
            return CupertinoTabView(
              routes: routes,
              builder: (context) {
                return const CupertinoPageScaffold(
                  child: LoginPage(),
                );
              },
            );
          default:
            return CupertinoTabView(
              routes: routes,
              builder: (context) {
                return const CupertinoPageScaffold(
                  child: TopPage(),
                );
              },
            );
        }
      },
    );
  }
}
```

## 遷移方法

```dart
// ナビゲーションバーを残したまま遷移
onPressed: () => Navigator.of(context).pushName(DammyPage.path),

// ナビゲーションバーを残さない遷移
onPressed: () => Navigator.of(context, rootNavigator: true,).pushNamed(DammyPage.path),
```

```dart: top_page.dart
import 'package:flutter/material.dart';

class TopPage extends StatelessWidget {
  const TopPage({Key? key}) : super(key: key);

  static const path = '/top';

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      child: Center(
        child: TextButton(
          child: const Text('ナビゲーションバーを残したまま遷移'),
          onPressed: () => Navigator.of(context).pushNamed(DammyPage.path),
        ),
      ),
    );
  }
}
```
