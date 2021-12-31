---
title: "【Flutter】flutter_hooksのuseAnimationControllerを使ってアニメーションを実装してみた"
emoji: "📑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter", "hooks", "アニメーション"]
published: true
---

# 概要
今回、下記パッケージを使っていくということでexsampleの方のStatefulWidgetで書かれたコードはちゃんと動いたのですが、プロジェクト自体基本的にはHooksWidgetを使っていたので、flutter_hooksには、デフォルトでuseAnimationという便利なフックを備えているらしいので今回は使用してみました。

# 実装イメージ
この画像のようなやつを作っていきます！！
![](https://storage.googleapis.com/zenn-user-upload/6e960e13a95e3902061fcac3.png)

# パッケージ

https://pub.dev/packages/liquid_progress_indicator

https://pub.dev/packages/flutter_hooks

# 実装
記事数は少ないですが、useAnimationControllerを使ってアニメーションを実装する方法の記事はあったんですが試してみたんですけど、正直どれも上手く動きませんでした。

https://qiita.com/ko-izumi/items/c3ec99d5a956ac94c9fc

https://qiita.com/toda-axiaworks/items/f1f7acf9a636f52c67b3

https://qiita.com/zwtin/items/5f5557a9dd8746038c87

:::message
そこまでこの記事自体の情報が詳しく書かれているということではないので、上記記事も参考にしていただけるといいと思います。
:::

色々と試していく中で上手く動かないなという中で下記の記事を参考にしますとなぜだか動きましたのでその動いたコードを紹介していきたいと思います！

https://stackoverflow.com/questions/64923336/flutter-useanimationcontroller-hook-doesnt-rebuild-widget

## HookWidget

```dart
import 'package:flutter/material.dart';
import 'package:flutter_hooks/flutter_hooks.dart';
import 'package:liquid_progress_indicator/liquid_progress_indicator.dart';

class LoadingIndicator extends HookWidget {
  const LoadingIndicator({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final animationController = useAnimationController(
      duration: const Duration(seconds: 10),
    );
    useAnimation(animationController);
    final percentage = animationController.value * 100;

    useEffect(() {
      animationController.repeat();
      return () {};
    }, const []);

    return Container(
      width: double.infinity,
      height: 75,
      padding: const EdgeInsets.symmetric(horizontal: 24),
      child: LiquidLinearProgressIndicator(
        value: animationController.value,
        backgroundColor: Colors.white,
        valueColor: const AlwaysStoppedAnimation(Colors.greenAccent),
        borderRadius: 12,
        center: Text(
          '${percentage.toStringAsFixed(0)}%',
          style: const TextStyle(
            color: Colors.lightGreenAccent,
            fontSize: 20,
            fontWeight: FontWeight.bold,
          ),
        ),
      ),
    );
  }
}
```

## statefulWidget
``` liquid_progress_indicator ``` のパッケージのexsampleそのままのコードですけど、Statefullwidgetのバージョンのコードも載せときます。

```dart
class LoadingIndicator extends StatefulWidget {
  const LoadingIndicator({Key? key}) : super(key: key);

  @override
  _LoadingIndicatorState createState() => _LoadingIndicatorState();
}

class _LoadingIndicatorState extends State<LoadingIndicator>
    with SingleTickerProviderStateMixin {
  late AnimationController _animationController;

  @override
  void initState() {
    super.initState();
    _animationController = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 5),
    );

    _animationController.addListener(() => setState(() {}));
    _animationController.repeat();
  }

  @override
  void dispose() {
    _animationController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final percentage = _animationController.value * 100;
    return Center(
      child: Container(
        width: double.infinity,
        height: 75,
        padding: const EdgeInsets.symmetric(horizontal: 24),
        child: LiquidLinearProgressIndicator(
          value: _animationController.value,
          backgroundColor: Colors.white,
          valueColor: const AlwaysStoppedAnimation(Colors.blue),
          borderRadius: 12,
          center: Text(
            '${percentage.toStringAsFixed(0)}%',
            style: const TextStyle(
              color: Colors.lightBlueAccent,
              fontSize: 20,
              fontWeight: FontWeight.bold,
            ),
          ),
        ),
      ),
    );
  }
}
```

# 最後に
二つのコードを見比べてもわかるようにuseAnimationControllerを使うことでコード量を短くできシンプルに書くことができました。是非参考にしてみてください！

またご意見や間違い箇所があればコメント等でお知らせいただくと有り難いです。