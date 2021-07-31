---
title: "【Flutter】flutter２週間学習でスマホアプリ開発未経験文系大学生が単位管理アプリをリリースしてみた話"
emoji: ":beer:"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["flutter","スマホアプリ","firebase","個人開発"]
published: false
---

個人開発で単位管理アプリ「UnitTracking」をリリースしました！

今回の記事ではサービスの簡単な紹介と開発からリリースまでのことをまとめていきたいと思います。
後で自分で振り返り用の備忘録的な意味もあるんですけど、個人開発している人だったり、なにか作りたいとは思っているんだけどなと感じている人の参考や何か役に立ったらいいなと思います。

# リリースしたアプリ
![UnitTracking](https://storage.googleapis.com/zenn-user-upload/fa5324b08a93fb3eae1ba2af.png)

ダウンロードはこちら:point_down:
[iOS]()
[Android](https://play.google.com/store/apps/details?id=net.unit_tracking.unit_tracking)


# アプリの主な機能
- 単位数を円グラフとバーグラフで表示できる
- 講義を検索できる
- 気になった講義をクリップできる

詳しくは[こちらのサイト](https://flash-dry-d44.notion.site/UnitTracking-c651599c1778403f84322af8ff24e734)でまとめました。
※iosへのリリースでサポートサイトが必要だったんで仕方なく作成したんですけど、、


# 開発した背景
作り始めたききっかけは、初めて参加したハッカソン([サポータズの技育CAMP](https://talent.supporterz.jp/geekcamp/2021/))で作成した
元祖「UnitTracking」という原型となるものがあり、大学生が使うなら「スマホアプリでしょ！」ってことで作り始めました

## スマホアプリでしょ！
それでスマホアプリってことは決めたんですけど、現状としては実際に作ったことがあるわけでもないし、Swift、Kotlin、Flutter、ReactNativeって名前を知っているだけの感じでした。(今振り返ってみたらすごいな、、)

それから色々調べたり、実際に触ったりしながら、バックエンドはFirebaseを使っていたということもあり、相性がいいFlutterで作ってみようと決めました。

## なぜ単位管理？
まず自分自身が大学生ということもあり、もし周りで実際に自分が作ったアプリが使われていたらめちゃくちゃ嬉しいだろうなということで大まかなセグメントを大学生としました。

大学生としたところでマッチング系のアプリだと差別化や技術的に大変そうであったり、また巷には時間割管理、課題管理等がある中で単位管理は調べてもなかったので、これだ！ということで単位管理アプリに決めました。

## 余談
きっかけはハッカソンということですけど、それ以外にも下記に上げたような作るメリットがあったのでここまでモチベーションを保ってリリースまでできたということはあります。

- スマホアプリという一つのプロダクトを作った経験はインターンとか就活の選考にプラスになる
- もしかしたらアプリの収入でウハウハ😋😋
- 新しい技術の勉強になる（スキルの向上）
- [技育展](https://talent.supporterz.jp/geekten/2021/)での発表のネタになる 


# 技術スタック
UnitTrackingの使用技術について紹介します。

- モバイルアプリ: Flutter(Dart)
  - 状態管理: Riverpod + StateNotifier + Freezed 
  [monoさんの記事を参考に↑](https://medium.com/flutter-jp/state-1daa7fd66b94)

- バックエンド: Firebase
  - 認証: Firebase Authentication
  - DB: Firestore

- パッケージ
  - 円グラフ: [fl_chart](https://pub.dev/packages/fl_chart)
  - バーグラフ: [percent_indicator](https://pub.dev/packages/percent_indicator)
  - Google広告: [google_mobile_ads](https://pub.dev/packages/google_mobile_ads)
  - スプラッシュ画面: [flutter_native_splash](https://pub.dev/packages/flutter_native_splash)

  パッケージについてはこちらのQiitaの記事がとても良かったです！

https://qiita.com/kazumaz/items/876e162cf429014661d8

## Flutter x Firebaseの相性が最高！
FirebaseとFlutterはどちらもGoogle社製で連携に対してプラグインを持つなど公式のFlutterチームによってメンテナンスもされていたり、ドキュメントも充実していてとても良かったです。

https://firebase.flutter.dev/docs/overview

またFirebaseではモバイルアプリで必要な**認証**、**DB**、**PUSH通知**、**広告**など20種類程度の様々なサービスを利用することができ、従量課金で無料枠もかなり広いため、気軽に利用することができます。

![](https://storage.googleapis.com/zenn-user-upload/d729536a4638023cf41aab9b.png)

*引用: [https://www.flutter-study.dev/firebase/about-firebase/](https://www.flutter-study.dev/firebase/about-firebase/)*

# ディレクトリ構成

```lib
.
├── app.dart
├── generated_plugin_registrant.dart
├── main.dart // ProviderScopeでラップ
├── data
│   ├── controllers //stateの変更などを管理
│   │   ├── controllers.dart
│   │   ├── course_list
│   │   │   └── course_list_controller.dart
│   │   ├── firebase_auth
│   │   │   └── firebase_auth_controller.dart
│   │   ├── plan_user
│   │   │   ├── plan_user_course_controller.dart
│   │   │   └── plan_user_unit_controller.dart
│   │   └── user_unit
│   │       └── user_unit_controller.dart
│   ├── models //データの保持
│   │   ├── course
│   │   │   ├── course.dart
│   │   │   ├── course.freezed.dart
│   │   │   └── course.g.dart
│   │   ├── course_list
│   │   │   ├── course_list.dart
│   │   │   └── course_list.freezed.dart
│   │   ├── models.dart
│   │   ├── plan_user_course
│   │   │   ├── plan_user_course.dart
│   │   │   ├── plan_user_course.freezed.dart
│   │   │   └── plan_user_course.g.dart
│   │   ├── plan_user_unit
│   │   │   ├── plan_user_unit.dart
│   │   │   ├── plan_user_unit.freezed.dart
│   │   │   └── plan_user_unit.g.dart
│   │   └── user_unit
│   │       ├── user_unit.dart
│   │       ├── user_unit.freezed.dart
│   │       └── user_unit.g.dart
│   ├── provider
│   │   ├── bottom_navigation_bar_provider.dart
│   │   ├── course_page_provider.dart
│   │   ├── firebase_provider.dart
│   │   ├── input_unit_provider.dart
│   │   ├── login_page_provider.dart
│   │   └── register_page_provider.dart
│   └── repository //Firestoreの処理
│       ├── firebase_auth_repository.dart
│       ├── plan_user_repository.dart
│       └── user_unit_repository.dart
├── pages // 画面単位で分割
│   ├── base.dart
│   ├── course
│   │   ├── course_page.dart
│   │   └── select_dialog.dart
│   ├── login
│   │   └── login_page.dart
│   ├── password_reset
│   │   └── password_reset_page.dart
│   ├── register
│   │   └── register_page.dart
│   ├── top
│   │   └── top_page.dart
│   ├── unit
│   │   ├── input_unit_dialog.dart
│   │   ├── input_unit_page.dart
│   │   └── unit_page.dart
│   └── user
│       └── user_page.dart
└── widgets // 共通widget
    ├── ad_banner.dart
    ├── error_dialog.dart
    ├── green_elevated_button.dart
    ├── indicator.dart
    └── widegts.dart
```

[こちらのサイトを参考にしました](https://qiita.com/mazahs/items/36d31c182a838a675f60)

# デザイン
デザインは[figma](https://www.figma.com/)を使いました。デザインはFlutterのWidgetに合わせて作っていくことを意識し、色はfigmaにはたくさんの良質なテンプレートがあるのでそれを参考に作りました。

またデザインとかレイアウトなどはできるわけでも得意なわけでもないので、もう既に素晴らしいデザイナーさんが関係しているアプリのLINEとかTwitter、Facebook、Googleなどの使えそうな良いところを探しながらっていう感じで、、

![](https://storage.googleapis.com/zenn-user-upload/08e5e724d4136c799ab0d6c2.png)

# 開発して良かったこと
- **Flutter + Firebase**について知ることができた
- スマホアプリの**開発からリリースまで**一通りは経験することができた
- 1人でやり遂げるという**経験**をつめた
- 自由に開発できる
- 自分のプロダクトを持つことができた(ポートフォリオにもなる!!)
- 単純に楽しかった 👍 

# 開発で大変だったこと
- **モチベーションの維持**
  やっぱ個人開発なので当たり前ですけど、ずっと一人で開発なので技術的な困難さにぶち当たってやる気がなくなったり、クオリティーの低さに沈んだりといろいろな壁にぶち当たりました。

- **モックアップ通りに作れない**
  FlutterのWidget的に作れないわけではないですけど、結構の労力を割かないと思いどおりのように作れなかったり、個人開発だからできたことだとは思いますが開発しながらできないとこは妥協して変更してという感じでなんとかモックアップ通りというわけにはいかないんですけど実装できました。

- **状態管理わからん**
  Flutterを2週間ぐらいしか学習していないということもあって、一番立ち止まったとこがprovider + state notifier + freezedでの状態管理です。
  ~~今でもまだわからないんですけど（笑）~~ 
  特にその状態管理で、Firestoreからとってきたデータを状態管理に当てはめるというとこがめちゃくちゃわかんなくて大変でした。。

- **codemagicらへんはわからん**
  testコードなどは早めにリリースしたかったので、そんなにちゃんとやってないので、ちょっと便利さをわからなかったですし、なんか色々と設定が大変だなと一応使ってみての感想ではそんな感じでした

# UnitTrackingのこれからについて
- **キャンパスの学生が約2万人いるみたいなので、その半分の1万人ユーザー登録が目標！！**
- **新しい機能（チャット機能）などの追加**
- **新入生が入ってきても毎年使ってもらえるようなアプリに**

初めてのアプリで経験という一番大きなものは得られて満足なんですけど、やっぱりこのアプリがどこまで成長していくのか、自分がどれぐらい成長させることができるのか頑張りどころであり、とても楽しみです！

またまだインストールして使ってない方はぜひとも使ってみての感想など、または記事への感想でも、もらえたら開発へのモチベーションにめちゃくちゃなりますので待ってます！ :palms_up_together:

後はところどころキャプチャー画面でのデバックとなっているところは直さないとな。。（笑）

![UnitTracking](https://storage.googleapis.com/zenn-user-upload/fa5324b08a93fb3eae1ba2af.png)

ダウンロードはこちら:point_down:
[iOS]()
[Android](https://play.google.com/store/apps/details?id=net.unit_tracking.unit_tracking)

# お世話になった記事など
https://medium.com/flutter-jp/state-1daa7fd66b94
https://qiita.com/masaibar/items/fdc1a7ee46de0edcd612
https://zenn.dev/kazutxt/books/flutter_practice_introduction
https://medium.com/@riscait/building-and-delivering-a-flavor-enabled-flutter-app-using-codemagic-5225a6070224
