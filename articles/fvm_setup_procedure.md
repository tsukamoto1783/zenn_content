---
title: "fvm導入"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, Dart, fvm]
published: false
---

# fvm導入
基本的には以下の記事参照
https://zenn.dev/altiveinc/articles/flutter-version-management#%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%94%E3%81%A8%E3%81%AB%E5%BF%85%E8%A6%81%E3%81%AA%E8%A8%AD%E5%AE%9A%2Bide%E3%81%AE%E8%A8%AD%E5%AE%9A

# プロジェクトで適応させる場合
## 補足が必要だった箇所
以下のsettings.jsonの記載について、よくわからなかった部分があったので少し補足。
[該当記事の箇所](https://zenn.dev/altiveinc/articles/flutter-version-management#vs-code)

<公式>
https://fvm.app/docs/getting_started/configuration#option-1---automatic-switching-recommended

公式には設定が推奨と書いているので、記載するものとする。

```yaml :.vscode/settings.json
{
  "dart.flutterSdkPath": ".fvm/flutter_sdk",
  // Remove .fvm files from search
  "search.exclude": {
    "**/.fvm": true
  },
  // Remove from file watching
  "files.watcherExclude": {
    "**/.fvm": true
  }
}
```

## この設定をするとどうなる？
"fvm"とprefixをつけなくても、自動的にfvmのバージョンが適用されるようになる。
具体的には、「flutter run」や、「f5ボタンでの debug run」が自動的に"fvm"適応されるようになる。

# 個々のlocal環境でのみfvmを有効にし、バージョン管理する場合
[該当記事の箇所](https://zenn.dev/altiveinc/articles/flutter-version-management#global-%E3%82%92%E4%BD%BF%E3%81%88%E3%81%B0%E3%81%A9%E3%81%93%E3%81%A7%E3%82%82-flutter-%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E3%81%8C%E4%BD%BF%E7%94%A8%E3%81%A7%E3%81%8D%E3%82%8B)

こちらのglobal設定をすると、local上での"flutter"コマンドは、全てfvmで設定したバージョンが適応される。
→F5ボタンでも適用されるはず？未検証。