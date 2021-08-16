---
title: "[flutter_chat_ui] Flutter x Firebaseでチャット機能を作成する"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter", "firebase", "chat", "tech"]
published: false
---

# 概要
[flutter_chat_ui](https://pub.dev/packages/flutter_chat_ui)というUIをいい感じにしてくれたり、ファイルやリンク、画像の送信をいい感じに手助けしてくれるパッケージを使って、チャット機能を作っていきます。

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