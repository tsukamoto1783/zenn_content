---
title: "【Flutter】スクロールの挙動をカスタマイズする"
emoji: "📜"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, dart, scrollBehavior, scrollPhysics, ScrollConfiguration]
published: true
---

スクロール系のWidgetで使用するスクロール挙動をカスタマイズする。

:::: details シンプルなサンプルコード
```dart: main.dart
void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(),
    );
  }
}

class MyHomePage extends StatelessWidget {
  const MyHomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: const Text('sample app'),
      ),
      body: Center(
        child: ListView(
          children: [
            for (int i = 0; i < 15; i++)
              const ListTile(
                title: Text('sample item'),
                subtitle: Text('sample subtitle'),
                leading: Icon(Icons.ac_unit),
                trailing: Icon(Icons.arrow_forward_ios),
              ),
          ],
        ),
      ),
    );
  }
}
```
::::

<br>

iosとandroidでデフォルトの挙動は以下。


|                                         ios                                          |                                       android                                        |                               android（useMaterial3）                                |
| :----------------------------------------------------------------------------------: | :----------------------------------------------------------------------------------: | :----------------------------------------------------------------------------------: |
| ![](https://storage.googleapis.com/zenn-user-upload/734a28711a65-20230825.gif =300x) | ![](https://storage.googleapis.com/zenn-user-upload/5e0d85ceeb26-20230822.gif =300x) | ![](https://storage.googleapis.com/zenn-user-upload/cbd20ad608b6-20230822.gif =270x) |
|                                                                                      |


<br>

# どうやってカスタマイズする？
アプリ全体で適用させたい場合は、MaterialAppのbuilderに`ScrollConfiguration`を設定する。

```dart: main.dart
void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      // 以下追加 
      builder: (context, child) {
        return ScrollConfiguration(
          behavior: const CustomScrollBehavior(),
          child: child!,
        );
      },
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(),
    );
  }
}
// ========================================
class CustomScrollBehavior extends ScrollBehavior {
  const CustomScrollBehavior();

  // ここにカスタマイズしたい挙動を記述する
}
```

各Widget個別で適用させたい場合は、それぞれでScrollConfigurationを設定する。

```dart
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('sample app'),
      ),
      body: Center(
        // 以下追加
        child: ScrollConfiguration(
          behavior: const CustomScrollBehavior(),
          child: ListView(
            children: [
                // 省略
```

<br>

# 引っ張り時のエフェクトを無効にする
以下の設定で、ios,android共に引っ張り時のエフェクトを全て無効にできる。

```dart
class CustomScrollBehavior extends ScrollBehavior {
  const CustomScrollBehavior();

  @override
  Widget buildOverscrollIndicator(
      BuildContext context, Widget child, ScrollableDetails details) {
    return child;
  }

  @override
  ScrollPhysics getScrollPhysics(BuildContext context) =>
      const ClampingScrollPhysics();
}
```

<br>

# 各種設定
エフェクト無効が一番使用するかとは思うが、それぞれの設定で何ができるかも確認してみる。

<br>

## getScrollPhysics
ひっぱり時のスクロール動作を設定できる。
冒頭で各OSでの挙動を載せている通り、デフォルトでは以下の設定になっている。
- ios: BouncingScrollPhysics
- android: ClampingScrollPhysics

```dart: scroll_configuration.dart
  static const ScrollPhysics _bouncingPhysics =
      BouncingScrollPhysics(parent: RangeMaintainingScrollPhysics());

  static const ScrollPhysics _bouncingDesktopPhysics = BouncingScrollPhysics(
      decelerationRate: ScrollDecelerationRate.fast,
      parent: RangeMaintainingScrollPhysics());
  
  static const ScrollPhysics _clampingPhysics =
      ClampingScrollPhysics(parent: RangeMaintainingScrollPhysics());

  /// The scroll physics to use for the platform given by [getPlatform].
  ///
  /// Defaults to [RangeMaintainingScrollPhysics] mixed with
  /// [BouncingScrollPhysics] on iOS and [ClampingScrollPhysics] on
  /// Android.
  ScrollPhysics getScrollPhysics(BuildContext context) {
    // When modifying this function, consider modifying the implementation in
    // the Material and Cupertino subclasses as well.
    switch (getPlatform(context)) {
      case TargetPlatform.iOS:
        return _bouncingPhysics;
      case TargetPlatform.macOS:
        return _bouncingDesktopPhysics;
      case TargetPlatform.android:
      case TargetPlatform.fuchsia:
      case TargetPlatform.linux:
      case TargetPlatform.windows:
        return _clampingPhysics;
    }
  }
```

以下のように指定すると、androidでもiosとおなじ引っ張り（バンジー）エフェクトになる。

```dart
class CustomScrollBehavior extends ScrollBehavior {
  const CustomScrollBehavior();

  // 以下追加
  @override
  ScrollPhysics getScrollPhysics(BuildContext context) =>
      const BouncingScrollPhysics();
}
```
※他にも`NeverScrollableScrollPhysics`や`AlwaysScrollableScrollPhysics`などもある。

<br>

## buildOverscrollIndicator
`GlowingOverscrollIndicator`を使用することで、androidの引っ張り時の色付き波線みたいな効果がカスタム or iosでも有効にできる。
（無効にする or 色の変更以外にカスタムしたい場面はそうそう無いかと思うが）

ソースからも分かるように、デフォルトだとandroidで有効になっている。
```dart: scroll_configuration.dart
  /// Applies a [GlowingOverscrollIndicator] to the child widget on
  /// [TargetPlatform.android] and [TargetPlatform.fuchsia].
  Widget buildOverscrollIndicator(
      BuildContext context, Widget child, ScrollableDetails details) {
    // When modifying this function, consider modifying the implementation in
    // the Material and Cupertino subclasses as well.
    switch (getPlatform(context)) {
      case TargetPlatform.iOS:
      case TargetPlatform.linux:
      case TargetPlatform.macOS:
      case TargetPlatform.windows:
        return child;
      case TargetPlatform.android:
        switch (androidOverscrollIndicator) {
          case AndroidOverscrollIndicator.stretch:
            return StretchingOverscrollIndicator(
              axisDirection: details.direction,
              child: child,
            );
          case AndroidOverscrollIndicator.glow:
            break;
        }
      case TargetPlatform.fuchsia:
        break;
    }
    return GlowingOverscrollIndicator(
      axisDirection: details.direction,
      color: _kDefaultGlowColor,
      child: child,
    );
  }
```

以下のように指定すると、iosでも波線を表示できる。

```dart: main.dart
class CustomScrollBehavior extends ScrollBehavior {
  const CustomScrollBehavior();

  @override
  Widget buildOverscrollIndicator(
      BuildContext context, Widget child, ScrollableDetails details) {
    return GlowingOverscrollIndicator(
      axisDirection: details.direction,
      color: Colors.green,
      child: child,
    );
  }

  // ========== 波線効果を無効にする場合は以下 ==========
  @override
  Widget buildOverscrollIndicator(
      BuildContext context, Widget child, ScrollableDetails details) {
    return child;
  }
}
```

![](https://storage.googleapis.com/zenn-user-upload/c0df14ba8424-20230824.gif =300x)

<br>

## buildScrollbar
スクロール系のWidgetにスクロールバーを表示するかどうかを設定できる。
アプリ全体でスクロールバーを表示するデザインなら、MaterialAppで統一して設定するのもおすすめ。

デフォルトだとios、android共に非表示で、desktop系のみに有効となっている。

```dart: scroll_configuration.dart
  /// Applies a [RawScrollbar] to the child widget on desktop platforms.
  Widget buildScrollbar(
      BuildContext context, Widget child, ScrollableDetails details) {
    // When modifying this function, consider modifying the implementation in
    // the Material and Cupertino subclasses as well.
    switch (getPlatform(context)) {
      case TargetPlatform.linux:
      case TargetPlatform.macOS:
      case TargetPlatform.windows:
        assert(details.controller != null);
        return RawScrollbar(
          controller: details.controller,
          child: child,
        );
      case TargetPlatform.android:
      case TargetPlatform.fuchsia:
      case TargetPlatform.iOS:
        return child;
    }
  }
```



以下、カスタム例。
    
```dart: main.dart
class CustomScrollBehavior extends ScrollBehavior {
  const CustomScrollBehavior();

  @override
  Widget buildScrollbar(
      BuildContext context, Widget child, ScrollableDetails details) {
    return RawScrollbar(
      controller: details.controller,
      padding: const EdgeInsets.all(12),
      thumbColor: Colors.grey,
      thickness: 8,
      radius: const Radius.circular(8),
      thumbVisibility: true,
      child: child,
    );
  }
}
```

![](https://storage.googleapis.com/zenn-user-upload/cce529fc06fe-20230825.gif =300x)

<br>

## copyWith
お馴染みの便利なcopyWithも用意されている。

## getPlatform
現在のプラットフォームを取得できる。
overrideしてカスタムすることはあまり無いと思うので、このメソッドを使用してos毎の設定したい場合に使用。

## velocityTrackerBuilder
スクロール速度を検知して、速度によって動作を切り替るために使用。
（深掘っていくと時間かかる部分なので記述割愛）
