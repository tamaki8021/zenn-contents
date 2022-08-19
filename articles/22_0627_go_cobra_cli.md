---
title: "【Golang】1日でGoのcobraでサクッとCLIが作れちゃった話"
emoji: "🥴"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Go", "cobra", "CLI"]
published: true
---

# 作ったもの
![](https://storage.googleapis.com/zenn-user-upload/9709b21d4dc9-20220627.gif)

[リポジトリ](https://github.com/mokupuro/geekhaku-cli)

# 初めに

## 作った背景
[技育博](https://talent.supporterz.jp/geekhaku/2022/)という学生のIT団体が交流するイベントがありまして、
その前日に「イベント用に何か作ろう！」ということで作り始めたというのが開発の背景といなります。
まさかのイベント前日に「1日ハッカソン」が開催されたというわけです、（笑）

ちなみに自分は今回紹介するGoでCLIを作る訳になったんですけど、この段階でGo言語を使った経験がバリバリにあるわけでもなく、
チュートリアルを少し触った程度であり、直近の自分のGoのリポジトリを見てみたら最後の更新が昨年の10月だったということもあり約8ヶ月ぶりにGoを触るという、、

このことから伝えたいことは、俺すごいだろ！ではなく、思ったより楽に見栄えがいい制作物が作れるよ！であり、LTなど発表する機会がある時に、時間も発表する材料もないという時にオススメじゃん！ということが伝わればいいなと思ってます！

## GoでCLIってどんなして作るの？？
色々なフレームワークなどがあるみたいですけど、色々と調べた限り下記3つ!
- Goのデフォで入っている[flag](https://pkg.go.dev/flag)
Goでの代表的なCLIフレームワークの2つ
- https://github.com/spf13/cobra
- https://github.com/urfave/cli

この3つの選択肢の中から今回は[cobra](https://github.com/spf13/cobra)を使ってCLIを作成することに決めました！
選んだ理由は2つ
- 有名どころにも使われているらしく、参考になる記事が比較的多い
- ボイラーテンプレートが使えて最初に簡単な雛形を作ってくれる！
  - ライセンスの作成などもやってくれる

> `go install` `go get`などまだまだ勉強が足りずどちらを使えばいいのかわからないので、状況に合してうまくできなかったらどっちかを試してみるなどやってみてください。[参考](https://qiita.com/eihigh/items/9fe52804610a8c4b7e41)

# やっていく

## Goのインストール・環境構築
自分の環境では[Go](https://go.dev/doc/install)のインストールは行っていなかったので、VSCodeの[Remote Container](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)を使って、ポチ、ポチ、ポチでGoの環境を作りました！
[こんな感じで簡単に立ち上げができる](https://twitter.com/sesere115/status/1538158545006342144)


```
$ mkdir myapp
$ cd myapp
// go modの作成
$ go mod init <importpath>
```

## cobra・cobra-cliのインストール
> cobra-cli is a command line program to generate cobra applications and command files. It will bootstrap your application scaffolding to rapidly develop a Cobra-based application. It is the easiest way to incorporate Cobra into your application. 

訳）cobra-cli は cobra アプリケーションとコマンドファイルを生成するための コマンドライン・プログラムです。これは、Cobraベースのアプリケーションを迅速に開発するために、アプリケーションの雛形をブートストラップします。

```
$ go install github.com/spf13/cobra-cli@latest
```

```
$ go get -u github.com/spf13/cobra@latest
```

## cobra-cli init

```
# コマンドのボイラーテンプレートを作成。
# viper=trueにすれば設定ファイル読み込み機能ありなものになる。
$ cobra init --license MIT --viper=false
```

こんな感じの雛形が出来上がる
```
├── cmd
│   └── root.go
├── go.mod
├── go.sum
├── LICENSE
├── main.go
```
### コマンド実行
最低限の準備ができたのでまずは動くか試してみる。うまく動けばデフォルトのディスクリプションが表示される。

```
$ go run main.go
A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.
```

## サブコマンドの追加

```
// cobra-cli add <サブコマンド名>
$ cobra-cli add version
```

```
├── cmd
│   └── root.go
│   └── version.go
```

```go: cmd/version.go

package cmd

import (
	"fmt"

	"github.com/spf13/cobra"
)

var versionCmd = &cobra.Command{
	Use:   "version",
	Short: "Print the version number of geekhaku-cli",
	Long: "All software has versions. This is geekhaku-cli's",
	Run: func(cmd *cobra.Command, args []string) {
    // ここに処理を書いていく
		fmt.Println("version 0.9 -- HEAD")
	},
}

func init() {
	rootCmd.AddCommand(versionCmd)
}
```

### 実行

```
$ go run main.go version
version 0.9 -- HEAD
```

# アスキーアートを表示してみる
今回は可愛い猫のアスキーアートを表示するコマンドを作成します。

## アスキーアート
ルートディレクトに`aa`ディレクトリを作って`.txt`ファイルに表示したいアスキーアートを貼り付けます。
> アスキーアートについては、「アスキーアート　ジェネレーター」「アスキーアート　コピー」などで検索してみると色々と出てくるのでお好きなもので試してみてください。

```txt: cat.txt

　　　　　　　　　 ;' ':;,,　　　　　,;'':;,
　　　　　　　　　;'　　 ':;,.,.,.,.,.,,,;'　　';, 　　っ
　　　　　　　　,:'　　　　　　　　 　　: :､
　　　　　　　,:'ノ　 　 　　　＼　　 ::::::::', 　　っ
　　　　　　 :'　 ●　　　　 ●　 　 　 :::::i.
　　　　　　 i　///(_人＿) ////＊　 :::::i 　 
　　　　　 　,i.　 　 　し'　 　 　　　　　:::::i　　　　　
　　　　　　(⌒) 　 　　　　　(⌒) ::::::::: / 　　 　　　　
　　　　 　　;,　:'　　　　　　　; : : ::::::::｀:､
　　　　 　　 ; ,:'　　　　　　　 ';,.　: : ::::::｀:､


```

## サブコマンドの作成
```
$ cobra-cli add cat
```

```
├── cmd
│   └── root.go
│   └── version.go
│   └── cat.go
```

## ファイルの読み込み出力の処理を追加

```go: cat.go
package cmd

import (
  "fmt"
  "os"

	"github.com/spf13/cobra"
)

var catCmd = &cobra.Command{
	Use:   "cat",
	Short: "Print the ascii art of cat",
	Run: func(cmd *cobra.Command, args []string) {
    b, err := os.ReadFile(fp)
    if err != nil {
      fmt.Println(err)
    }
    fmt.Print(string(b))
	},
}

func init() {
  rootCmd.AddCommand(catCmd)
}
```

## 実行

```
$ go run main.go cat

　　　　　　　　　 ;' ':;,,　　　　　,;'':;,
　　　　　　　　　;'　　 ':;,.,.,.,.,.,,,;'　　';, 　　っ
　　　　　　　　,:'　　　　　　　　 　　: :､
　　　　　　　,:'ノ　 　 　　　＼　　 ::::::::', 　　っ
　　　　　　 :'　 ●　　　　 ●　 　 　 :::::i.
　　　　　　 i　///(_人＿) ////＊　 :::::i 　 
　　　　　 　,i.　 　 　し'　 　 　　　　　:::::i　　　　　
　　　　　　(⌒) 　 　　　　　(⌒) ::::::::: / 　　 　　　　
　　　　 　　;,　:'　　　　　　　; : : ::::::::｀:､
　　　　 　　 ; ,:'　　　　　　　 ';,.　: : ::::::｀:､

```

🎉🎉事表示できました!!

# リリースしてみる

## 静的ファイルがバイナリに含まれるように変更
Go は通常、ビルド時にソースファイル以外の静的ファイルをバイナリに含めることができないので（今回で言うと`.txt`ファイル） ~~[statik](https://github.com/rakyll/statik)というパッケージを使って、~~ ビルド時に一緒にバイナリに含めていきます！

~~[こちらの記事](https://zenn.dev/kou_pg_0131/articles/go-build-with-static-files-by-statik)で紹介されている通りにやっていけば問題ないと思います！~~


今回は[statik](https://github.com/rakyll/statik)を使った方法を紹介していますが、
下記の[コメント](https://zenn.dev/link/comments/fa20d3469b66b3)にあるように`Go1.16`からオフィシャルとしてサポートされた、`go:embed`を使った方が良さそうです！

:::message
 `go:embed`を使用した方法を追加しました！
:::

## go:embedを使用した方法

### ファイルを読み込む設定をする

`embed`では、埋め込むファイルはembedを記述するファイルが有る場所からの相対パスになります。例えば、 libs/hoge.go の中でembedする場合、 libs/ 以下のファイルのみ埋め込むことができます。
今回は`aa`以下のファイルを読み取りたいので、`embed.go`ファイルを`aa`ディレクトリ下に作成し、独自の独自のパッケージとしてプログラムの他の部分でも使用できるようにします。
```
.
├── aa
│   └── cat.txt
│   └── embed.go
```

```go: embed.go
package aa

import "embed"

//go:embed *.txt
var Aa embed.FS
```

### 生成されたファイルを読み込む

```go: cat.go
package cmd

import (
  "fmt"
  "os"

 	 "<mod initの時に宣言したimportpath>/aa"
	"github.com/spf13/cobra"
)

var catCmd = &cobra.Command{
	Use:   "cat",
	Short: "Print the ascii art of cat",
	Run: func(cmd *cobra.Command, args []string) {
    file, err := aa.Open("cat.txt") // 相対パスなので / を取り除いてファイル名を指定
    if err != nil {
      fmt.Println(err)
    }
    defer file.Close()

    // ファイルを読み込んで出力
    buf := new(bytes.Buffer)
    buf.ReadFrom(file)

    fmt.Print(buf.String())
	},
}

func init() {
  rootCmd.AddCommand(catCmd)
}
```

:::details statikを使用した方法

### ファイルを生成する
```
 $ statik -src=./aa
```
statik/statik.go というファイルが生成されます。

```
.
├── aa
│   └── cat.txt
└── statik
    └── statik.go
```

### 生成されたファイルを読み込む

```go: cat.go
package cmd

import (
  "fmt"
  "os"

 	_ "<mod initの時に宣言したimportpath>/statik"
	"github.com/rakyll/statik/fs"
	"github.com/spf13/cobra"
)

var catCmd = &cobra.Command{
	Use:   "cat",
	Short: "Print the ascii art of cat",
	Run: func(cmd *cobra.Command, args []string) {
    statikFS, err := fs.New()
    if err != nil {
      fmt.Println(err)
    }

    file, err := statikFS.Open("/cat.txt") // 絶対パスで / が先頭に付く
    if err != nil {
      fmt.Println(err)
    }
    defer file.Close()

    // ファイルを読み込んで出力
    buf := new(bytes.Buffer)
    buf.ReadFrom(file)

    fmt.Print(buf.String())
	},
}

func init() {
  rootCmd.AddCommand(catCmd)
}
```
:::

## GitHub Actions と GoReleaser を使って brew コマンドでインストールできるようにする

[Go で書いた CLI ツールを GitHub Actions と GoReleaser を使って brew コマンドでインストールできるようにした
](https://michimani.net/post/development-publish-cli-tool-to-homebrew/)
この記事を参考にリリースまでを行うことができたので貼っておきます！

`.github/workflows/release.yml`のGITHUB_TOKENについては今回はリポジトリがorganizationの配下だったため、[こちらの記事](https://zenn.dev/suzutan/articles/how-to-use-github-apps-token-in-github-actions)の手順でtokenを設定することで無事にリリースまでできました。

# 最後に
今回は簡単にアスキーアートを表示するだけのCLIでしたが、Goがわからないなりにリリースまでできたので、見栄えがいい制作物をサクッと作れたり、Goの勉強がてらに色々といじってみたりしたら勉強にもなるし楽しいだろうなと思いました！

また、今回は紹介しなかった CLIでの選択式の実装は[promptui](https://github.com/manifoldco/promptui)で楽に実装できるので、もしよければそちらも試してみてください！
最後に今回の記事で動かない場所やうまくいかなかった場所などがあれば、コメント等で指摘いただけるとありがたいです！