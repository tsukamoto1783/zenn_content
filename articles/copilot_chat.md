---
title: "GitHub Copilot Chatの基本的な使い方"
emoji: "✈️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [copilot,vscode,github,copilotChat]
published: true
publication_name: ncdc
---

基本的な使い方だけ抑えて便利なCopilot Chatを使ってみよう！

<br>

::: message
主な使い方は２つ。
- 拡張機能ウインドウで使用。
  - コードの回答はチャット欄に回答される。
- コード上で直接使用。
  - コードの回答は直接コード上に表示される。
:::

### 前提条件
- GitHub Copilotが導入済みであること。
- エディタはVSCodeを使用していること。

### 使用準備
1. GitHub Copilot Chatを有効化する
    GitHubの`settings/copilot/policies` から GitHub Copilot Chatを有効化する。
    ![](https://storage.googleapis.com/zenn-user-upload/6f94852f4ba0-20230822.png)
2. 拡張機能の「GitHub Copilot Chat」をインストールする
    ![](https://storage.googleapis.com/zenn-user-upload/18649cc04f18-20230822.png)

<br>
<br>


# 拡張機能ウインドウで使用
チャット形式なので、基本的には質問したい内容を質問すればOK。
**コードの回答はチャット欄に回答される。**

## 使用方法
拡張機能をインストールした後、表示されるアイコンを押下するとチャット画面が開く。
![](https://storage.googleapis.com/zenn-user-upload/b854a12a0a8c-20230822.png)

表示中のコードを自動的に読み込んだ上で、質問に回答してくれる。
![](https://storage.googleapis.com/zenn-user-upload/6a4fb739f492-20230822.png)

また、コードを選択した状態で質問すると、その箇所が解答の対象となる。
![](https://storage.googleapis.com/zenn-user-upload/c0a2ba2cd9da-20230822.png)

<br>

## コマンド
Copilot Chatにはデフォルトでいくつかのコマンドが用意されている。
よくあるチャット内容をショートカットで質問できるイメージ。

:::: details コマンド一覧
- /createNotebook: 新しいJupyterノートブックを作成する。
- /createWorkspace: 新しいワークスペースのスケルトンコードを生成する。
- /explain: 選択したコードがどのように機能するかを説明する。
- /extApi: VS Codeの拡張開発について質問する。
- /fix: 選択されたコードの問題の修正を提案する。
- /help: Copilot Chat の使い方を学ぶ。
- /search: ワークスペース検索のクエリパラメータを生成する。
- /simplify: 選択したコードを簡素化する。
- /tests: 選択したコードの単体テストを生成する。
- /vscode: VS Codeに関する質問をする。
- /clear: セッションをクリアする。
::::
![](https://storage.googleapis.com/zenn-user-upload/f79ab9924eca-20230822.png)

<br>

いくつか抜粋して紹介。
### /explain
選択したコードがどのように機能するかを説明する。
![](https://storage.googleapis.com/zenn-user-upload/f19cd10f42f7-20230822.png)

### /tests
選択したコードの単体テストを生成する。
![](https://storage.googleapis.com/zenn-user-upload/1285263922cb-20230822.png)


<br>
<br>


# コード上で直接使用
**コードの回答が直接コード上に表示される。**

## 使用方法
コードを選択して、右クリックで「Copilot」を選択。以下のメニューが表示される。
![](https://storage.googleapis.com/zenn-user-upload/c3d50c34d665-20230822.png)

<br>

### Start Code Chat
コード上で最小のチャット画面が表示され、質問等ができる。
![](https://storage.googleapis.com/zenn-user-upload/ea9e8b19b483-20230822.png)
以下のcommandも使用可能。
- /docs
- /explain
- /fix
- /tests

<br>

### Explain This （/explain）
選択したコードがどのように機能するかを説明する。

<br>

### Fix This （/fix）
選択されたコードの問題を修正する。
![](https://storage.googleapis.com/zenn-user-upload/be2f0b55cdd5-20230822.png)

<br>

### Generate Docs （/docs）
選択されたコードのdocコメントを追加する。
![](https://storage.googleapis.com/zenn-user-upload/fbffe71b01da-20230822.png)

<br>

### Generate Tests （/tests）
選択したコードの単体テストを生成する。
![](https://storage.googleapis.com/zenn-user-upload/d0fc34a49404-20230822.png)
