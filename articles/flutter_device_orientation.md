---
title: "【Flutter】デバイスの向きを検知しよう"
emoji: "🤳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, dart, flutterDeviceOrientation]
published: true
---
[native_device_orientation](https://pub.dev/packages/native_device_orientation)パッケージを使用すると、デバイスの向き検知することができます。

以下シンプルな動作サンプルです。
デバイスの向きによって画面の文字が変わっているのが確認できます。

※動画の手ぶれに関してご愛嬌願います。。

![](https://storage.googleapis.com/zenn-user-upload/edddab4d7414-20230610.gif =200x)

&nbsp;
# どんな時に使う？
ほとんどのアプリではデバイスの向きまでは気にしないが、カメラを使用する場合など、どの方向に回転しているか判定したいことがあるかと思います。

通常のFlutterアプリでは、`MediaQuery.of(context).orientation` 等を使用するとデバイスの縦横を検知することができます。

しかし、これらはデバイスが縦か横かを判定するだけで、右向きなのか左向きなのかまでは判定することができません。

そういった時にこのライブラリを使用すると、デバイスの向きを検知することができます。

ライブラリの使用パターンは大きくは三つあります。
- `NativeDeviceOrientationReader`
- `NativeDeviceOrientedWidget`
- `NativeDeviceOrientationCommunicator`


&nbsp;
# NativeDeviceOrientationReader 
useSensorというプロパティがあり、これをtrueに設定するとデバイスのセンサーが直接使用され、デバイスの向きが検知されます。

デバイスの向きが変わる度に、`NativeDeviceOrientationReader` Widgetの`builder`が呼ばれ、**デバイスの回転情報を取得する**ことができます。

取得したデバイスの回転情報を基に、表示するWidgetを変更したり、デバイスの向きそれぞれに対して独自の処理を実行させることができます。

```dart
// 省略
Widget customText(String text) {
    return Text(text,
        style: const TextStyle(
        fontSize: 50,
        fontWeight: FontWeight.bold,
        ));
}

return Scaffold(
    body: Center(
        child: NativeDeviceOrientationReader(
            builder: (context) {
            // デバイスの向き情報を取得
            NativeDeviceOrientation orientation =
                NativeDeviceOrientationReader.orientation(context);

            /// それぞれの向きに応じたWidgetを表示したり、向きに応じた処理を実行できる
            if (orientation == NativeDeviceOrientation.portraitUp) {
                print("portrait Up");
                return customText("portrait Up");
            } else if (orientation == NativeDeviceOrientation.portraitDown) {
                print("portrait Down");
                return customText("portrait Down");
            } else if (orientation == NativeDeviceOrientation.landscapeLeft) {
                print("landscape Left");
                return customText("landscape Left");
            } else if (orientation ==
                NativeDeviceOrientation.landscapeRight) {
                print("landscape Right");
                return customText("landscape Right");
            } else {
                print("unknown");
                return customText("unknown");
            }
            },
            useSensor: true,
        ),
    ),
);
```

&nbsp;
# NativeDeviceOrientedWidget
基本的には上記の`NativeDeviceOrientationReader`と同じですが、以下二点が異なります。

1. Widgetを再描画させたい場合、自前で分岐ロジックを書く必要がない。
`NativeDeviceOrientationReader`だとデバイスの向き毎に分岐ロジックを自前で書く必要があったが、`NativeDeviceOrientedWidget`だと予めデバイスの向き毎のプロパティが用意されているので、簡潔に書くことができます。

```dart
// 省略
Widget customWidget(String text, Color color) {
    return Container(
    padding: const EdgeInsets.all(20),
    color: color,
    child: Text(text,
        style: const TextStyle(
            fontSize: 50,
            fontWeight: FontWeight.bold,
        )),
    );
}

return Scaffold(
    body: Center(
        /// それぞれの向きに応じたWidgetを表示したり、向きに応じた処理を実行できる
        child: NativeDeviceOrientedWidget(
            portraitUp: (context) => customWidget("portrait Up", Colors.red),
            landscapeLeft: (context) =>
                customWidget("landscape Left", Colors.blue),
            landscapeRight: (context) {
            print("landscape Right");
            return customWidget("landscape Right", Colors.green);
            },
            fallback: (context) => customWidget("fallback", Colors.yellow),
            useSensor: true,
        ),
    ),
);
```

&nbsp;
2. デバイスの向き情報は取得できない
`NativeDeviceOrientedWidget`は向き毎のWidgetを再描画するためのWidgetなので、デバイスの向き情報を取得することはできません。
取得したデバイスの向き情報を基に、Widgetの描画に関係のない処理群だけを実行したい場合は、`NativeDeviceOrientationReader`や、後述の`NativeDeviceOrientationCommunicator` を使用する必要があります。

```dart :"NativeDeviceOrientationReader"の場合
return Scaffold(
    body: Center(
        child: NativeDeviceOrientationReader(
            builder: (context) {
            NativeDeviceOrientation orientation =
                NativeDeviceOrientationReader.orientation(context);

            if (orientation == NativeDeviceOrientation.portraitUp) {
            // Widgetの描画とは関係のない処理だけをすることも可能
            } 
            useSensor: true,
            }
            return Container();
        ),
    ),
);
```

```dart :"NativeDeviceOrientedWidget"の場合
return Scaffold(
    body: Center(
        child: NativeDeviceOrientedWidget(
            landscapeRight: (context) {
            /// 向きそれぞれに応じた処理も指定できるが、必ずWidgetを返す必要あり
            return customWidget("landscape Right", Colors.green);
            },
            fallback: (context) => customWidget("fallback", Colors.yellow),
            useSensor: true,
        ),
    ),
);
```

&nbsp;
# NativeDeviceOrientationCommunicator
上記の２パターンよりもより詳細に制御したい場合に使用します。

上記の２パターンはWidgetのcontext内でのみ利用できたが、`NativeDeviceOrientationCommunicator`はアプリケーション全体で利用できるため、どこからでもデバイスの向き情報を取得/共有することができます。

::::details サンプルコード
```dart
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:native_device_orientation/native_device_orientation.dart';

class NativeDeviceOrientationCommunicatorSample extends StatefulWidget {
  const NativeDeviceOrientationCommunicatorSample({super.key});

  @override
  State<NativeDeviceOrientationCommunicatorSample> createState() =>
      _NativeDeviceOrientationCommunicatorSampleState();
}

class _NativeDeviceOrientationCommunicatorSampleState
    extends State<NativeDeviceOrientationCommunicatorSample> {
  late StreamSubscription<NativeDeviceOrientation> subscription;
  NativeDeviceOrientation _currentOrientation = NativeDeviceOrientation.unknown;

  @override
  void initState() {
    super.initState();
    subscription = NativeDeviceOrientationCommunicator()
        .onOrientationChanged(
            useSensor: true,
            defaultOrientation: NativeDeviceOrientation.unknown)
        .listen((orientation) {
      setState(() {
        _currentOrientation = orientation;
      });
    });
  }

  @override
  void dispose() {
    subscription.cancel();
    super.dispose();
  }

  Widget customWidget(String text, Color color) {
    return Container(
      padding: const EdgeInsets.all(20),
      color: color,
      child: Text(text,
          style: const TextStyle(
            fontSize: 50,
            fontWeight: FontWeight.bold,
          )),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.secondary,
        title: const Text("device orientation test"),
      ),
      body: Center(
        child: _buildOrientationDependentWidget(),
      ),
    );
  }

  Widget _buildOrientationDependentWidget() {
    switch (_currentOrientation) {
      case NativeDeviceOrientation.portraitUp:
        return customWidget("portrait Up", Colors.red);
      case NativeDeviceOrientation.landscapeLeft:
        return customWidget("landscape Left", Colors.blue);
      case NativeDeviceOrientation.landscapeRight:
        return customWidget("landscape Right", Colors.green);
      default:
        return customWidget("fallback", Colors.yellow);
    }
  }
}
```
::::

# 備考
READMEには記載されていないが[パッケージのソース](https://github.com/rmtmckenzie/flutter_native_device_orientation/blob/master/lib/native_device_orientation.dart)を見ると、アプリケーションがバックグラウンドに移動したときやフォアグラウンドに戻ったときに、センサーのリスニングを一時停止または再開する`pause()`や`resume()`メソッドの記載もありました。

他にもREADMEに記載の無いメソッドもあるので、必要に応じてソースを確認してみてください。