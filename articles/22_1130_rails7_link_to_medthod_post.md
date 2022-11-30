---
title: "【Rails7】link_toで作成したリンクのmethod: :postがGETになる問題"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rails", "turbo"]
published: true
---

## エラー内容
「Rails7」でviewテンプレートに以下のコードを書きました。

```rb
  <%= link_to chat_rooms_path(params: { user_id: user.id }), method: :post, class: "no-underline" %>
```

チャットルームコントローラーのPOSTメソッドを使いたいところ、GETで呼び出されたためGETのメソッドはなかったので下記の画像のようにエラーになりました。

![](https://storage.googleapis.com/zenn-user-upload/8d00e64a2eff-20221130.png)

## 解決方法
- Rails6の場合「rails-ujs」を読み込む必要がある。[参考](https://blog.ezic.info/43631.html)
- Rails7からはTurboがデフォルトになり、rails-ujsの機能はTurboが引き受ける形になったので、data-methodではなくdata-turbo-methodを使うようになった[参考](https://zenn.dev/shita1112/books/cat-hotwire-turbo/viewer/turbo-drive#%E3%83%AA%E3%83%B3%E3%82%AF%E3%81%A7post%E3%83%BBput%E3%83%BBpatch%E3%83%BBdelete%E3%82%92%E4%BD%BF%E3%81%86)

```rb
  <%= link_to chat_rooms_path(params: { user_id: user.id }), data: { "turbo-method": :post }, class: "no-underline" %> 
```