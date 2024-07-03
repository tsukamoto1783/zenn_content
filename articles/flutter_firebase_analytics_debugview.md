---
title: "【Flutter】Firebase AnalyticsのDebugViewの起動方法メモ"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, firebase, analytics, debugview]
published: false
# publication_name: ncdc
---

コード上に Firebase Analytics の DebugView の設定を実装した後に、普通に flutter 実行しても DebugView が起動しないことを知らずにやや詰まったためメモ。

基本はドキュメントに書いてある通りですが、一部詰まったとこを記載します。
https://firebase.google.com/docs/analytics/Debugview?hl=ja#enable_debug_mode

## android

### 1.zsh: command not found: adb

`adb shell setprop debug.firebase.analytics.app <PACKAGE_NAME>`を実行すると`zsh: command not found: adb`となる。
→ お馴染みのパスを通しましょう。

adb は Android SDK に含まれているみたいなので、Flutter 環境を問題なく使用しているならインストールされているはず。

私の環境だと以下にインストールされていたので、そのパスを記載。

```.zshrc
export PATH="$PATH:$HOME/Library/Android/sdk/platform-tools"
```

### 2.adb: more than one device/emulator

パスを通した後に`adb shell setprop debug.firebase.analytics.app <PACKAGE_NAME>`を実行すると`adb: more than one device/emulator`となる。
→ デバイスが複数接続されている場合は、デバイスを指定してあげましょう。

![](https://storage.googleapis.com/zenn-user-upload/596269e4364b-20240703.png)
_↑ デバイス選択で確認すると android 端末が二つ存在している_

`adb devices`でデバイス一覧を確認して、DebugView を有効化したいデバイスに対して、`adb -s <DEVICE_ID> shell setprop debug.firebase.analytics.app <PACKAGE_NAME>`を実行。
![](https://storage.googleapis.com/zenn-user-upload/f7dd930edfe2-20240703.png)
_↑adb devices 実行するとデバイス ID が取得できる_

上記で設定した後に、実行すると DebugView が起動します。

DebugView を終了（無効化）する場合は、`adb shell setprop debug.firebase.analytics.app .none.`を実行。

## iOS

ドキュメントや以下の記事を参考に`-FIRDebugEnabled`を Xcode で設定してみても、DebugView が起動しない。シミュレーターと実機で試したがどちらも。

ここは引き続き調査。解決し次第追記予定。

iOS 設定参考記事：
https://dev.classmethod.jp/articles/check-firebase-analytics-events-sent-from-the-flutter-ios-app-in-real-time-with-debug-view/
https://qiita.com/tsururoll20201004/items/5079145b091d4a13b432
