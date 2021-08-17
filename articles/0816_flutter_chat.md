---
title: "[flutter_chat_ui] Flutter x Firebaseでチャット機能を作成する"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter", "firebase", "chat", "tech"]
published: true
---

# 概要
[flutter_chat_ui](https://pub.dev/packages/flutter_chat_ui)というUIをいい感じにしてくれたり、ファイルやリンク、画像の送信をいい感じに手助けしてくれるパッケージを使って、チャット機能を作っていきます。

- 今回実装したコードのGitHubリポジトリ

https://github.com/tamaki8021/flutter-demo-firebase

:::message
状態管理は範囲外なので、そこのところよろしくおねがいします。
:::


# 環境

```
$ flutter doctor   
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 2.2.2, on macOS 11.4 20F71 darwin-x64, locale ja-JP)
[✓] Android toolchain - develop for Android devices (Android SDK version 30.0.3)
[✓] Xcode - develop for iOS and macOS
[✓] Chrome - develop for the web
[✓] Android Studio (version 4.2)
[✓] IntelliJ IDEA Ultimate Edition (version 2021.2)
[✓] VS Code (version 1.59.0)
[✓] Connected device (1 available)
```

# パッケージインストール
- コマンド

```
$ flutter pub add flutter_chat_ui
```

- pubspec.yaml

```yaml:pubspec.yaml
dependencies:
  flutter_chat_ui: ^1.2.0
```

# チャットルームリスト画面の作成

```dart:main.dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:flutter_demo_firebase/roomList_page.dart';
import 'package:provider/provider.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  // firebaseの初期化
  await Firebase.initializeApp();

  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    //子Widgetにデータを渡す
    return ChangeNotifierProvider<UserState>(
      create: (context) => UserState(),
      child: MaterialApp(
        title: 'Flutter Demo',
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: RoomListPage(),
      ),
    );
  }
}
```

```dart:room_list_page.dart
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:provider/provider.dart';
import 'package:flutter_demo_firebase/addRoom_page.dart';
import 'package:flutter_demo_firebase/chat_page.dart';

class RoomListPage extends StatelessWidget {
  // 引数からユーザー情報を受け取れるようにする
  RoomListPage();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('チャット'),
      ),
      body: Column(
        children: [
          Expanded(
            // Stream 非同期処理の結果を元にWidgetを作る
            child: StreamBuilder<QuerySnapshot>(
            // 投稿メッセージ一覧の取得
            stream: FirebaseFirestore.instance
                .collection('chat_room')
                .orderBy('createdAt')
                .snapshots(),
            builder: (context, snapshot) {
              // データが取得できた場合
              if (snapshot.hasData) {
                final List<DocumentSnapshot> documents = snapshot.data!.docs;
                return ListView(
                  children: documents.map((document) {
                    return Card(
                      child: ListTile(
                        title: Text(document['name']),
                        trailing: IconButton(
                          icon: Icon(Icons.input),
                          onPressed: () async {
                            // チャットページへ画面遷移
                            await Navigator.of(context).push(
                              MaterialPageRoute(
                                builder: (context) {
                                  return ChatPage(document['name']);
                                },
                              ),
                            );
                          },
                        ),
                      ),
                    );
                  }).toList(),
                );
              }
              // データが読込中の場合
              return Center(
                child: Text('読込中……'),
              );
            },
          )),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () async {
          await Navigator.of(context)
              .push(MaterialPageRoute(builder: (context) {
            return AddRoomPage();
          }));
        },
      ),
    );
  }
}
```

![](https://storage.googleapis.com/zenn-user-upload/a612bf7871a85c9d414c6d8a.png)

# チャットルームの作成画面

```dart: add_room_page.dart
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';

class AddRoomPage extends StatefulWidget {
  AddRoomPage();

  @override
  _AddPostPageState createState() => _AddPostPageState();
}

class _AddPostPageState extends State<AddRoomPage> {
  String roomName = '';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('ルーム作成'),
      ),
      body: Center(
        child: Container(
          padding: EdgeInsets.all(32),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              TextFormField(
                decoration: InputDecoration(labelText: 'チャットルーム名'),
                keyboardType: TextInputType.multiline,
                maxLines: 3,
                onChanged: (String value) {
                  setState(() {
                    roomName = value;
                  });
                },
              ),
              const SizedBox(
                height: 8,
              ),
              Container(
                width: double.infinity,
                child: ElevatedButton(
                  child: Text('投稿'),
                  onPressed: () async {
                    final date = DateTime.now().toLocal().toIso8601String();

                    await FirebaseFirestore.instance
                        .collection('chat_room')
                        .doc(roomName)
                        .set({
                      'name': roomName,
                      'createdAt': date,
                    });
                    Navigator.of(context).pop();
                  },
                ),
              )
            ],
          ),
        ),
      ),
    );
  }
}
```

![](https://storage.googleapis.com/zenn-user-upload/045710547dff8d9d3740a521.png)

# チャットページの作成

- メッセージ送信ができるだけのシンプルな実装

```dart:chat_page.dart
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:flutter/material.dart';

// flutter_chat_uiを使うためのパッケージをインポート
import 'package:flutter_chat_ui/flutter_chat_ui.dart';
import 'package:flutter_chat_types/flutter_chat_types.dart' as types;

import 'package:provider/provider.dart';
// ランダムなIDを採番してくれるパッケージ
import 'package:uuid/uuid.dart';


class ChatPage extends StatefulWidget {
  const ChatPage(this.name, {Key? key}) : super(key: key);

  final String name;
  @override
  _ChatPageState createState() => _ChatPageState();
}

class _ChatPageState extends State<ChatPage> {
  List<types.Message> _messages = [];
  String randomId = Uuid().v4();
  final _user = const types.User(id: '06c33e8b-e835-4736-80f4-63f44b66666c', firstName: '名前');


  void initState() {
    _getMessages();
    super.initState();
  }

  // firestoreからメッセージの内容をとってきて_messageにセット
  void _getMessages() async {
    final getData = await FirebaseFirestore.instance
        .collection('chat_room')
        .doc(widget.name)
        .collection('contents')
        .get();

    final message = getData.docs
        .map((d) => types.TextMessage(
            author:
                types.User(id: d.data()['uid'], firstName: d.data()['name']),
            createdAt: d.data()['createdAt'],
            id: d.data()['id'],
            text: d.data()['text']))
        .toList();

    setState(() {
      _messages = [...message];
    });
  }

  // メッセージ内容をfirestoreにセット
  void _addMessage(types.TextMessage message) async {
    setState(() {
      _messages.insert(0, message);
    });
    await FirebaseFirestore.instance
        .collection('chat_room')
        .doc(widget.name)
        .collection('contents')
        .add({
      'uid': message.author.id,
      'name': message.author.firstName,
      'createdAt': message.createdAt,
      'id': message.id,
      'text': message.text,
    });
  }

  // リンク添付時にリンクプレビューを表示する
  void _handlePreviewDataFetched(
    types.TextMessage message,
    types.PreviewData previewData,
  ) {
    final index = _messages.indexWhere((element) => element.id == message.id);
    final updatedMessage = _messages[index].copyWith(previewData: previewData);

    WidgetsBinding.instance?.addPostFrameCallback((_) {
      setState(() {
        _messages[index] = updatedMessage;
      });
    });
  }

  // メッセージ送信時の処理
  void _handleSendPressed(types.PartialText message) {
    final textMessage = types.TextMessage(
            author: _user,
            createdAt: DateTime.now().millisecondsSinceEpoch,
            id: randomId,
            text: message.text,
          );

    _addMessage(textMessage);
  }

  @override
  Widget build(BuildContext context) {

    return Scaffold(
      appBar: AppBar(
        title: Text('チャット'),
      ),
      body: Chat(
        theme: const DefaultChatTheme(
          // メッセージ入力欄の色
          inputBackgroundColor: Colors.blue,
          // 送信ボタン
          sendButtonIcon: Icon(Icons.send),
          sendingIcon: Icon(Icons.update_outlined),
        ),
        // ユーザーの名前を表示するかどうか
        showUserNames: true,
        // メッセージの配列
        messages: _messages,
        onPreviewDataFetched: _handlePreviewDataFetched,
        onSendPressed: _handleSendPressed,
        user: _user,
      ),
    );
  }
}
```

![](https://storage.googleapis.com/zenn-user-upload/8689d9d17b636fdd1e9b4ac2.png)

:::message
ここからは実際に実装して試していないので参考程度に
:::

## 画像送信の処理の追加
- [image_picker](https://pub.dev/packages/image_picker)を使用して画像をメッセージとして送信する

```diff dart:chat_page.dart
import 'package:image_picker/image_picker.dart';

class _ChatPageState extends State<ChatPage> {

+void _handleImageSelection() async {
+    final result = await ImagePicker().pickImage(
+      imageQuality: 70,
+      maxWidth: 1440,
+      source: ImageSource.gallery,
+    );

+    if (result != null) {
+      final bytes = await result.readAsBytes();
+      final image = await decodeImageFromList(bytes);

+      final message = types.ImageMessage(
+        author: _user,
+        createdAt: DateTime.now().millisecondsSinceEpoch,
+        height: image.height.toDouble(),
+        id: randomString(),
+        name: result.name,
+        size: bytes.length,
+        uri: result.path,
+        width: image.width.toDouble(),
+      );

+      _addMessage(message);
+    }
+  }

  @override
  Widget build(BuildContext context) {

    return Scaffold(
      appBar: AppBar(
        title: Text('チャット'),
      ),
      body: Chat(
        theme: const DefaultChatTheme(
          // メッセージ入力欄の色
          inputBackgroundColor: Colors.blue,
          // 送信ボタン
          sendButtonIcon: Icon(Icons.send),
          sendingIcon: Icon(Icons.update_outlined),
        ),
        // ユーザーの名前を表示するかどうか
        showUserNames: true,
        // メッセージの配列
        messages: _messages,
+        onAttachmentPressed: _handleImageSelection,
        onPreviewDataFetched: _handlePreviewDataFetched,
        onSendPressed: _handleSendPressed,
        user: _user,
      ),
    );
  }
```

# flutter x firebase専用のパッケージもあるらしい...
今回は最初の方でも紹介したとおり、[flutter_chat_ui](https://pub.dev/packages/flutter_chat_ui)という名前のパッケージを使ってfirebaseにデータを保存して引っ張ってくるところまで実装しましたが、[flutter_firebase_chat_core](https://github.com/flyerhq/flutter_firebase_chat_core)という名前のflutterとfirebaseに特化したパッケージがあるみたいです。

- GitHub:

https://github.com/flyerhq/flutter_firebase_chat_core

- document:

https://docs.flyer.chat/flutter/firebase/firebase-usage

- exsample:

https://github.com/flyerhq/flutter_firebase_chat_core/tree/main/example

# 参考
- GitHub:

https://github.com/flyerhq/flutter_firebase_chat_core

- exsample:

https://github.com/flyerhq/flutter_chat_ui/tree/main/example

- document:

https://docs.flyer.chat/flutter/chat-ui/basic-usage
