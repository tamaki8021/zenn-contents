---
title: "【Flutter】AdMobインタースティシャル広告を表示する"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter", "AdMob", "interstitial", "hooks", "広告"]
published: true
---

# はじめに

Flutter のアプリ開発で広告を付けたいときに Adbanner 広告についての記事や公式ドキュメントではたくさんの情報があったのですが、インタースティシャル広告などの記事はなかなか見つけることができなかったので、後からの振り返り用の議事録的な感じこの記事を作成しました。

# 環境

```
$ flutter doctor
[✓] Flutter (Channel stable, 2.2.2, on macOS 11.4 20F71 darwin-x64, locale ja-JP)
[✓] Android toolchain - develop for Android devices (Android SDK version 30.0.3)
[✓] Xcode - develop for iOS and macOS
[✓] Chrome - develop for the web
[✓] Android Studio (version 4.2)
[✓] IntelliJ IDEA Ultimate Edition (version 2021.1.2)
[✓] VS Code (version 1.59.0)
[✓] Connected device (1 available)
```

パッケージは Flutter2 がリリースされて推奨になった[**googleads-mobile-flutter**](https://github.com/googleads/googleads-mobile-flutter)を使います

https://github.com/googleads/googleads-mobile-flutter

# セットアップ

いろいろな記事でも書かれてると思いますが、AdMob を ios,Android で使うためのセットアップを行います。
ios と Android の AdMobId はご自身の AdMob アカウントで確認してください。

https://support.google.com/admob/answer/7356431

## ios

ios/Runner/Info.plist へのコードの追加を行います。

```Info.plist
<key>GADApplicationIdentifier</key>
// ↓自分のAdMobのappIDに変更
<string>ca-app-pub-3940256099942544~1458002511</string>
<key>SKAdNetworkItems</key>
  <array>
    <dict>
      <key>SKAdNetworkIdentifier</key>
      <string>cstr6suwn9.skadnetwork</string>
    </dict>
  </array>
```

## Android

android/app/src/main/AndroidManifest.xml のファイルの<meta-data>タグへ下記のように追加する

```xml:AndroidManifest.xml
<manifest>
    <application>
        <!-- サンプル AdMob App ID: ca-app-pub-3940256099942544~3347511713 -->
        <meta-data
            android:name="com.google.android.gms.ads.APPLICATION_ID"
            android:value="ca-app-pub-xxxxxxxxxxxxxxxx~yyyyyyyyyy"/>
    </application>
</manifest>
```

# 初期化

広告を読み込む前に、MobileAds.instance.initialize（）を呼び出して Mobile Ads SDK を初期化し、アプリで初期化します

```dart:main.dart
import 'package:google_mobile_ads/google_mobile_ads.dart';
import 'package:flutter/material.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  MobileAds.instance.initialize();

  runApp(MyApp());
}
```

# インタースティシャル広告

## インタースティシャル広告用の管理用クラスを作る

```dart:AdInterstitial.dart
import 'dart:io';

import 'package:google_mobile_ads/google_mobile_ads.dart';

class AdInterstitial {
  InterstitialAd? _interstitialAd;
  int num_of_attempt_load = 0;
  bool? ready;

  // create interstitial ads
  void createAd() {
    InterstitialAd.load(
      adUnitId: interstitialAdUnitId,
      request: const AdRequest(),
      adLoadCallback: InterstitialAdLoadCallback(
        // 広告が正常にロードされたときに呼ばれます。
        onAdLoaded: (InterstitialAd ad) {
          print('add loaded');
          _interstitialAd = ad;
          num_of_attempt_load = 0;
          ready = true;
        },
        // 広告のロードが失敗した際に呼ばれます。
        onAdFailedToLoad: (LoadAdError error) {
          num_of_attempt_load++;
          _interstitialAd = null;
          if (num_of_attempt_load <= 2) {
            createAd();
          }
        },
      ),
    );
  }

  // show interstitial ads to user
  Future<void> showAd() async {
    ready = false;
    if (_interstitialAd == null) {
      print('Warning: attempt to show interstitial before loaded.');
      return;
    }
    _interstitialAd!.fullScreenContentCallback = FullScreenContentCallback(
      onAdShowedFullScreenContent: (InterstitialAd ad) {
        print("ad onAdshowedFullscreen");
      },
      onAdDismissedFullScreenContent: (InterstitialAd ad) {
        print("ad Disposed");
        ad.dispose();
      },
      onAdFailedToShowFullScreenContent: (InterstitialAd ad, AdError aderror) {
        print('$ad OnAdFailed $aderror');
        ad.dispose();
        createAd();
      },
    );

    // 広告の表示には.show()を使う
    await _interstitialAd!.show();
    _interstitialAd = null;
  }

  // 広告IDをプラットフォームに合わせて取得
  static String get interstitialAdUnitId {
    if (Platform.isAndroid) {
      return 'ca-app-pub-3940256099942544/6300978111';
    } else if (Platform.isIOS) {
      return 'ca-app-pub-3940256099942544/2934735716';
    } else {
      //どちらでもない場合は、テスト用を返す
      return BannerAd.testAdUnitId;
    }
  }
}
```

## インタースティシャル広告を表示する

```dart:main.dart
import 'package:flutter/material.dart';
import 'package:flutter_app_sample/AdInterstitial.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  AdmobHelper.initialization();
  runApp(MyApp());
}
class MyApp extends StatelessWidget  {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Flutter Admob ad example',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomepage(),
    );
  }
}
class MyHomepage extends StatefulWidget {
  @override
  _MyHomepageState createState() => _MyHomepageState();
}


class _MyHomepageState extends State<MyHomepage> {
  InterstitialAd? _interstitialAd;
  AdInterstitial adInterstitial = new AdInterstitial();

  @override
  void initState() {
    super.initState();
    adInterstitial.createAd();
  }

  @override
  void dispose() {
    super.dispose();
   _interstitialAd?.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      child: Text('show Interstitial Ad'),
      onPressed: () {
        adInterstitial.show();
      },
    );
  }
}
```

# StatefullWidget じゃないんだよな。。

私自身が flutter 初心者で State のサイクルよくわからないし、コードの量が多いから基本的には HookWidet を使っていて結局どう書けばいいのかわからなかったので、一応色々と試した結果できたので同じく HookWidget を使っている方がいたら参考程度に変更箇所のコード置いときます。

```diff dart:AdInterstitial.dart
  void createAd() {
    InterstitialAd.load(
      adUnitId: interstitialAdUnitId,
      request: const AdRequest(),
      adLoadCallback: InterstitialAdLoadCallback(
        // 広告が正常にロードされたときに呼ばれます。
        onAdLoaded: (InterstitialAd ad) {
          print('add loaded');
          _interstitialAd = ad;
          num_of_attempt_load = 0;
          ready = true;
+         showAd(); // showAdをcreateAdで呼び出す
        },
        // 広告のロードが失敗した際に呼ばれます。
        onAdFailedToLoad: (LoadAdError error) {
          num_of_attempt_load++;
          _interstitialAd = null;
          if (num_of_attempt_load <= 2) {
            createAd();
          }
        },
      ),
    );
  }
```

```diff dart:main.dart
class MyApp extends StatelessWidget  {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Flutter Admob ad example',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomepage(),
    );
  }
}

- class MyHomepage extends StatefulWidget {
-  @override
-  _MyHomepageState createState() => _MyHomepageState();
- }

- class _MyHomepageState extends State<MyHomepage> {
+ class MyHomepage extends HookWidget {
  InterstitialAd? _interstitialAd;
  AdInterstitial adInterstitial = new AdInterstitial();

- @override
- void initState() {
-   super.initState();
-   adInterstitial.createAd();
- }

- @override
- void dispose() {
-   super.dispose();
-  _interstitialAd?.dispose();
- }

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      child: Text('show Interstitial Ad'),
      onPressed: () {
-       adInterstitial.show();
+       adInterstitial.createAd();
      },
    );
  }
}
```

:::message
useEffectなどでcreateしてloadしてクッリクしたときにshowで表示という流れが理想的な気がしますが、どうやってもcreateが走ったときにloadがうまくいかずshowで弾かれてしまっていたので、createの中でshowを呼ぶという感じにしました。

もっといいやり方があればコメントなどで教えていただけるとありがたいです。
:::

# 参考になった記事・サイト

- **ちょっと記法が古いけど**

https://uedive.net/2021/5410/flutter2-gad/
https://qiita.com/yukihiroK/items/8e9f0ebfb83f39d08a31
https://codelabs.developers.google.com/codelabs/admob-ads-in-flutter?continue=https%3A%2F%2Fcodelabs.developers.google.com%2F%3Fcat%3Dads#8

- **英語の記事だけどコードを書くときにとても参考になったもの**

https://protocoderspoint.com/monetize-your-flutter-app-with-google-admob-ads/
https://github.com/googleads/googleads-mobile-flutter/blob/master/packages/google_mobile_ads/example/lib/main.dart

- **公式**

https://github.com/googleads/googleads-mobile-flutter
https://developers.google.cn/admob/flutter/interstitial?hl=ja