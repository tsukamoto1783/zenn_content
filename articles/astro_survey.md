---
title: "ウェブフレームワークのAstroを触ってみた"
emoji: "🧑‍💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [react, astro, nextjs, frontend]
published: true
publication_name: ncdc
---

以前から気になっていたウェブフレームワークの`Astro` について、社内フロントチームでざっくり学び、チュートリアル程度の動作確認をしてみた感想をまとめてみました。

<br>

# Astro とは

静的コンテンツ中心のウェブサイトに特化した ウェブフレームワークです。
2022 年 8 月に v1.0 がリリースされ、2024 年 2 月時点では v4.3.6 がリリースされています。
https://Astro.build/

<br>

# 特徴

https://Astro.build/integrations/

> **Astro はオールインワンのウェブフレームワークです。** Astro には、ウェブサイトを作成するために必要なすべてが組み込まれています。

とのことです。

特筆すべき点は以下。

- **アイランド（フロントエンドアーキテクチャ）**
- **自由な UI**
- **サーバーファースト**
- **デフォルトで JS 無し**
- **コンテンツコレクション（表示コンテンツ管理機能）**
- **カスタマイズ可能**

<br>

### アイランド（フロントエンドアーキテクチャ）

- Astro ではインタラクティブな UI コンポーネントを`Islands（アイランド）`と呼びます。
- インタラクティブな UI 部分だけに javascript を適用し、他の静的な HTML 部分はサーバー側でレンダリングすることで、**従来の SPA に比べて高速なページ読み込みを実現します。このパフォーマンスの高さがアイランドの最大の利点**みたいです。
  - （静的な HTML 部分が`海`で、その海に独立して浮かぶ島が`アイランド（インタラクティブなUIコンポーネント）`といったイメージ。）

<br>

### 自由な UI

- React、Preact、Svelte、Vue、SolidJS、AlpineJS、Lit のようなさまざまな人気のフレームワークがサポートされてます。
- 各アイランド（インタラクティブな UI コンポーネント）が独立していることにより、**複数のフレームワークを混在させることも可能です。**
  この機能によって、プロジェクト内で以下のようなことも可能に。
  - コンポーネントごとに最適なフレームワークを選択する。
  - 新しくプロジェクトを作成せずに Astro プロジェクトひとつで色んなフレームワークを学べる。
  - メンバーそれぞれで異なるフレームワークを使いながら、共同開発できる。
  - 既存のサイトを別のフレームワークへと、ダウンタイム無しで段階的に移行できる。

<br>

### サーバーファースト

- Astro では、処理の多くがブラウザではなくサーバーでおこなわれます。
  そのため、スペックの低いデバイスや、通信環境の悪い状態でサイトやアプリを表示したときに、**クライアントサイドレンダリングよりも高速になります。**
  サーバーでレンダリングされた HTML は、高速で SEO にも優れており、デフォルトでアクセシブルです。
- Astro の**デフォルト設定は事前レンダリング（SSG）**。
  - ex.) 更新頻度が低いサイト、全てのユーザーに共通のコンテンツを表示するようなコンテンツ重視のサイト、etc.
- ルートがリクエストされたときにサーバーで**オンデマンドレンダリング（サーバーサイドレンダリング(SSR)）するように設定することも可能。**
  - ex.) ログインしたユーザーにアカウント情報を表示したり、サイト全体を再ビルドすることなく最新のデータを表示したり、etc.

<br>

### デフォルトで JS 無し

- サイトの読み込みを遅くするクライアントサイドの JavaScript をデフォルトで減らしてくれます。
  - **インタラクティブな UI コンポーネントにだけ明示的に JavaScript を適用**させることで、
    高速なページ読み込み、パフォーマンスの最適化、SEO とアクセシビリティの向上などを実現してくれます。

<br>

### コンテンツコレクション（表示コンテンツ管理機能）

- Markdown コンテンツの効率的な管理、front matter（フロントマター）[^1]の検証による品質保証、TypeScript による型安全を提供します。
- `src/content`ディレクトリ内で適用され、このディレクトリ内にはコンテンツコレクションのみが許可され、他の用途で使用することができない専用のディレクトリとなります。

<br>

### カスタマイズ可能

- React、Vue、Svelte、Solid などの一般的な UI フレームワークを利用でき、Tailwind や Partytown などのツールも数行のコードで簡単に統合可能。
- 様々なカテゴリーの 100 種類以上のインテグレーションのセットも[コミュニティ](https://Astro.build/integrations/)によって公開されています。

<br>

# 設計原則

ドキュメントには Astro の特徴とは別に、[Astro の設計原則](https://Astro.build/integrations/)も記載されています。
上記の特徴と重複する内容も多いが、以下記載しておきます。

- **コンテンツ駆動**
  - Astro は、コンテンツを上手く表示するために設計されています。
  - ブログや LP,EC サイトなど、コンテンツ重視のウェブサイトを作成するために特化しており、高パフォーマンスで構築が可能に。
- **サーバーファースト**
  - HTML をサーバーでレンダリングすることで、ウェブサイトの動作が速くなります。
  - 最初のロードパフォーマンスが重要なコンテンツ重視のウェブサイトに適しています。
- **デフォルトで高速**
  - `Astro で遅いウェブサイトを作成することは不可能であるべき。` が目標。
  - 上記二つの原則（コンテンツ駆動、サーバーファースト）によって、デフォルトで高速なウェブサイトを構築できる。
- **簡単に使える**
  - スキルや経験に関係なく、「誰でも親しみやすく、誰でも何かを作成できるように」と設計されています。
  - ブラウザではなくサーバー上でレンダリングされるため、React で言うフックやクロージャなどの複雑な概念などを心配する必要がなく利用できます。
- **開発者を重視**
  - Astro を使って構築する際に必要なサポート全てを提供します。
  - CLI 体験、VSCode 拡張機能、多言語ドキュメント、Discord コミュニティ、など。

<br>

# 触ってみた感想

実際に軽くさわり、感じた開発体験を以下にざっくりと記載します。

- 動作やフォルダ構成など理解しやすかった。
  - API を呼び出したり、凝ったことをやろうとするとどれくらい大変そうなのかは未検証。
- ページを追加するのは、`pages` 以下にファイルを作って、画面遷移のためのリンクを貼るだけで楽。
  - ルーティングは、<a> タグでできる。
- hooks のような複雑な概念が少ないので、開発がシンプル。
- SSG に特化していて、Next.js のように多機能過ぎずに理解しやすい。
- React などと比べると、記法などはシンプルで分かりやすい気がする。
- SSR 設定も可能だが、動的ページが多い場合は遅くなるので、Next.js とかの方がよさそう。
- 独自の拡張子が見慣れないが、中身は react などとほぼ同じなので、慣れれば問題なさそう。
- 静的コンテンツ中心のサイトだと、Next.js よりパフォーマンスも開発体験も良さそう。
- etc.
  ![](https://storage.googleapis.com/zenn-user-upload/e528c7800a80-20240214.png)
  _ブログチュートリアルのフォルダ構成_

<br>

※ ウェブフレームワークの第一候補である Next.js と比較してみた系の記事は、検索すると沢山見つかるので、詳細が気になる方は下記[「参考」](#参考)をご参照ください。

<br>

# まとめ

[特徴](#特徴)や[設計原則](#設計原則)にも書いているように、以下の特徴のプロジェクトならば、Astro を選択肢として考えても良いかもしれません。

- ブログや LP、EC サイトなどの静的コンテンツ中心のウェブサイトを作成する場合。
  - ユーザー主体の操作ではなく、操作動線がある程度決められているウェブサイトを作成する場合。
  - 逆に DB とのやりとりが多い業務系のアプリケーションなどでは不向き。動的な部分がどれぐらい実装必要かが選定ポイント。
- ブログやランディングページなどを少しリッチに作りたいが、React や Next.js 触れる人がいない場合。

<br>

今後、条件にマッチする小さめのプロジェクトがあれば、本格的に試しても良さそうだなと思う `Astro` でした。

<br>

## 参考

https://zenn.dev/shuufei/articles/6e115fe4c8f74e

https://zenn.dev/jimbo/articles/f7f8ea2aafc177

https://commte.net/astro-introduction

https://kakaku-techblog.com/entry/create-website-with-astro

[^1]: front matter: Markdown コンテンツファイルの先頭に書かれるメタデータのブロックのこと。タイトルや日付、タグなどが記載される。主に Markdown や HTML で使用され、整理や管理が容易になる。
