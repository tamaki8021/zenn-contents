---
title: "Firebaseで作るチャット機能のFirestoreルールの例"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["firebase", "firestore", "rule", "セキュリティー", "チャット"]
published: true
---

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // すべてのコレクションで使える関数
    // リクエストが認証されたユーザーからのものかどうか
    function isSignedIn() {
      return request.auth != null;
    }

    // ユーザーコレクション
    match /users/{userId} {
      // ユーザーのオブジェクト形式を検証する
      // 名前を指定する予定がない場合は使わない
      function isUserCorrect() {
        return isSignedIn() && request.resource.data.firstName is string && request.resource.data.lastName is string;
      }

      // ドキュメントが現在ログインしているユーザーによって作られたものかどうかを確認する
      function isSelf() {
        return request.auth.uid == userId;
      }

      // ユーザーコレクションのルールを設定
      allow create: if isUserCorrect();
      allow delete: if isSelf();
      allow read: if isSignedIn();
      allow update: if isUserCorrect() && isSelf();
    }

    // ルームコレクション
    match /rooms/{roomId} {
      // 現在ログインしているユーザーがusersコレクションに存在しているかどうかを確認
      function userExists() {
        return isSignedIn() && exists(/databases/$(database)/documents/users/$(request.auth.uid));
      }

      // 現在ログインしているユーザーがroomにいるかどうか
      function isUserInRoom() {
        return isSignedIn() && request.auth.uid in resource.data.userIds;
      }

      // ルームルームのオブジェクト形式を検証
      function isRoomCorrect() {
        return request.resource.data.type is string && request.resource.data.userIds is list;
      }

      // ルームコレクションのルールを設定
      allow create: if userExists() && isRoomCorrect();
      allow delete, read, update: if isUserInRoom();

      // メッセージコレクション
      match /messages/{messageId} {
        // 現在ログインしているユーザーがルームに居るかどうかを確認
        function isUserInRoomUsingGet() {
          return isSignedIn() && request.auth.uid in get(/databases/$(database)/documents/rooms/$(roomId)).data.userIds;
        }

        // メッセージのオブジェクト形式を検証
        function isMessageCorrect() {
          return request.resource.data.authorId is string && request.resource.data.createdAt is timestamp;
        }

        // メッセージの作成者が現在ログインしているユーザーかどうか
        function isMyMessage() {
          return request.auth.uid == resource.data.authorId;
        }

        // メッセージコレクションのルールを設定
        allow create: if isSignedIn() && isMessageCorrect();
        allow delete, read: if isUserInRoomUsingGet();
        allow update: if isUserInRoomUsingGet() && isMyMessage();
      }
    }
  }
}
```