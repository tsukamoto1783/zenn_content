---
title: "【Flutter】【Riverpod】NotifierProviderでdispose()したい"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter,Dart,Riverpod,NotifierProvider]
published: false
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
NotifierProviderの場合は、build()内で、ref.onDispose()を実装すれば、従来と同じようにProvider破棄のタイミングで、controller等を破棄することができる。

```dart
@override
build() {
    ref.onDispose(() {
        // Provider破棄のタイミングで、実行したい処理をここ記載
    });
}
```

※【build()とは？】
<!-- buildについての説明 -->
build()は、Providerの実装に必要なメソッドで、Providerの状態を返すメソッド。
build()は、Providerの状態が変更された場合に、再度実行される。



# 挙動確認
【環境】
:::: details :flutetr doctor
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

【サンプルアプリのファイル構成】
```yaml
project(root)
├─ lib
│  ├─ main.dart
│  ├─ samplePage.dart
│  └─ notifierProvider.dart
└─ pubspec.yaml
```

# 


# 参考
https://docs-v2.riverpod.dev/docs/concepts/modifiers/auto_dispose#example-canceling-http-requests-when-no-longer-used
https://github.com/rrousselGit/riverpod/issues/1858
https://github.com/rrousselGit/riverpod/issues/1420
https://github.com/rrousselGit/riverpod/issues/1929
https://github.com/rrousselGit/riverpod/issues/2309
