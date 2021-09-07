---
title: "【Ruby sinatra】DiscordのメッセージをLINEのグループでも共有する"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ruby", "sinatra", "discord", "linebot", "業務効率化"]
published: true
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

## 環境
- MacOS11.4
- Rubyインストール済み
ruby 3.0.2p107 (2021-07-07 revision 0db68f0233) [x86_64-darwin20]
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
```rb:Gemfile
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

```rb:app_main.rb
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


# DiscordBotを作成する
コマンド + <message内容>を送るとbotがLINEににメッセージを送るようにbotを作ります。

Discord Developerにアクセスし、botを作成する
この記事を参考にしてサーバーにbotを追加する
https://note.com/exteoi/n/nf1c37cb26c41


# Herokuに環境変数を設定する
コマンドで環境変数も設定できるみたいですけど、なんか自分側の環境ではできなかったのでサイト側から設定しました。

- LINE

|   環境変数名(KEY)   |       値(VALUE)      |
| ------------------- | -------------------- |
| LINE_CHANNEL_SECRET | Channel Secret       |
| LINE_CHANNEL_TOKEN  | Channel access token |


- Discord

|   環境変数名(KEY)  |     値(VALUE)     |
| ---- | ---- |
| DISCORD_BOT_TOKEN  | Discord Bot token |
| DISCORD_CLIENT_ID  | Discord Client ID |

計4つの環境変数をheroku側で設定します。

https://qiita.com/mzmz__02/items/64db94b8fc67ee0a9068#%E3%82%B5%E3%82%A4%E3%83%88%E3%81%8B%E3%82%89%E7%92%B0%E5%A2%83%E5%A4%89%E6%95%B0%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B


# コード

```rb:app_main.rb
require 'line/bot'
require 'discordrb'
require 'net/http'
require 'uri'
require 'json'

token = ENV["DISCORD_BOT_TOKEN"]
clientID = ENV["DISCORD_CLIENT_ID"]

bot = Discordrb::Commands::CommandBot.new token: token, client_id: clientID, prefix: "!"

def client
  @client ||= Line::Bot::Client.new { |config|
    config.channel_secret = ENV["LINE_CHANNEL_SECRET"]
    config.channel_token = ENV["LINE_CHANNEL_TOKEN"]
  }
end

# messageと送信者名を合わせる
def add_message(str1, str2)
  return str1 + str2
end

bot.command :line do |event|
  bot_message = event.message.content
  bot_message.slice!(0,6)
  event.respond "#{bot_message}"
  push_line(add_message(event.user.name, bot_message))
  # messageだけ送信したい場合
  # push_line(bot_message)
end


def push_line(message)

  uri = URI.parse("https://api.line.me/v2/bot/message/push")

  request = Net::HTTP::Post.new(uri)
  request.content_type = "application/json"
  request["Authorization"] = "Bearer #{ENV["LINE_CHANNEL_TOKEN"]}"
  request.body = JSON.dump({
    "to" => "groupId",
    "messages" => [
      {
        "type" => "text",
        "text" => message
      }
    ]
  })

  req_options = {
    use_ssl: uri.scheme == "https",
  }

  response = Net::HTTP.start(uri.hostname, uri.port, req_options) do |http|
  http.request(request)
  end
end

bot.run
```

## groupIdの取得方法
この記事を参考にLINEのグループIDを取得する
https://blog.taka73.net/2020/12/23/line-messaging-api-line-bot/

```rb:app_main.rb
  request.body = JSON.dump({
    "to" => "groupId", <- このgroupIdを取得したIDに置き換える
    "messages" => [
      {
        "type" => "text",
        "text" => message
      }
    ]
  })
```

## Procfileの作成
Procfileは"Herokuに何をさせるか"が書かれた指示書のようなファイルです。

```rb:Procfile
bundle exec ruby app_main.rb
```

## ディレクトリ構造
```
.bundle 
.gitignore 
Gemfile 
Gemfile.lock 
Procfile 
app_main.rb 
vendor
```


# Herokuにデプロイする
Herokuへのデプロイにはgitを使います。
なのでまずはHeroku側に必要のないファイルをアップロードしないためにも.gitignoreファイルを作成します。

## .gitignoreファイルのコード
```git:.gitignore
#----------------------------------------------------------------------------
# This is the .gitignore file for the rails-composer repository.
#
# Rails Composer is an application template that creates a Rails starter app.
#
# The Rails Composer application template is generated by the rails_apps_composer gem:
# https://github.com/RailsApps/rails_apps_composer
#
# The Rails Composer application template will create a .gitignore file for any starter app.
# This file isn't used by the Rails Composer application template; instead the template uses:
# https://github.com/RailsApps/rails-composer/blob/master/files/gitignore.txt
#
# Corrections? Improvements? Create a GitHub issue: 
# http://github.com/RailsApps/rails-composer/issues
#----------------------------------------------------------------------------

# bundler state
/.bundle
/vendor/bundle/
/vendor/ruby/

# minimal Rails specific artifacts
db/*.sqlite3
/log/*
/tmp/*

# various artifacts
**.war
*.rbc
*.sassc
.rspec
.redcar/
.sass-cache
/coverage.data
/coverage/
/db/*.javadb/
/db/*.sqlite3
/doc/api/
/doc/app/
/doc/features.html
/doc/specs.html
/public/cache
/public/stylesheets/compiled
/public/system/*
/spec/tmp/*
/cache
/capybara*
/capybara-*.html
/gems
/spec/requests
/spec/routing
/spec/views
/specifications
rerun.txt
pickle-email-*.html

# If you find yourself ignoring temporary files generated by your text editor
# or operating system, you probably want to add a global ignore instead:
#   git config --global core.excludesfile ~/.gitignore_global
#
# Here are some files you may want to ignore globally:

# scm revert files
**.orig

# Mac finder artifacts
.DS_Store

# Netbeans project directory
/nbproject/

# RubyMine project files
.idea

# Textmate project files
/*.tmproj

# vim artifacts
**.swp
```

## デプロイ
主なデプロイの方法が2つあって一つは実際にheroku createからheroku pushとコマンドを打ってデプロイするやり方と、GitHubのブランチと連結してデプロイするやり方があります。

今回はこちらの記事を参考にGitHubからのデプロイをやりました。
https://qiita.com/sho7650/items/ebd87c5dc2c4c7abb8f0
https://j-hack.gitbooks.io/deploy-meteor-app-to-heroku/content/step4.html

GitHubにコードを挙げないという人は下記の順序でデプロイできるはずです。
herokuのブラウザ画面から今回使用するアプリケーションのDeployタブの説明をもとに進めていくのが確実です。
```
$ git init
$ heroku git:remote -a <herokuのアプリケーション名>
$ git add .
$ git commit -am "make it better"
$ git push heroku master
```


# bundle exec ruby app_main.rbをしたらエラーになった
```
libsodium not available! You can continue to use discordrb as normal but voice support won't work.
        Read https://github.com/shardlab/discordrb/wiki/Installing-libsodium for more details.
[ERROR : websocket @ 2021-08-02 15:17:57.858] The websocket connection has closed: (no information)
[ERROR : websocket @ 2021-08-02 15:17:57.858] Websocket close frame received!
[ERROR : websocket @ 2021-08-02 15:17:57.858] Code: 4004
[ERROR : websocket @ 2021-08-02 15:17:57.858] Message: Authentication failed.
[WARN : websocket @ 2021-08-02 15:17:57.858] The WS loop exited! Not sure if this is a good thing
```

libsodiumがないよ！っていっているのでURLからコマンドをコピペでインストール
```
$ brew install libsodium
```

https://teratail.com/questions/166426

# herokuにデプロイしようとしたらエラーが出た
```
-----> Installing dependencies using bundler 2.2.21
       Running: BUNDLE_WITHOUT='development:test' BUNDLE_PATH=vendor/bundle BUNDLE_BIN=vendor/bundle/bin BUNDLE_DEPLOYMENT=1 bundle install -j4
       Your bundle only supports platforms ["x86_64-darwin-20"] but your local platform
       is x86_64-linux. Add the current platform to the lockfile with `bundle lock
       --add-platform x86_64-linux` and try again.
       Bundler Output: Your bundle only supports platforms ["x86_64-darwin-20"] but your local platform
       is x86_64-linux. Add the current platform to the lockfile with `bundle lock
       --add-platform x86_64-linux` and try again.
 !
 !     Failed to install gems via Bundler.
 !
 !     Push rejected, failed to compile Ruby app.
 !     Push failed
```

エラー分の中にかいてあるコードをターミナルで実行して、add,commitしてまたプッシュする
```
$ budle lock --add-platform x88_64-linux
```

# 今回のコードのGitHubレポジトリ
https://github.com/tamaki8021/line_discord_connect

# 参考
https://qiita.com/kkbase515/items/d7ada101e3095c2bd051
https://qiita.com/fullmated/items/81d1a49ed3d49eda2285