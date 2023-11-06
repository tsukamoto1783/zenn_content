---
title: "【Codecov】PRページに表示されるカバレッジ結果の見方"
emoji: "📊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Codecov, GitHub, PR, カバレッジ, テスト]
published: true
publication_name: ncdc
---

# GitHubのPR時に表示されるcodecovの計測結果の見方
![](https://storage.googleapis.com/zenn-user-upload/bd5714b0de22-20231020.png)
*↑GitHubのPR時に表示されるcodecov例*

## Attention: ○lines in your changes ...
```
Attention: 12 lines in your changes are missing coverage. Please review.
```
コミット内の変更があったファイルに対して、カバレッジが欠けている行を示している。
上記の例だと12行分テストがカバーされて無いことがわかる。
GitHubのPRページの`File changed`を確認すると、警告がある行それぞれにコメントが入っていることが確認できる。
![](https://storage.googleapis.com/zenn-user-upload/2763ac9b12ac-20231019.png)

<br>

## Comparison is base ...
```
Comparison is base (xxxxxxx) 8.05% compared to head (xxxxxxx) 8.78%.
```
ベースbranchのcommitから、今回のcommitまでの全体のカバレッジ変更率を示している。
上記の例だと、ベースbranchのcommitから今回のcommitまでのカバレッジの変化は`+0.73%`であることがわかる。

<br>

## Additional details and impacted files
```diff yml: Additional details and impacted files
# 値は全てプロジェクト全体の値
======================================== 
@@           Coverage Diff            @@ 
##             dev    #607      +/-   ## 
======================================== 
+ Coverage   8.05%   8.78%   +0.73%      
======================================== 
  Files         51      61      +10      
  Lines       1962    2492     +530      
======================================== 
+ Hits         158     219      +61      
- Misses      1804    2273     +469      
```

### Coverage Diff
ベースbranchからの変更によるカバレッジの増減を示す。
左側がベースbranchの値、右側が今回のcommitの値。

### Coverage
カバレッジ率の変化を表示。

### Files
ファイル数の変化を表示。

### Lines
行数の変化を表示。

### Hits
テストによって実行されたコード行の数を表示。

### Misses
テストによって実行されていないコード行の数を表示。

【Tips】
[Attention: ○lines ...](#attention-○lines-in-your-changes)で表示されている行数と、`Misses`で表示される行数は、どちらもカバレッジが欠けている行数を示しているが違いは何か？

`Attention ...` の部分で表示されている行は、今回の変更で追加した行が表示されているので分かる。
しかし、`Misses`の部分で追加された未確認の行は`+469`行と表示されていて、そもそもそんなにファイルの変更をしていない。これはなに？

codecovのコンソールを確認してみる。
![](https://storage.googleapis.com/zenn-user-upload/efdbf30dc14a-20231020.png)

`file changed`と`indirect changes`が存在する。
見慣れない`indirect changes`を見てみると、変更を加えていないファイルに変更がかかっているのが確認できた。
また、注意書きには「これらは著者のリビジョンがないファイルだが、予期せぬカバレッジの変更が含まれている。」と書かれており、ドキュメントを見ると「カバレッジが予期せぬ形で間接的に変更される...」との記載があり色々と原因が記載されている。
どうやらこの`indirect changes`が`Misses`に影響を与えてそう。


なので、とりあえずは`indirect changes`と `Misses`については、あまり気にせずに、
それよりは`Attension ...`の部分で表示されている行をまずは優先で確認することが重要かと。

余裕があればドキュメントに記載のあるように、「indirect changes」の根本原因を探して改善していくのが良さそう。

[Codecov Docs / Unexpected Coverage Changes](https://docs.codecov.com/docs/unexpected-coverage-changes)

## Flag
フラグ設定している場合は、そのフラグ毎のカバレッジ情報を表示。

## Files
変更のあったファイル毎のカバレッジ情報を表示。


# [Coverage Delta(Δ)](https://docs.codecov.com/docs/codecov-delta)
Coverage Δ 例
| absolute | <relative> | (change)  |
| -------- | ---------- | --------- |
| 8.78%    | <52.00%>   | (+0.73%)  |
| 94.44%   | <94.44%>   | (ø)       |
| 68.84%   | <100.00%>  | (+13.36%) |

- **absolute** : commit後のプロジェクト全体のカバレッジ率を表示。
- **<relative>** : commit分だけのカバレッジ率を表示。
- **(change)**: ベースbranchと今回のcommit後とを比較したカバレッジの増減値を表示。
- **ø** : カバレッジに変更がなかったことを表す。
