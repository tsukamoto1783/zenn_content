---
title: "【Flutter】flutter build ipa でipaファイルが生成されない"
emoji: "⬆️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter,dart,ios,build,ipa]
published: true
---

## エラー内容
`flutter build ipa`を実行すると、buildは成功するが、ipaファイルの作成に失敗する。
```
Running Xcode build...
Xcode archive done.

Building App Store IPA...
Encountered error while creating the IPA:
error: exportArchive: "Runner.app" requires a provisioning profile.
```

error内容より、provisioning profileが必要だとかごにょごにょ言われている。

Xcodeを確認すると、provisioning profileは設定されている。
build自体は成功しているので、Xcodeから手動でArchive実行すると、成功する。

## エラー原因
よく調べると、ipaファイルをコマンドで生成するには、`--export-options-plist`オプションが必要らしい。
このオプションを指定して`flutter build ipa --export-options-plist=ExportOptions.plist`を実行すると、コマンド一つでipaファイル生成まで実行できるとのこと。

https://docs.flutter.dev/deployment/ios#upload-the-app-bundle-to-app-store-connect


## 手順
まずは、`flutter build ipa export-options-plist=ExportOptions.plist`コマンドで急に出てきた`ExportOptions.plist`を準備する。

`ExportOptions.plist`は、XcodeでArchive実行してエクスポートしたフォルダに同封されている。

![](https://storage.googleapis.com/zenn-user-upload/6b56ef4caba2-20230930.png)

|                                     xcode                                      |                                     xcode                                      |
| :----------------------------------------------------------------------------: | :----------------------------------------------------------------------------: |
| ![](https://storage.googleapis.com/zenn-user-upload/51872ff7623d-20240122.png) | ![](https://storage.googleapis.com/zenn-user-upload/bf528df07b64-20240122.png) |


```yaml
export_folder
 ├─ DistributionSummary.plist
 ├─ ExportOptions.plist ←これ
 ├─ Packaging.log
 └─ ×××.ipa
```

`ExportOptions.plist`を取得したら、プロジェクトの任意の場所に格納する。

そして、格納先のパスをコマンドに指定してあげて実行すれば、コマンドでipaファイルの生成まで実行ができるようになる。
`flutter build ipa export-options-plist=ExportOptions.plist`
