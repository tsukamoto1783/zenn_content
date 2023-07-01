---
title: "GitHub管理のドキュメントをPDF化する"
emoji: "🧾"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [GitHub, vscode, pdf, markdown, MarkdownPdf]
published: true
---
GitHubで管理しているドキュメント類をPDF化するために、vscode拡張の「[Markdown PDF](https://marketplace.visualstudio.com/items?itemName=yzane.markdown-pdf)」を使用した際のメモ。

::: message
【注意事項】
- エディタはvscodeを使用する前提での記事となります。
- 「Markdown PDF」について、何年も更新はされてないので、環境によっては正常に動作しないかもしれません。
ただ、基本的な機能については十分使用できる拡張機能なので使い勝手は良いです。
もし、使っていく中で不具合等が発生した場合は、[「md-to-pdf」](https://github.com/simonhaenisch/md-to-pdf)など、他のツールを検討してみてください。
:::

# Markdown PDFの特徴
- vscodeの拡張機能なので、vscodeのエディタ上だけでPDF化が完結。
- コマンド一つで生成されるので楽。
- PDF化に関する設定を、`.vscode/settings.json`で管理すればチームで共有しやすい。
- etc.

# 基本的な使い方
基本的な使い方は、vscodeの拡張機能のインストールして、.mdファイルに対して実行するだけ。
1. vscodeの拡張機能から「[Markdown PDF](https://marketplace.visualstudio.com/items?itemName=yzane.markdown-pdf)」をインストール。
2. .mdファイル開いて、「Shift + Cmd + P」でコマンドパレットを開く。
3. 「Markdown PDF: Export (pdf)」を選択すると、PDFが出力される。


【デフォルト設定での出力結果】
![](https://storage.googleapis.com/zenn-user-upload/8e7a5c373373-20230701.png =300x)

# 各種オプション設定
「拡張機能の設定」から、pdf出力の設定を変更することも可能。
設定がデフォルト値から変更されたものは、`.vscode/settings.json`に記載される。

【headerを無効+footer部分の設定修正での出力結果】
![](https://storage.googleapis.com/zenn-user-upload/e33fb3e3d3a4-20230701.png =300x)

設定できるプロパティは沢山あるので以下に一覧を記載。
各種プロパティの詳細は[GitHubのドキュメント](https://github.com/yzane/vscode-markdown-pdf/blob/master/README.ja.md#list)参照。

【プロパティ一覧】
| Option | Description |
|---|---|
| markdown-pdf.type | 出力フォーマットを指定。pdf, html, png, jpegが選択可能。デフォルトはpdf。 |
| markdown-pdf.convertOnSave | 保存時に自動変換を有効にするかどうかを指定。デフォルトはfalse。 |
| markdown-pdf.convertOnSaveExclude | convertOnSaveオプションの除外ファイル名を指定。 |
| markdown-pdf.outputDirectory | 出力ディレクトリを指定。 |
| markdown-pdf.outputDirectoryRelativePathFile | markdown-pdf.outputDirectoryで設定した相対パスがファイルからの相対パスとして解釈されるかどうかを指定。デフォルトはfalse。 |
| markdown-pdf.styles | markdown-pdfで使用するスタイルシートのパスを指定。 |
| markdown-pdf.stylesRelativePathFile | markdown-pdf.stylesで設定した相対パスがファイルからの相対パスとして解釈されるかどうかを指定。デフォルトはfalse。 |
| markdown-pdf.includeDefaultStyles | デフォルトのスタイルシートを有効にするかどうかを指定。デフォルトはtrue。 |
| markdown-pdf.highlight | Syntax highlightingを有効にするかどうかを指定。デフォルトはtrue。 |
| markdown-pdf.highlightStyle | スタイルシートのファイル名を指定。 |
| markdown-pdf.breaks | 改行を有効にするかどうかを指定。デフォルトはfalse。 |
| markdown-pdf.emoji | 絵文字を有効にするかどうかを指定。デフォルトはtrue。 |
| markdown-pdf.executablePath | バンドルされたChromiumの代わりに実行するChromiumまたはChromeのパスを指定。 |
| markdown-pdf.scale | ページレンダリングのスケールを指定。デフォルトは1。 |
| markdown-pdf.displayHeaderFooter | ヘッダーとフッター表示を有効にするかどうかを指定。デフォルトはtrue。 |
| markdown-pdf.headerTemplate, markdown-pdf.footerTemplate | ヘッダーとフッターを出力するためのHTMLテンプレートを指定。 |
| markdown-pdf.printBackground | 背景のグラフィックを出力するかどうかを指定。デフォルトはtrue。 |
| markdown-pdf.orientation | ページの向きを指定。portrait(縦向き)またはlandscape(横向き)が選択可能。デフォルトはportrait。 |
| markdown-pdf.pageRanges | 出力するページ範囲を指定。デフォルトは全ページ。 |
| markdown-pdf.format |用紙のフォーマットを指定。Letter, Legal, Tabloid, Ledger, A0, A1, A2, A3, A4, A5, A6が選択可能。デフォルトはA4。 |
| markdown-pdf.width, markdown-pdf.height | 用紙の幅/高さを指定。単位はmm, cm, in, px。 |
| markdown-pdf.margin.top, markdown-pdf.margin.bottom, markdown-pdf.margin.right, markdown-pdf.margin.left | 用紙の余白を指定。単位はmm, cm, in, px。 |
| markdown-pdf.quality | jpegのみ。イメージの品質を0-100の範囲で指定。 |
| markdown-pdf.clip.x, markdown-pdf.clip.y, markdown-pdf.clip.width, markdown-pdf.clip.height | ページの切り抜き領域を指定。 |
| markdown-pdf.omitBackground | デフォルトの白い背景ではなく、透過によるスクリーンショットのキャプチャーを有効に。デフォルトはfalse。 |
| markdown-pdf.plantumlOpenMarker, markdown-pdf.plantumlCloseMarker | plantumlパーサーの開始/終了区切り文字を指定。デフォルトは@startumlと@enduml。 |
| markdown-pdf.plantumlServer | Plantuml serverを指定。デフォルトはhttp://www.plantuml.com/plantuml。 |
| markdown-pdf.markdown-it-include.enable | markdown-it-includeを有効に。デフォルトはtrue。 |
| markdown-pdf.mermaidServer | mermaid serverを指定。デフォルトはhttps://unpkg.com/mermaid/dist/mermaid.min.js。 |


# PDFのstyle設定
以下の記事のように、cssファイルを読み込んで、全体的なstyleを変えることも可能。
参考：[VScodeのMarkdown PDFでGithub StyleのCSSを適用する](https://www.jackjasonb.com/2021/03/22/markdown-pdf-css/)
【css適用後の出力結果】
![](https://storage.googleapis.com/zenn-user-upload/4a7ae86118c8-20230701.png =300x)

# プレビュー設定
.mdファイルを作成中は、「Shift + Cmd + P」→「Markdown: Open Preview to the Side」でのプレビュー表示がマスト。
プレビューに関する拡張機能を以下に記載。

- [Markdown Preview Mermaid Support](https://marketplace.visualstudio.com/items?itemName=bierner.markdown-mermaid)
    [Mermaid](https://mermaid.js.org/)を使用している場合は、これを入れないとプレビューに図が反映されないので注意。

- [Markdown Preview Github Styling](https://marketplace.visualstudio.com/items?itemName=bierner.markdown-preview-github-styles)
    プレビュー画面が、Githubのスタイルで表示される。
    【デフォルトのプレビュー画面】
    ![](https://storage.googleapis.com/zenn-user-upload/17c951456da7-20230701.png =300x)
    【拡張機能適用後のプレビュー画面】
    ![](https://storage.googleapis.com/zenn-user-upload/b0ebe276fc50-20230701.png =300x)


# Markdown設定
「PDF化とは少し異なる」＆「これだけで1記事になる」の内容のため、別途記事参照。
[作成中...]()



### 参考記事
:::: details 参考記事
https://mseeeen.msen.jp/vscode-markdown-pdf-v1-header-footer-settings/

https://www.jackjasonb.com/2021/03/22/markdown-pdf-css/

::::

