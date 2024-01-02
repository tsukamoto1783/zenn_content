---
title: "【Flutter】mockitoでclassのstaticメソッドをモック化する"
emoji: "🔧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter,dart,mockito,static,test]
published: true
---

staticメソッドをmockitoでモック化する方法がわからなかったので調べてみた。

<br>

### 環境
```txt: flutter doctor
[✓] Flutter (Channel stable, 3.13.2, on macOS 14.0 23A344 darwin-arm64, locale ja-JP)
[✓] Android toolchain - develop for Android devices (Android SDK version 33.0.2)
[✓] Xcode - develop for iOS and macOS (Xcode 15.0)
[✓] Chrome - develop for the web
[✓] Android Studio (version 2022.1)
[✓] VS Code (version 1.85.1)
[✓] Connected device (2 available)
[✓] Network resources

• No issues found!
```
```yaml: pubspec.yaml
dependencies:
  mockito: ^5.4.4
  build_runner: ^2.4.7
  fluttertoast: ^8.2.4
  dio: ^5.4.0
```

<br>

## staticではないclassメソッドをmockitoでモック化する
まずはstaticではないclassメソッドをモック化してテストする。
ここでは自作のCatクラスと、外部ライブラリの[Dio](https://pub.dev/packages/dio)クラスをモック化してテストする。

以下は意図した通り正常にモック化してtestが通る。

```dart
import 'package:dio/dio.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';

import 'static_normal_test.mocks.dart';

class Cat {
  String sound() => "Meow";
}

// 自作のCatクラスをモック化
@GenerateNiceMocks([MockSpec<Cat>()])

// 外部ライブラリのDioクラスをモック化
@GenerateNiceMocks([MockSpec<Dio>()])

void main() {
  test('Cat method test', () {
    final cat = MockCat();

    cat.sound();

    // 指定の処理が呼び出されかどうかを確認
    verify(cat.sound()); // test ok
  });

  test('Dio method test', () {
    final dio = MockDio();

    when(dio.get("")).thenAnswer(
      (_) async => Response(requestOptions: RequestOptions()),
    );
    dio.get("");

    // 指定の処理が呼び出されかどうかを確認
    verify(dio.get("")); // test ok
  });
}
```

<br>

## staticなclassメソッドをmockitoでモック化する
staticなclassメソッドをモック化してみる。
自作のCatクラスと、外部ライブラリの[Fluttertoast](https://pub.dev/packages/fluttertoast)クラスをモック化してテストする。

テスト実施しようとすると、実行する以前に以下のように`そんなメソッドは存在しない`と怒られる。
`build_runner`で生成されたファイルを見てみても、staticメソッドはモック化されていないことが確認できる。

そもそもstaticなclassメソッドの場合はインスタンスを生成せずに直接呼び出すメソッドなので、当然モック化もできない。

>The method 'sound' isn't defined for the type 'MockCat'.
Try correcting the name to the name of an existing method, or defining a method named 'sound'.

> The method 'toast' isn't defined for the type 'MockFluttertoast'.
Try correcting the name to the name of an existing method, or defining a method named 'toast'.


```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:fluttertoast/fluttertoast.dart';
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';

import 'static_ng_test.mocks.dart';

class Cat {
  static sound() => "Meow";
}

// モック化
// → staticメソッドはモック化されない
@GenerateNiceMocks([MockSpec<Cat>()])
@GenerateNiceMocks([MockSpec<Fluttertoast>()])

void main() {
  test('Cat static method test', () {
    final cat = MockCat();

    cat.sound(); // ←error

    // 指定の処理が呼び出されかどうかを確認
    verify(cat.sound()); // ←error
  });

  test('Fluttertoast static method test', () {
    final toast = MockFluttertoast();

    toast.toast(); // ←error

    // 指定の処理が呼び出されかどうかを確認
    verify(toast.toast()); // ←error
  });
}
```

<br>

## staticなclassメソッドがmockitoでテスト通るようにモック化する
staticなclassメソッドをモック化するには、自作のラッパークラスを作成してあげるとモック化できるようになる。

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:fluttertoast/fluttertoast.dart';
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';

import 'static_ok_test.mocks.dart';

class Cat {
  static sound() => "Meow";
}

// ラッパークラスを作成
class CatWrapper {
  String soundWrapper() => Cat.sound();
}

// ラッパークラスを作成
class FluttertoastWrapper {
  Future<bool?> showToastWrapper() async =>
      await Fluttertoast.showToast(msg: "test");
}

// CatWrapperクラスをモック化
@GenerateNiceMocks([MockSpec<CatWrapper>()])

// FluttertoastWrapperクラスをモック化
@GenerateNiceMocks([MockSpec<FluttertoastWrapper>()])

void main() {
  test('Cat static method test', () {
    final cat = MockCatWrapper();

    cat.soundWrapper();

    // 指定の処理が呼び出されかどうかを確認
    verify(cat.soundWrapper()); // test ok
  });

  test('Fluttertoast static method test', () {
    final toast = MockFluttertoastWrapper();

    toast.showToastWrapper();

    // 指定の処理が呼び出されかどうかを確認
    verify(toast.showToastWrapper()); // test ok
  });
}
```

<br>

### 参考
https://github.com/dart-lang/mockito/issues/214
