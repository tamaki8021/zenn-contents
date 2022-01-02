---
title: "【Flutter】アプリ内にAppleの地図を表示する"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter", "ios", "map", "dart"]
published: true
---

# 概要

flutter では google 公式が出している[google_maps_flutter](https://pub.dev/packages/google_maps_flutter)があり、検索するとほとんど google マップ関連の導入記事が溢れています。

しかし、google マップのパッケージを使う場合 API の登録・取得といった、面倒くささや API の期限切れなど管理のコストなどもかかります。
Android の場合はそうした登録を経て、google マップを導入するんですが、まずは iOS 版をリリースするしたい場合、Apple のマップを使ったほうが APIkey の取得といった煩わしさや利用料金などと考えなくてもいいということがあります。

今回の記事ではそうした導入コストをかけずにマップを導入する方法を紹介していきます。

## 想定読者

- まずは iOS をリリースすることを目標としている人
- APIkey の登録・取得といったことを避けたい人
- 利用料金などを考えたくない人

## パッケージ紹介

**Apple Maps Widget を提供する Flutter のパッケージ**
[apple_maps_flutter](https://pub.dev/packages/apple_maps_flutter#sample-usage)

**Google Maps Widget を提供する Flutter のパッケージ**
[google_maps_flutter](https://pub.dev/packages/google_maps_flutter)

**Android と iOS の両デバイスにネイティブマップを提供する Flutter のパッケージ**
[platform_maps_flutter](https://pub.dev/packages/platform_maps_flutter)

今回は将来的に両デバイスにマップを導入することを想定し、3 番目の[platform_maps_flutter](https://pub.dev/packages/platform_maps_flutter)パッケージを使った実装を紹介します。

# 環境

```
>flutter doctor -v
[✓] Flutter (Channel stable, 2.5.3, on macOS 11.4 20F71 darwin-x64, locale ja-JP)
    • Flutter version 2.5.3 at /Users/tamakikyou/fvm/versions/2.5.3
    • Upstream repository https://github.com/flutter/flutter.git
    • Framework revision 18116933e7 (3 months ago), 2021-10-15 10:46:35 -0700
    • Engine revision d3ea636dc5
    • Dart version 2.14.4

[✓] Android toolchain - develop for Android devices (Android SDK version 31.0.0)
    • Android SDK at /Users/tamakikyou/Library/Android/sdk
    • Platform android-31, build-tools 31.0.0
    • Java binary at: /Applications/Android Studio.app/Contents/jre/Contents/Home/bin/java
    • Java version OpenJDK Runtime Environment (build 11.0.10+0-b96-7281165)
    • All Android licenses accepted.

[✓] Xcode - develop for iOS and macOS
    • Xcode at /Applications/Xcode.app/Contents/Developer
    • Xcode 13.1, Build version 13A1030d
    • CocoaPods version 1.10.2
```

# 実装

## パッケージ追加

```yaml: pubpec.yaml
dependencies:
  platform_maps_flutter: ^1.0.2
```

## Info.plist の編集

```plist: Info.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <!-- ... -->

  // platformViewを使うためのフラグ 指定しなくても良いかも？
  <!-- <key>io.flutter.embedded_views_preview</key>
  <true/> -->

  //位置情報を使いますというユーザー向けの説明文
  <key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
  <string>This app requires user’s location for better user experience.</string>
</dict>
</plist>
```

## PlatformMap Widget

```dart
  class MyHomePage extends StatefulWidget {
    MyHomePage({Key? key}) : super(key: key);

    @override
    _MyHomePageState createState() => _MyHomePageState();
  }

  class HomePage extends StatelessWidget {
    @override
    Widget build(BuildContext context) {
      return Scaffold(
        body: SizedBox(
              height: 300,
              child: PlatformMap(
                // マップのカメラの初期値
                initialCameraPosition: const CameraPosition(
                  // 地理情報
                  target: LatLng(47.6, 8.8796),
                  zoom: 16,
                ),

                markers: <Marker>{
                  Marker(
                    // マーカーID リストの中で一意であればいい
                    markerId: MarkerId('marker_1'),
                    // 地理情報
                    position: const LatLng(47.6, 8.8796),
                    // タップイベント
                    // デフォルトのタップ処理では、マップをマーカーの中央に配置し、マーカーを表示します。情報ウィンドウを表示します。
                    consumeTapEvents: true,
                    // 情報ウィンドウのエキストラベル
                    infoWindow: const InfoWindow(
                      title: 'PlatformMarker',
                      snippet: "Hi I'm a Platform Marker",
                    ),
                    onTap: () {
                      print("Marker tapped");
                    },
                  ),
                },
                // 現在地のレイヤーを表示 
                myLocationEnabled: true,
                // マイロケーションボタンの有効・無効を設定します。マイロケーションボタンは、ユーザーの位置が移動するようにカメラを移動させます。
                myLocationButtonEnabled: true,
                // [GoogleMap] がタップされる度に呼び出されます。 
                onTap: (location) => print('onTap: $location'),
                // カメラが移動し続けるたびに呼び出される
                onCameraMove: (cameraUpdate) =>
                    print('onCameraMove: $cameraUpdate'),
                // 地図が回転したときにコンパスを表示
                compassEnabled: true,
                // マップが使用可能になったときのコールバック、メソッド
                onMapCreated: (controller) {
                  Future<dynamic>.delayed(const Duration(seconds: 2)).then(
                    (dynamic _) {
                      controller.animateCamera(
                        CameraUpdate.newCameraPosition(
                          const CameraPosition(
                            bearing: 270.0,
                            target: LatLng(51.5160895, -0.1294527),
                            tilt: 30.0,
                            zoom: 18,
                          ),
                        ),
                      );
                    },
                  );
                },
              ),
            ),
      );
    }
  }
```

## 原文

```dart
class PlatformMap extends StatefulWidget {
  const PlatformMap({
    Key? key,
    required this.initialCameraPosition,
    this.onMapCreated,
    this.gestureRecognizers = const <Factory<OneSequenceGestureRecognizer>>{},
    this.compassEnabled = true,
    this.mapType = MapType.normal,
    this.minMaxZoomPreference = MinMaxZoomPreference.unbounded,
    this.rotateGesturesEnabled = true,
    this.scrollGesturesEnabled = true,
    this.zoomControlsEnabled = true,
    this.zoomGesturesEnabled = true,
    this.tiltGesturesEnabled = true,
    this.myLocationEnabled = false,
    this.myLocationButtonEnabled = false,
    this.padding = const EdgeInsets.all(0),
    this.trafficEnabled = false,
    this.markers = const <Marker>{},
    this.polygons = const <Polygon>{},
    this.polylines = const <Polyline>{},
    this.circles = const <Circle>{},
    this.onCameraMoveStarted,
    this.onCameraMove,
    this.onCameraIdle,
    this.onTap,
    this.onLongPress,
  }) : super(key: key);

  /// Callback method for when the map is ready to be used.
  ///
  /// Used to receive a [GoogleMapController] for this [GoogleMap].
  final MapCreatedCallback? onMapCreated;

  /// The initial position of the map's camera.
  final CameraPosition initialCameraPosition;

  /// True if the map should show a compass when rotated.
  final bool compassEnabled;

  /// Type of map tiles to be rendered.
  final MapType mapType;

  /// Preferred bounds for the camera zoom level.
  ///
  /// Actual bounds depend on map data and device.
  final MinMaxZoomPreference minMaxZoomPreference;

  /// True if the map view should respond to rotate gestures.
  final bool rotateGesturesEnabled;

  /// True if the map view should respond to scroll gestures.
  final bool scrollGesturesEnabled;

  /// True if the map view should show zoom controls. This includes two buttons
  /// to zoom in and zoom out. The default value is to show zoom controls.
  ///
  /// This is only supported on Android. And this field is silently ignored on iOS.
  final bool zoomControlsEnabled;

  /// True if the map view should respond to zoom gestures.
  final bool zoomGesturesEnabled;

  /// True if the map view should respond to tilt gestures.
  final bool tiltGesturesEnabled;

  /// Padding to be set on map. See https://developers.google.com/maps/documentation/android-sdk/map#map_padding for more details.
  final EdgeInsets padding;

  /// Markers to be placed on the map.
  final Set<Marker> markers;

  /// Polygons to be placed on the map.
  final Set<Polygon> polygons;

  /// Polylines to be placed on the map.
  final Set<Polyline> polylines;

  /// Circles to be placed on the map.
  final Set<Circle> circles;

  /// Called when the camera starts moving.
  ///
  /// This can be initiated by the following:
  /// 1. Non-gesture animation initiated in response to user actions.
  ///    For example: zoom buttons, my location button, or marker clicks.
  /// 2. Programmatically initiated animation.
  /// 3. Camera motion initiated in response to user gestures on the map.
  ///    For example: pan, tilt, pinch to zoom, or rotate.
  final VoidCallback? onCameraMoveStarted;

  /// Called repeatedly as the camera continues to move after an
  /// onCameraMoveStarted call.
  ///
  /// This may be called as often as once every frame and should
  /// not perform expensive operations.
  final CameraPositionCallback? onCameraMove;

  /// Called when camera movement has ended, there are no pending
  /// animations and the user has stopped interacting with the map.
  final VoidCallback? onCameraIdle;

  /// Called every time a [GoogleMap] is tapped.
  final ArgumentCallback<LatLng>? onTap;

  /// Called every time a [GoogleMap] is long pressed.
  final ArgumentCallback<LatLng>? onLongPress;

  /// True if a "My Location" layer should be shown on the map.
  ///
  /// This layer includes a location indicator at the current device location,
  /// as well as a My Location button.
  /// * The indicator is a small blue dot if the device is stationary, or a
  /// chevron if the device is moving.
  /// * The My Location button animates to focus on the user's current location
  /// if the user's location is currently known.
  ///
  /// Enabling this feature requires adding location permissions to both native
  /// platforms of your app.
  /// * On Android add either
  /// `<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />`
  /// or `<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />`
  /// to your `AndroidManifest.xml` file. `ACCESS_COARSE_LOCATION` returns a
  /// location with an accuracy approximately equivalent to a city block, while
  /// `ACCESS_FINE_LOCATION` returns as precise a location as possible, although
  /// it consumes more battery power. You will also need to request these
  /// permissions during run-time. If they are not granted, the My Location
  /// feature will fail silently.
  /// * On iOS add a `NSLocationWhenInUseUsageDescription` key to your
  /// `Info.plist` file. This will automatically prompt the user for permissions
  /// when the map tries to turn on the My Location layer.
  final bool myLocationEnabled;

  /// Enables or disables the my-location button.
  ///
  /// The my-location button causes the camera to move such that the user's
  /// location is in the center of the map. If the button is enabled, it is
  /// only shown when the my-location layer is enabled.
  ///
  /// By default, the my-location button is enabled (and hence shown when the
  /// my-location layer is enabled).
  ///
  /// See also:
  ///   * [myLocationEnabled] parameter.
  final bool myLocationButtonEnabled;

  /// Enables or disables the traffic layer of the map
  final bool trafficEnabled;

  /// Which gestures should be consumed by the map.
  ///
  /// It is possible for other gesture recognizers to be competing with the map on pointer
  /// events, e.g if the map is inside a [ListView] the [ListView] will want to handle
  /// vertical drags. The map will claim gestures that are recognized by any of the
  /// recognizers on this list.
  ///
  /// When this set is empty, the map will only handle pointer events for gestures that
  /// were not claimed by any other gesture recognizer.
  final Set<Factory<OneSequenceGestureRecognizer>> gestureRecognizers;
  @override
  _PlatformMapState createState() => _PlatformMapState();
}
```