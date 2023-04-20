---
title: "【Flutter】【Riverpod】NotifierProviderでdispose()したい"
emoji: "🗑️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter,Dart,Riverpod,NotifierProvider]
published: true
---

ChangeNotifierProviderやStateNotifierProviderを使用する場合は、以下のようにcontrollerの破棄をしていたが、NotifierProviderの場合は、この書き方ではdispose()を実行できない。

```dart
@override
void dispose() {
  super.dispose();
  _controller.dispose();
}
```


# 結論
NotifierProviderの場合は、build()内で"ref.onDispose()"を実装すれば、上記と同じようにcontroller等を破棄することができる。

```dart
@override
build() {
    ref.onDispose(() {
        // Provider破棄のタイミングで、実行したい処理を記載
        _controller.dispose();
    });
}
```

<!-- ※【build()とは？】 -->
<!-- buildについての説明 -->
<!-- build()は、Providerの実装に必要なメソッドで、Providerの状態を返すメソッド。 -->
<!-- build()は、Providerの状態が変更された場合に、再度実行される。 -->


# 挙動確認
サンプルでは、遷移前の画面に戻り、Providerが破棄されたタイミングで"ref.onDispose()"が実行されることが、デバックコンソールで確認できる。
NotifierProviderの定義は、riverpod_generatorを使用して作成。

![](https://storage.googleapis.com/zenn-user-upload/945de09e8dc8-20230414.gif =250x)


【環境】
:::: details flutetr doctor
```text
[✓] Flutter (Channel stable, 3.7.10, on macOS 13.2.1 22D68 darwin-arm64, locale ja-JP)
[✓] Android toolchain - develop for Android devices (Android SDK version 33.0.2)
[✓] Xcode - develop for iOS and macOS (Xcode 14.2)
[✓] Chrome - develop for the web
[✓] Android Studio (version 2022.1)
[✓] VS Code (version 1.77.1)
[✓] Connected device (3 available)
[✓] HTTP Host Availability
```
::::

【ファイル構成】
```yaml
project(root)
├─ lib
│  ├─ main.dart
│  ├─ notifier_sample.dart
│  ├─ provider.dart
│  └─ provider.g.dart
└─ pubspec.yaml
```

```dart: main.dart
// 定番の部分は省略
// 画面遷移するだけのPage
class MyHomePage extends StatelessWidget {
  const MyHomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('NotifierProvider Sample'),
      ),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            Navigator.push(
              context,
              MaterialPageRoute(builder: (context) => const NotifierSample()),
            );
          },
          child: const Text('Notifier Sample'),
        ),
      ),
    );
  }
}
```

```dart: notifier_sample.dart
// 単一のCheckBoxがあるだけのPage
class NotifierSample extends ConsumerWidget {
  const NotifierSample({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final isCheckValue = ref.watch(checkValueProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Notifier Sample'),
      ),
      body: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          isCheckValue ? const Text("ON") : const Text("OFF"),
          Center(
            child: Checkbox(
              value: isCheckValue,
              onChanged: (bool? value) {
                ref.read(checkValueProvider.notifier).updateCheckValue(value!);
              },
            ),
          ),
        ],
      ),
    );
  }
}
```
    
```dart: provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'provider.g.dart';

@riverpod
class CheckValue extends _$CheckValue {
  @override
  bool build() {
    ref.onDispose(() {
      print("dispose()");
    });

    print("build()");

    // stateの初期値はfalse
    return false;
  }

  void updateCheckValue(bool updateValue) {
    state = updateValue;
  }
}
```

```dart: provider.g.dart（riverpod_generatorで自動生成されたファイル）
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'provider.dart';

// **************************************************************************
// RiverpodGenerator
// **************************************************************************

String _$checkValueHash() => r'14f35606435e5fe538c0f3853fd4d29362e42a26';

/// See also [CheckValue].
@ProviderFor(CheckValue)
final checkValueProvider =
    AutoDisposeNotifierProvider<CheckValue, bool>.internal(
  CheckValue.new,
  name: r'checkValueProvider',
  debugGetCreateSourceHash:
      const bool.fromEnvironment('dart.vm.product') ? null : _$checkValueHash,
  dependencies: null,
  allTransitiveDependencies: null,
);

typedef _$CheckValue = AutoDisposeNotifier<bool>;
// ignore_for_file: unnecessary_raw_strings, subtype_of_sealed_class, invalid_use_of_internal_member, do_not_use_environment, prefer_const_constructors, public_member_api_docs, avoid_private_typedef_functions
```

```yaml : pubspec.yaml
  riverpod_generator: ^2.1.6
  riverpod_annotation: ^2.0.4
  hooks_riverpod: ^2.3.4
  build_runner: ^2.3.3
```



# 参考
https://docs-v2.riverpod.dev/docs/concepts/modifiers/auto_dispose#example-canceling-http-requests-when-no-longer-used
https://github.com/rrousselGit/riverpod/issues/1858
https://github.com/rrousselGit/riverpod/issues/2309
