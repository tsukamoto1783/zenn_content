---
title: "【Swift】Xcode で設定値の変更保存が一切出来なくなった時の対応方法"
emoji: "🔧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Swift, Xcode, chmod]
published: true
publication_name: ncdc
---

## 【事象】

Xcode を操作していると、ふとした時に設定値の変更保存が一切できなくなる現象が発生しました。

なにやら「プロジェクトがロックされているので、ロック解除しますか？」とのこと。
もちろん保存したいので "UnLock" を選択するも、直後に「権限が無いからロック解除できませんでした」とのエラーに。

この状態では設定値の変更の保存はもとより、 Xcode を終了することもできずに強制終了するしか手立てがない状態となってしまいます。

![](https://storage.googleapis.com/zenn-user-upload/4ad8f5491746-20250702.png)
![](https://storage.googleapis.com/zenn-user-upload/1df6d137e67d-20250702.png)

なぜ発生したかの原因は不明ですが、エラー文にあるように、"project.pbxproj" の書込み権限を追加してみましょう。

<br>

## 【対応】

まず、そもそも `project.pbxproj` が Xcode プロジェクトのどこの階層に存在するかを確認します。
VSCode でフォルダを確認すると、`<PJ Name>.xcodeproj`配下に`project.pbxproj`が存在することが確認できました。

![](https://storage.googleapis.com/zenn-user-upload/f18b66ae3fa0-20250702.png)

ただ、上記を Finder から確認しようとすると、`<PJ Name>.xcodeproj`配下のフォルダが一切見れません。
`.xcodeproj`は、フォルダではなく ファイル扱いとなります。

![](https://storage.googleapis.com/zenn-user-upload/a9758f6297c5-20250702.png)

Finder 上で確認できるファイルであれば、"右クリック → 情報を見る" より、以下のような GUI 操作で権限の設定も可能なのですが、`project.pbxproj`は GUI 操作では権限の設定等ができません。

![](https://storage.googleapis.com/zenn-user-upload/b0b3e708447b-20250702.png)

そうなると、コマンド操作での権限操作しか無いので、`<PJ Name>.xcodeproj`ディレクトリに移動し、`sudo chmod 777 project.pbxproj` のような権限付与のコマンドを実行すると、無事に Xcode 上の変更を保存できるようになりました。

※ 取り急ぎ `777` の権限としましたが、Xcode 上の保存が済んだら適宜適切な権限設定をお願いします。

<br>

**【備考・補足】**

- chmod とは、UNIX 系 OS で使えるファイルのアクセス権限を変更するコマンドです。
  - chmod = "change mode"
  - ex. "drwxr-xr-x"
    - 1 桁：d：ファイル種別（ディレクトリ）
    - 3 桁：rwx：所有者の権限（読み取り、書き込み、実行）
    - 3 桁：r-x：グループの権限（読み取り、実行）
    - 3 桁：r-x：その他の権限（読み取り、実行
  - コマンドを全部覚えるのは大変なので、必要に応じて適宜調べてください。（そこまで頻繁に権限をコマンドでいじることも無い場合は特に。）
