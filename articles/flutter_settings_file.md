---
title: "【Flutter】設定ファイルはどれをどこまでgit管理する？"
emoji: "⚙️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter,dart,git]
published: false
---
flutterやパッケージのバージョンアップ等で、設定ファイルが変更された際に、どれをgit管理したらいいかがいつも悩む。

以下、Flutter3.3系から、FLutter3.7系にバージョンアップした際の変更があった設定ファイルについて、git管理するかどうかを調べてみた。

# .lockファイル
.lockフィアルは、基本的にはgit管理する。
ファイルの整合性を保つために。

https://github.com/flutter/flutter/issues/9299
https://minpro.net/pubspec-lock
https://guides.cocoapods.org/using/using-cocoapods.html


# project.pbxproj
https://totutotu.hatenablog.com/entry/2015/07/31/project.pbxproj%E3%81%AEgit%E3%81%A7%E3%81%AE%E6%89%B1%E3%81%84%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6%E3%81%A8conflict%E3%81%AE%E8%A7%A3%E6%B6%88



# pubspec.lock の記載内容の変更 
以下のようにFlutter3.7系から記載が変更に。

```yaml: pubspec.lock
    description:
      name: hogehoge
+      sha256: "98d1d33ed129b372846e862de23a0fc365745f4d7b5e786ce667fcbbb7ac5c07"
+      url: "https://pub.dev"
-      url: "https://pub.dartlang.org"
```
https://zenn.dev/kingu/articles/7215bb92d8a897
https://github.com/PalisadoesFoundation/talawa/issues/1643
https://stackoverflow.com/questions/75751974/dependabot-version-fails-due-to-outdated-flutter-version


# 参考アプリ
[mono/github](https://github.com/mono0926/wdb106-flutter/tree/main/ios)