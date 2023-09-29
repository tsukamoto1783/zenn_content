---
title: "【codecov】codecov.ymlの設定についてのあれこれ"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [codecov, GitHubActions, GitHub,CI,CD]
published: true
publication_name: ncdc
---

※この記事では、[codecovの公式ドキュメント](https://docs.codecov.io/docs/codecov-yaml)を参考に、codocov.ymlの設定に関わる部分をかいつまんで記載しております。
より詳細に知りたい場合は、要所要所に対応する公式ドキュメントのリンクを入れているので、そちらを参照ください。
<!-- 記載内容全てを実際に動かして確認してみたわけではないので、詳細は公式Documentを参照ください。（記載に間違いがあった場合はそっとコメントでお知らせ頂けたら幸いです。） -->



# codecov.yml
codecov.ymlとは、codecovの各種設定ファイル。
必須ではなく、存在しなくてもデフォルトの設定でcodecov自体は動作するが、codecov.ymlを設定することでより様々なことができる。

主な設定項目は以下5つ。[「About Code Coverage」](https://docs.codecov.com/docs/about-code-coverage#top-5-codecov-features)
- **PRコメント**
- **ステータスチェック（PRブロック）**
- **レポートのマージ**
- **フラグ**
- **パスの操作**


## codecov.ymlの種類
[「About the Codecov YAML」](https://docs.codecov.com/docs/codecov-yaml)
codecov.ymlにはGlobal YAMLとRepository YAMLの2種類ある。
- **Global YAML** : 設定が組織全体で適用される。Codecovの設定ページから設定可能。
- **Repository YAML** : 各リポジトリ固有で設定を行う。両方が設定されている場合、Repository YAMLが優先される。

<br>

## 使用できるトップレベルのセクション（キー）
[「codecov.yml Reference / Top-level sections used in codecov.yml」](https://docs.codecov.com/docs/codecovyml-reference#top-level-sections-used-in-codecovyml)

```yaml: codecov.yml
codecov :               # トップレベルのセクション。
component_management :  # コンポーネントの管理に関する設定。
coverage:               # カバレッジに関する設定。
parsers:                # さまざまな言語やツールのカバレッジレポートを解析する方法に関する設定。
ignore:                 # 特定のパスやファイルをカバレッジレポートから除外する設定。
fixes:                  # パスの修正に関する設定。
flags:                  # フラグに関する設定。
comment:                # プルリクエストのコメントに関する設定。
github_checks:          # GitHubユーザー専用の設定。
# 他省略
```
::: message
この記事では、以下のセクションについて少し深掘って見ていく。
- **codecov**
- **coverage**
- **comment**
- **flags**
:::

<!-- - Codecov Cloud Report Archiving: Codecovクラウドレポートのアーカイブに関する設定が含まれています。 -->
<!-- - annotations: GitHub Checksの注釈や通常のステータスを使用するかどうかを指定する設定が含まれています。 -->


### 注意事項
- この記事ではGitHub Actionsを使用してcodecovを利用することを前提として記載する。
actionsで設定できるパラメータについては[Codecov's Github Action v2/README](https://github.com/marketplace/actions/codecov#arguments)参照。
他の方法でcodecovを利用する場合は、[「Codecov Uploader」](https://docs.codecov.com/docs/codecov-uploader)などを参照。
- codocovはテスト自体は実行しない。
  actions内でテストを実行させて、その結果をcodocovに渡し、そのテスト結果に基づきcodecovは静的解析のためにカバレッジレポートやその他の重要なデータを収集する。
- 設定ファイルは`.github/codecov.yml`に配置する必要あり。
`.github/workflow/codecov.yml`でないことを注意。[「Frequently asked questions」](https://docs.codecov.com/docs/codecov-yaml#can-i-name-the-file-codecovyml)
- codecov.ymlを変更した際は、都度検証すること。[「Common Configurations」](https://docs.codecov.com/docs/common-recipe-list)
  `curl -X POST --data-binary @codecov.yml https://codecov.io/validate`

<br>
<br>

# [codecov:](https://docs.codecov.com/docs/codecovyml-reference#codecov)
codecovセクションでは、codecov全体の設定ができる。
```yaml
codecov:
  token: "<some token here>"        # リポジトリアップロードトークン
  bot: "codecov-io"                 # Codecovの操作に使用するユーザー名
  ci:
      - "travis.org"                # Codecovに認識させたいCIプロバイダーのURLを追加。
  strict_yaml_branch: "yaml-config" # Codecovに常にYAMLだけを読み込ませたいブランチを指定。
  max_report_age: 24                # カバレッジレポートの有効期限を設定。デフォルトは12h。
  disable_default_path_fixes: no    # Codecovのデフォルトのパス修正の有無。
  require_ci_to_pass: yes           # ステータスを送信する前に、他のすべてのステータスが通過するのを待つか。
  notify:
      after_n_builds: 2             # ステータスを送信する前に、Codecovはアップロードされたレポートの受信を何回待つか。
      wait_for_ci: yes              # Codecovは自分たちのステータスを送信する前に、すべてのCIステータスが完了するのを待つか。
```

<br>

# [coverage:](https://docs.codecov.com/docs/codecovyml-reference#coverage)
coverageセクションでは、カバレッジに関する設定ができる。
```yaml
coverage:
  range: "60...80" # 表示される色の閾値の設定。
  round: down      # 設定した小数点以下の対応。
  precision: 2     # 小数点第何位まで表示するかの設定。
  notify:
    # notification blocks. 
  status:
    # status blocks.
```

**[range:](https://docs.codecov.com/docs/coverage-configuration#range)**
この値は、Codecovの可視色範囲をカスタマイズするために使用される。
例えば、`60...80`と設定すると、カバレッジが50%未満の場合は赤色の背景が表示され、50%から75%の間は黄色のバーが表示され、カバレッジが75%に達すると色が緑に変わる。

**[round:](https://docs.codecov.com/docs/coverage-configuration#rounding)**
カバレッジの小数点表示の設定。デフォルトだと切り捨て設定。
以下の設定が可能。
- **up**
- **down**
- **nearest**

**[precision:](https://docs.codecov.com/docs/coverage-configuration#precision)**
カバレッジの小数点第何位まで表示するかの設定。桁数を0~5で設定できる。

**[notify:](https://docs.codecov.com/docs/notifications)**
カバレッジの結果をslack連携して通知したいなどの場合は、ここに設定していく。
```yaml
coverage:
  notify:
    url: "https://url.to.service"
    branches': 
    - master
    - dev
    - staging
    threshold': 1%
    flags: 
    - backend
    - frontend
    base: "parent"
    only_pulls: false
    paths: "*/**/*"
```

**[status:](https://docs.codecov.com/docs/commit-status)**
特定のカバレッジ閾値を満たさないPRをブロックできる。
```yaml
coverage:
  status:
    patch:
      # patch status blocks.
    default_rules:
      # default rules.
    project:
      default:
        # basic settings
        target: ○% || auto # カバレッジの基準値。autoの場合は前回のベースコミット時のカバレッジが基準値となる。
        threshold: ○%      # 前回のカバレッジとの差分閾値。前回から○%以上カバレッジが落ちる場合はステータスチェック失敗となる。
        base: auto 
        flags: 
          - unit
        paths: 
          - "src"
       # advanced settings
        branches: 
          - master
        if_ci_failed: error #success, failure, error, ignore
        informational: false
        only_pulls: false
        removed_code_behavior:
```
<br>

カバレッジ結果によってマージをブロックしたりする必要がない（カバレッジ率をPR時などに確認したいだけ）場合は、明示的に情報提供モードを設定できる。[「Tips and Tricks」](https://docs.codecov.com/docs/quick-start#tips-and-tricks)
```yaml: codecov.yml
coverage:
  status:
    project:
      default:
        informational: true
    patch:
      default:
        informational: true
```
<br>

以下の設定にすると、カバレッジステータス自体を無効にし、カバレッジ情報をPRなどに表示しない。[「Disabling a status」](https://docs.codecov.com/docs/commit-status#disabling-a-status)
```yaml: codecov.yml
coverage:
  status:
    project: off
    patch: off
```

<br>
<br>


# [comment:](https://docs.codecov.com/docs/pull-request-comments)
commentセクションでは、カバレッジ結果をPRコメントとして表示する際の設定ができる。
```yaml: codecov.yml
comment:
  layout: " diff, flags, files" # コメントレイアウトのカスタマイズ。
  behavior: default             # PRへのコメント投稿方法を選択。
  require_changes: false        # trueの場合、カバレッジが変更された場合のみコメントを投稿。
```

**[layout:](https://docs.codecov.com/docs/pull-request-comments#layout)**
コメントレイアウトをカスタマイズできる。
以下は、追加・選択できるコンポーネント。レイアウトからこれらのいずれかを省略すると、表示されなくなる。

- **Reach** : コメントに埋め込まれたカバレッジグラフ。
- **Diff** : プルリクエストのカバレッジDiff。
- **Flags** : ユーザーが定義したFlagsのリストと、そのカバレッジへの影響。
- **Files、Tree** : プルリクエストによって影響を受けるファイルのリスト（カバレッジの変更、ファイルの新規作成、削除）。
- **Header** : コメントの一番上にあり、マージされる二つのコミット、カバレッジの変更、差分カバレッジを表示。
- **Footer** : コメントの一番下で、凡例、最終更新コミット、ドキュメントへのリンクを表示。
- **Feedback** : Codecov にフィードバックを提供するためのリンクを提供する。


**[behavior:](https://docs.codecov.com/docs/pull-request-comments#behavior)**
Codecovがプルリクエストへのコメント投稿方法を選択。

- **default**: 存在すれば更新。そうでなければ新規投稿。
- **once**: 存在すれば更新。そうでなければ新規投稿。削除されたらスキップ。
- **new**: 古いものを削除し、新しいものを投稿。

**[require_changes:](https://docs.codecov.com/docs/pull-request-comments#requiring-changes)**
カバレッジに変更が検出された場合に、コメントが投稿されるタイミングを変更する。

trueにすると、コメントはカバレッジが変更されたときにのみ投稿されるようになる。さらに、すでにコメントが存在し、新しいコミットによって全体のカバレッジに変更がなかった場合、そのコメントは削除される。

<br>
<br>

# [flags:](https://docs.codecov.com/docs/flags)
flagsセクションでは、テストの種類やサブプロジェクト/チーム毎に、カバレッジレポートをグループ化することができる。

フラグを使用すると、プロジェクト内のさまざまなテストや機能のカバレッジレポートを分離して分類することができる。
これは特に以下のような場合に便利。
- テストの種類が複数ある場合。 (例: ユニットテスト、統合テスト、フロントエンドテスト、バックエンドテストなど)
- モノレポのセットアップを採用しており、各プロジェクトのテストカバレッジを個別にカプセル化したい場合。

<br>

flagsの管理には以下の2つの方法がある。
1つは、フラグの管理を自動化し、デフォルトのルールに基づいてフラグを処理するための方法。
一方は、各フラグを個別に管理し、特定の要件に合わせてカスタマイズするための方法。

1. **Recommended: Automatic Flag Management:**
    Codecovが推奨するフラグの管理方法。
    YAMLの`flag_management:`セクション下で設定。
    `default_rules:`には、追加されるフラグに対して自動的に適用されるルールを設定する。
    特定のフラグに対してデフォルトのルールを上書きする場合は、`individual_flags_rules:`を使用してフラグ毎にカスタムルールを設定できる。

```yaml: codecov.yml
flag_management:
    default_rules: # the rules that will be followed for any flag added, generally
    carryforward: true
    statuses:
        - type: project
        target: auto
        threshold: 1%
        - type: patch
        target: 90%
    individual_flags: # exceptions to the default rules above, stated flag by flag
    - name: feature_1  #fill in your own flag name
        paths: 
        - src/feature_1  #fill in your own path. Note, accepts globs, not regexes
        carryforward: true
        statuses:
        - type: project
            target: 20%
        - type: patch
            target: 100%
    - name: feature_2 #fill in your own flag name
        paths: 
        - src/feature_2  #fill in your own path. Note, accepts globs, not regexes
        carryforward: true
        statuses:
        - type: project
        target: 20%
        - type: patch
        target: 100%
```

2. **Advanced: Bespoke Flag Management:**
    より高度なフラグの管理方法で、すべてのフラグを個別にYAMLに追加する必要するため、手動での作業が多くなる可能性がある。
    ドキュメントでは、モノリポの各プロジェクトでフラグと個別のカバレッジゲートを生成するためのYAMLの構造が示されている。
    この設定により、特定のカバレッジ基準を満たさない場合、プルリクエストが失敗するようになる。
```yaml: codecov.yml
# Setting coverage targets per flag
coverage:
  status:
    project:
      default:
        target: 90% #overall project/ repo coverage
      front-end:
        target: 60%
       flags:
          - front-end
      back-end:
        target: 100%
        flags:
          - back-end
      mobile:
        target: 80%
        flags:
          - mobile

# adding Flags to your `layout` configuration to show up in the PR comment
comment:
  layout: "reach, diff, flags, files"
  behavior: default
  require_changes: false  
  require_base: yes
  require_head: yes       
  branches: null

# New root YAML section = `flags:`
# This is where you would define every flag from your
# uploader, and update when new Flags added

flags:
  front-end:
    paths: #note, accepts globs, not regexes
      - src/front-end/code.js
    carryforward: false
  back-end:
    paths: #note, accepts globs, not regexes
      - src/back-end/api_code.py
    carryforward: true
  mobile:
    paths: #note, accepts globs, not regexes
      - src/new/mobile/app_code.java
    carryforward: true
```



**[Carryforward Flags](https://docs.codecov.com/docs/carryforward-flags)**
各コミットでリポジトリのコードすべてをテストしない場合、Codecov では`Carryforward Flags`と呼ばれる機能を使用して、実行されたテストのカバレッジのみを更新することができる。


<br>

# テスト結果[（Coverage Delta(Δ)）](https://docs.codecov.com/docs/codecov-delta)
- **absolute** : プロジェクト全体のカバレッジを表示。
- **<relative>** : コミットの差分におけるカバレッジを表示。
- **(change)**: コミットの親と比較したカバレッジの増減値を表示。
- **ø** : カバレッジに変更がなかったことを表す。
  
【Coverage Δ】
| absolute | <relative> | (change) |
| -------- | ---------- | -------- |
| 35%      | <72%>      | (+4%)    |
| 94.44%   | <94.44%>   | (ø)      |


