---
title: "SDKMANでプロジェクト毎にJavaバージョンを切り替える"
emoji: "🏀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [sdkman, java, jdk, claudecode]
published: true
publication_name: ncdc
---

kotlin プロジェクトなどの場合、プロジェクト毎に Java のバージョンを切り替えたいことがあるかと思います。

そんな時にサクッと Java のバージョンを切り替えられる[SDKMAN](https://sdkman.io/)というツールがあります。

<br>

## SDKMAN とは

[SDKMAN](https://sdkman.io/)は `Software Development Kit Manager`であり、macOS などの Unix 系システム上で、**SDK （フレームワークやビルドツールなど含む）を管理するためのツールです。**

今回は Java のみを対象に説明しますが、Java 以外にも有名どこで言うと Gradle や Spring Boot なども管理できるみたいです。

煩雑なコマンドや設定が不要で、比較的簡単にバージョン管理ができるのが特徴です。

<br>

## SDKMAN の導入

[ドキュメント](https://sdkman.io/install/)にある通り、基本的な導入はこれだけです。

![](https://storage.googleapis.com/zenn-user-upload/629c23cf77f8-20251113.png)

<br>

SDKMAN のインストールが成功したら、使用したいツールのバージョンを確認し、install を実行してください。

基本のコマンドを記載します。その他のコマンドについてはドキュメントを参照してください。

- `sdk list`: 使用可能ツール一覧
- `sdk list <tool-name>`: ツールの使用可能バージョン一覧
- `sdk install <tool-name> <version>`: 指定バージョンの install
- `sdk use <tool-name> <version>`: install 済みバージョンの適用
- `sdk current <tool-name>`: 現在使用中のバージョン確認

※ 今回で言うと`<tool-name>`部分が`java`に置き換わります。

https://sdkman.io/usage/

<br>

## プロジェクト毎の SDKMAN 設定

ここから本題です。
**SDKMAN でプロジェクト毎に Java のバージョンを切り替えたい場合、
プロジェクトの実行ディレクトリ配下にて、`sdk env init`を実行します。**

すると、java バージョンが指定された`.sdkmanrc`という設定ファイルが生成されます。

```txt
# Enable auto-env through the sdkman_auto_env config
# Add key=value pairs of SDKs to use below
java=21.0.8-oracle # 適宜バージョンを指定
```

<br>

こちらのファイルをコミットしてプロジェクトで共有することで、**プロジェクトメンバーは`sdk env`コマンドを実行するだけで、プロジェクト毎に Java バージョンを切り替えることができます。**

この状態で`./gradlew build`などを実行すると、指定された Java バージョンでビルドが実行されます。
戻したい場合は、`sdk env clear`で default 設定に戻せます。

![](https://storage.googleapis.com/zenn-user-upload/744a78a2468b-20251122.png)
_一連のコマンド実行確認例_

<br>

### 注意点

デフォルトでは`sdk env`コマンドは、実行したシェルセッション内でのみ有効です。
**`sdk env`を実行した後に、ターミナルを閉じて新しいターミナルを開いた場合などは、再度`sdk env`を実行する必要があることには注意です。**

毎回`sdk env`を実行するのが面倒な場合は、自動的に設定したいバージョンを読み込む`sdkman_auto_env`設定を有効にする方法があります。

`~/.sdkman/etc/config`ファイル内の`sdkman_auto_env`設定を`true`にすると、自動的に`.sdkmanrc`ファイルを検出して設定を読み込むようになります。
ユーザー毎のグローバルな設定にはなるので、参画時のルールや仕組みは追加する必要がありますが、頻繁に使う人は良さそうです。
[詳細は Doc 参照](https://sdkman.io/usage/#configuration)。

<br>

## AI コーディングツール使用時の注意点

AI コーディングツールを使用する際には、ツールによってはバージョンが反映されない場合があります。

私の場合は Claude Code でしか確認できていませんが、
ローカル環境で`sdk current java`で使用したいバージョンが適用されていることを確認後、Claude Code で単純に`./gradlew build`を実行したところ、デフォルトの Java バージョンでビルドが実行されてしまいました。

**解決策としては、Claude Code に実行してもらう際は、`Bash(source ~/.sdkman/bin/sdkman-init.sh && sdk env && ./gradlew build)`のように`sdkman-init.sh && sdk env`と同じプロセス内で実行してもらうように指示してあげましょう。**

※（`sdkman_auto_env`が有効になっている場合は、`Bash(source ~/.sdkman/bin/sdkman-init.sh && ./gradlew build)`で OK です。）

````diff: CLAUDE.md
# 例

## Build Commands

```bash
# ビルド
- ./gradlew [コマンド]
+ source ~/.sdkman/bin/sdkman-init.sh && sdk env && ./gradlew [コマンド]
```

````

※ 注意：
以下のように Claude Code 上で連続実行だと、環境変数が引き継がれない（反映されない）ため、`&&`で同一プロセス内で実行する必要があります。

```
1. Bash(source ~/.sdkman/bin/sdkman-init.sh && sdk env)
2. Bash(./gradlew build)
→ env のバージョンではなく default のバージョンが反映される
```

<br>

### 備考：

動作検証時：`Claude Code: v2.0.30`

Claude Code のドキュメント見る限りでは、同一のシェルプロセス内では、環境変数などは引き継がれるようです。（永続的なセッション維持）

https://platform.claude.com/docs/en/agents-and-tools/tool-use/bash-tool#example-usage

![](https://storage.googleapis.com/zenn-user-upload/d5789f377b40-20251123.png)

しかし、現状ではそうはならないケースがあると、Issue が上がってるみたいです。
（いけるはずなのになんでなんだろう？と詰まりポイントでした。。）

バージョンによっては改善されていたり、アプデで再発したり、などが発生しているみたいです。

https://github.com/anthropics/claude-code/issues/2508
https://github.com/anthropics/claude-code/issues/334

どこかのバージョンアップで改善されることを期待しています 🙏
