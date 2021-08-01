---
title: "DiscordのメッセージをLINEに共有する"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ruby", "sinatra", "discord", "LINE"]
published: false
---

# 概要
Discord使っていて通知の面だったり、LINEのほうが日常的に使っていて浸透しやすいという事があったので、LINEとDiscord両方に同じメッセージを送るのがめんどくさいので、DiscordのメッセージをLINEにも送るbotを作ることにしました。

下記のサイトを参考にさせてもらいました。
https://qiita.com/kkbase515/items/d7ada101e3095c2bd051
https://qiita.com/fullmated/items/81d1a49ed3d49eda2285

# はじめに
この記事では
- Heroku使ったことない
- RubyはProgateで一周ぐらい
- Sinatraは全く知らない
という人に向けて説明していきます。
かくいう私自身がその状態だったので、復習、議事録という意味でもこの記事を書きました。

環境はMacOSです。
Rubyはインストール済みです。
https://prog-8.com/docs/ruby-env

# Herokuの準備をする
1. Herokuのサイトからアカウントを作成する
https://www.heroku.com/home
2. Heroku CLIをインストールしてコマンドライン上でherokuの操作を可能にする

```
// ダウンロード及びインストール
$ brew tap heroku/brew && brew install heroku

// インストールの確認
$ heroku --version
heroku/7.0.0 (darwin-x64) node-v8.0.0
```

3. herokuコマンドが叩けて、ログインできたらOK
```
$ heroku login
heroku: Press any key to open up the browser to login or q to exit
 ›   Warning: If browser does not open, visit
 ›   https://cli-auth.heroku.com/auth/browser/***
heroku: Waiting for login...
Logging in... done
Logged in as me@example.com
```

4. 作業ディレクトリを作成
```
$ mkdir freechat_app
$ cd freechat_app/
$ heroku create
```
https://devcenter.heroku.com/ja/articles/heroku-cli#verifying-your-installation

# sinatraを使う
- sinatraをBundler経由でインストールします。
```
$ bundle init
$ vim Gemfile
```

- vimでGemfileを編集していきます。
```Gemfile
source "https://rubygems.org"
gem 'sinatra'
gem 'line-bot-api'
gem 'discordrb'
get 'webrick'
```

- インストールする
```
$ bundle install --path=vendor/bundle
```

- app_mainファイルを作成する
```
$ touch app_main.rb
```

```app_main.rb
require 'sinatra'
require 'line/bot'

get '/' do
  "Hello world"
end
```

- Hello wordを表示してみる
```
$ bundle exec ruby app_main.rb
[2021-08-01 17:43:55] INFO  WEBrick 1.7.0
[2021-08-01 17:43:55] INFO  ruby 3.0.2 (2021-07-07) [x86_64-darwin20]
== Sinatra (v2.1.0) has taken the stage on 4567 for development with backup from WEBrick
[2021-08-01 17:43:55] INFO  WEBrick::HTTPServer#start: pid=62465 port=4567
```

この状態で
http://localhost:4567
にアクセスしてHello worldできたらOK.

# LINE botアカウントの作成
下記公式サイトを参考にしながら作成します。
https://developers.line.biz/ja/docs/messaging-api/building-bot/

作成したらLINE Developersページの作成したアカウントのBasic settingsタブの**Channel secret**とMessaging APIタブの**Channel access token**を発行します。

# Herokuに環境変数を設定する
コマンドで環境変数も設定できるみたいですけど、なんか自分側の環境ではできなかったのでサイト側から設定しました。

|   環境変数名(KEY)   |       値(VALUE)      |
| ------------------- | -------------------- |
| LINE_CHANNEL_SECRET | Channel Secret       |
| LINE_CHANNEL_SECRET | Channel access token |

https://qiita.com/mzmz__02/items/64db94b8fc67ee0a9068#%E3%82%B5%E3%82%A4%E3%83%88%E3%81%8B%E3%82%89%E7%92%B0%E5%A2%83%E5%A4%89%E6%95%B0%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B