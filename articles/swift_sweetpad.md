---
title: "【Swift】VSCodeやCursorで快適なSwift開発ライフを送りたい"
emoji: "🐦️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [swift, vscode, cursor, sweetpad, xcode]
published: true
publication_name: ncdc
---

「新たに Swift 案件やることになったが Xcode は慣れず使いづらい」
「使い慣れた VSCode で Swift 開発したい」
「Xcode プロジェクトを VSCode で開いても大量のエラーが出て見るに耐えない」

<br>

そんな悩みの１つの解決策として、 **SweetPad** という VSCode 拡張機能があります。
**SweetPad を使うことで、VSCode や Cursor で基本的な Swift プロジェクト（Xcode プロジェクト）開発ができるようになります。**
https://sweetpad.hyzyla.dev/

※ 以降、`VSCode`と記載箇所は`（Cursor含む）`の前提で記載いたします。

<br>

## 検証時の動作環境

- Xcode 16.2
- SweetPad 0.1.66

<br>

## SweetPad とは

SweetPad とは、VSCode または Cursor を使用して、 Swift プロジェクトを開発できるようにする拡張機能です。
Xcode の代替手段を目指して開発されている拡張機能みたいです。

https://marketplace.visualstudio.com/items?itemName=sweetpad.sweetpad

<br>

SweetPad を用いることで実現可能となる機能は以下です。（引用）

- ✅ **Autocomplete** — `xcode-build-server` を使用してオートコンプリートを設定する
- 🛠️ **Build & Run** — `xcodebuild` を使用してアプリケーションをビルドして実行します
- 🪲 **Debug** — `CodeLLDB` を使用して iOS アプリケーションをデバッグする
- 💅🏼 **Format** — `swift-format` または他の任意のフォーマッタを使用してファイルをフォーマットします
- 📱 **Simulator** — iOS シミュレーターを管理する
- 📱 **Devices** — iPhone または iPad で iOS アプリケーションを実行する
- 🛠️ **Tools** — `Homebrew` を使用して重要な iOS 開発ツールを管理します
- ✅ **Tests** — シミュレータとデバイスでテストを実行する

どれも設定が簡単なのも嬉しいポイントです。
使用条件としては、「MacOS であること」と「Xcode がインストールされていること」が必要となります。

<br>

## 基本設定

基本的には、以下２つの拡張機能をインストールするだけで OK です。

- [SweetPad](https://marketplace.visualstudio.com/items?itemName=sweetpad.sweetpad)
- [Swift](https://marketplace.visualstudio.com/items?itemName=swiftlang.swift-vscode)

SweetPad のインストール後、🍬アイコンを選択すると、３つのメニューが表示されます。

- **BUILD**
  - ビルド対象一覧
- **DESTINATIONS**
  - 実行可能なデバイス一覧
- **TOOLS**
  - 機能実行に必要なツール設定状況
    - 全てを必ず有効にする必要はなく、使いたい機能次第で有効とするでも問題ありません。
    - ここからインストールも可能（右側のコンソールボタンを押下するとインストールコマンドを打ってくれます）

![](https://storage.googleapis.com/zenn-user-upload/abbf662d896a-20250612.png =350x)

以降、各機能について詳細に見ていきます。

## ✅ [AutoComplete](https://sweetpad.hyzyla.dev/docs/autocomplete)

VSCode でも Xcode のように補完や参照がされるようになります。
こちらを有効化することで、エラーだらけの真っ赤な VSCode 画面が解消されます。

1. TOOLS から`xcode-build-sever`をインストール
2. コマンド パレットからコマンド `SweetPad: Generate Build Server Config（buildServer.json）` を実行し、設定ファイルを作成。（生成された設定ファイルは特にいじらなくて OK）
3. 一度 build を実行すると、設定ファイルが反映されて補完機能等が有効に

![](https://storage.googleapis.com/zenn-user-upload/1a7a309f0386-20250612.png =500x)

**【備考】**

- **内部的な関係性**
  - SweetPad ↔️ xcode-build-server ↔️ SourceKit-LSP
- **LSP**
  LSP（Language Server Protocol）とは、「コード補完、定義移動、エラーチェック」などのサポート機能（言語サービス）を、**エディタや IDE に提供するためのプロトコル**。
  LSP は Protocol であるため、**LSP 自体に言語サービスが備わっているものではありません**。
- **SourceKit**
  Apple が提供する Swift 言語の解析エンジン。Swift の「コード補完、定義移動、エラーチェック」などの**サポート機能（言語サービス）を提供**。
  Xcode をエディタとして使用する場合は、デフォルトで Xcode に組み込まれています。
- **[SourceKit-LSP](https://github.com/swiftlang/sourcekit-lsp)**
  **SourceKit が持つサポート機能を**、共通の通信規格（LSP）で**他のエディタや IDE に提供**するためのもの。
  SourceKit-LSP を用いることで、Xcode 以外のエディタでも、**素の Swift コードに対して SourceKit が有効となります**。

  ただし、**「Xcode プロジェクト」に対しては、 SourceKit-LSP はサポートしていません。**
  ※「Xcode プロジェクト」 ≒ 「.xcodeproj や .xcworkspace などを持つ、Xcode 独自のビルドシステム、プロジェクト構造のこと」を、本記事では指します。

- **[xcode-build-server](https://github.com/SolaWing/xcode-build-server)**
  - SourceKit-LSP に対して、「Xcode プロジェクト」に対してもサポート機能（言語サービス）を機能させるようにするためのもの。

<br>

## 🛠️ [Build & Run](https://sweetpad.hyzyla.dev/docs/build)

`xcodebuild`を使用して、VSCode から直接ビルドして実行することができます。
また、TOOLS の`xcbeautify`を有効化して使用することがオススメされています。

- 🔽 アイコン：ビルド&実行
- 🔧 アイコン：ビルド
- 右クリック：その他タスク
  - テスト実行もここから実行可能

![](https://storage.googleapis.com/zenn-user-upload/9a99b5d07a7a-20250612.png =500x)

`settings.json` や `tasks.json` を設定することで、ビルドや実行の挙動をカスタマイズすることも可能です。詳細は[ドキュメント](https://sweetpad.hyzyla.dev/docs/build)を参照ください。

**【備考】**

- **xcodebuild**
  Xcode のビルドシステムをコマンドラインから操作するためのコマンドラインツール。
- **[xcbeautify](https://github.com/cpisciotta/xcbeautify)**
  `xcodebuild` の出力を見やすく整形するツール。[fastlane](https://docs.fastlane.tools/best-practices/xcodebuild-formatters/) でも推奨。

<br>

## 🪲 [Debug](https://sweetpad.hyzyla.dev/docs/debug)

VSCode のデバッグ機能を使用して、iOS アプリケーションのデバッグ実行が可能となります。
Flutter などではお馴染みの F5 キー 実行によるデバッグ機能が使えます。

![](https://storage.googleapis.com/zenn-user-upload/cf4e9d22785d-20250612.png)

ひとまずは、生成された`launch.json` の設定をそのまま使用すれば、デバッグ実行が可能です。
実行環境や実行モードを切り替えて実行したい場合などは、適宜[ドキュメント](https://sweetpad.hyzyla.dev/docs/debug)を参照に`launch.json`を編集してください。

**【備考】**

- **[LLDB](https://lldb.llvm.org/)**
  OSS の debugger であり、Xcode デフォルトの debugger。
- **[CodeLLDB](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb)**
  LLDB を搭載した VSCode 用の拡張機能。
  SweetPad では、内部的に CodeLLDB を使用している。

<br>

## 💅🏼 [Format](https://sweetpad.hyzyla.dev/docs/format)

`settings.json` で設定した format を使用して、Swift ファイルを自動フォーマットすることができます。

```json: settings.json
{
  "[swift]": {
    "editor.defaultFormatter": "sweetpad.sweetpad",
    "editor.formatOnSave": true
  }
}
```

SweetPad では Apple 純正の[swift-format](https://github.com/swiftlang/swift-format)がデフォルト使用されますが、他の format を指定して使用することも可能です。

<br>

## 📱 [Simulator & Devices（DESTINATIONS）](https://sweetpad.hyzyla.dev/docs/destinations)

SweetPad では、iOS シミュレータや実機デバイスを簡単に管理することができます。
SweetPad では、特定のシミュレータや接続されたデバイスなど、アプリを実行できるデバイスのことを「Destination」と呼ぶようです。

<br>

## おわり

SweetPad で快適な Swift 開発ライフを。

## 参考記事

:::details 【参考記事】

https://zenn.dev/mtshiba/books/language_server_protocol
https://qiita.com/ushisantoasobu/items/320db16691948a52b5e4
https://www.docswell.com/s/usami-k/5XEQY4-develop-ios-app-with-ai#p1
https://dimillian.medium.com/how-to-use-cursor-for-ios-development-54b912c23941
https://qiita.com/usamik26/items/50f22983fcac0bbd6e74
https://qiita.com/m1k50118/items/d421a569e7f53eb91763
https://zenn.dev/aromarious/articles/sweetpad-extension-introduction

:::
