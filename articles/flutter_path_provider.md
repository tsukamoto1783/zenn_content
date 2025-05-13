---
title: "【Flutter】アプリ内ストレージに書き込んだファイルをアプリ外から確認したい"
emoji: "💾"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, dart, pathProvider, storage]
published: true
publication_name: ncdc
---

[path_provider](https://pub.dev/packages/path_provider)ライブラリで、アプリ内ストレージにファイルを保存する際、
デバッグ時などにアプリ**外**から参照・確認したい場合はどうしたらいいのか、とふと思ったので調べてみました。

※ ここで検証として使用する処理は、以下の数行だけです。

```dart
final appDir = await getApplicationDocumentsDirectory(); // ← path_providerの関数
final file = File('${outputDir.path}/log.txt');
await file.writeAsString('ログの内容');
```

<br>

## path_provider

少しだけ[path_provider](https://pub.dev/packages/path_provider)についても触れておきます。

`path_provider`は、ファイルシステム上でよく使われる path を見つけるためのライブラリです。

OS によって使用できる関数があることは注意が必要です。

![](https://storage.googleapis.com/zenn-user-upload/4f48cd55013d-20250403.png)
_path_provider の readme から引用_

<br>

## Android

２つの関数の実行結果を例に見ていきます。

- getApplicationDocumentsDirectory() => `/data/user/0/<packageName>/app_flutter`
- getDownloadsDirectory() => `/storage/emulated/0/Android/data/<packageName>/files/downloads`

内部ストレージか外部ストレージかの違いはあるが、両者はどちらも`アプリ専門領域`であり、ユーザーからは見えません。（アクセスできません。）
※ （内部か外部かで、制限や容量などは若干異なりますが、ここでは割愛します。）

上記の `アプリ専用領域` に関しては、Android の場合は `AndroidStudio` を使用すれば確認することができます。

確認方法の手順は以下です。

1. Android Studio を開く
2. Header メニューから、View/ToolWindow/DeviceExplorer を選択
3. 取得したパスを辿り、該当ファイルを選択して表示 or ダウンロード

![](https://storage.googleapis.com/zenn-user-upload/18f6fa9eb695-20250403.png)

<br>

ユーザーがアクセスできる場所に保存したい場合は、アクセス可能なフォルダ階層を直接指定してあげると、アクセスは可能となります。

```dart
final file = File('/storage/emulated/0/Download/output/log.txt');
await file.writeAsString('ログの内容');
```

![](https://storage.googleapis.com/zenn-user-upload/8fd201bd3320-20250403.png =300x)

<br>

## iOS

Android と同様に、２つの関数の実行結果を例に見ていきます。

- getApplicationDocumentsDirectory() => `/var/mobile/Containers/Data/Application/<AppId>/Documents`
- getDownloadsDirectory() => `/var/mobile/Containers/Data/Application/<AppId>/Downloads`

Android 同様、これらはどちらも`アプリ専門領域`であり、ユーザーからは見えません（アクセスできません）。

上記の `アプリ専用領域` に関しては、シミュレーターであれば、`Finder` から確認することができます。

確認できるパスは以下です。
`/Users/<user>/Library/Developer/CoreSimulator/Devices/<DeviceID>/data/Containers/Data/Application/<AppID>/`

<br>

実機の場合は、path からは直接確認はできませんが、OS の権限を付与すれば、実機上で直接確認できる（ユーザーがアクセスできる）ようになります。

```info.plist: info.plist
<key>LSSupportsOpeningDocumentsInPlace</key>
<true/>
<key>UIFileSharingEnabled</key>
<true/>
```

<br>

iPhone で `ファイル` アプリから確認ができるようになります。

![](https://storage.googleapis.com/zenn-user-upload/33842c85dd3e-20250403.png =300x)

## 備考

- Android の `Files` アプリで表示される「内部ストレージ」画面のディレクトリ名は、`/sdcard/`です。（Android は"SD カード" もあるのでややこしいですが、、、）
  「仮想 SD カード ≒ 内部ストレージ ≒ /sdcard」
  ![](https://storage.googleapis.com/zenn-user-upload/f5e0bca5a299-20250403.png =300x)

- `/sdcard`と`/storage/emulated/0`は同じディレクトリを指します。
  `/storage/emulated/0`のシンボリックリンクが`/sdcard`みたいです。

<!-- ## 参照 -->

<!-- https://flutter.salon/plugin/pathprovider/ -->
