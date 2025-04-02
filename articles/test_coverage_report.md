---
title: "【Flutter】カバレッジ測定対象外のファイルもカバレッジレポートに含めたい"
emoji: "📊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, dart, coverage, test, codecov]
published: false
# publication_name: ncdc
---

## はじめに

お馴染みの`flutter test --coverage` コマンド。
デフォルトでは、`test/`配下のテストコードが実行された際に、直接または間接的に実行されたコードがカバレッジ計測の対象となります。

<br>

例えば、
import 記載のファイルに対してのテストコードが一切書いていない場合、カバレッジは 0%となります。

```dart
import 'package:hoge/utils/sample1.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  // import されているファイルに対して、一切テストコード無い場合、sample1.dartのカバレッジは0%となる。
  test('sample1', () {
    expect(1, 1);
  });
}
```

<br>

import 記載が無く、test 実行時に一切参照されないコードは測定対象外となり、カバレッジレポートに表示されません。

```dart
// importとして参照ファイルの記載が無い場合
import 'package:flutter_test/flutter_test.dart';

void main() {
  // testコマンド自体は成功するが、カバレッジ測定外となるため、カバレッジレポートには表示されない。= lcov.infoの中身が空。
  test('sample1', () {
    expect(1, 1);
  });
}
```

<br>

であれば極論、
「テストコードが import1 つだけの独立した 1 つのファイル」かつ「そのファイルに関してはテストが網羅してある場合」、見かけ上はプロジェクトのカバレッジは 100%であるように見えてしまいます。

```yaml:フォルダ構成例
project(root)
├─ lib
│  ├─ main.dart
│  └─ utils
│     ├─ sample1.dart
│     ├─ sample2.dart
│     ├─ sample3.dart
│     └─ sample4.dart
└─ test
  └─ sample1_test.dart // sample1.dartだけが参照されたテストファイルで、カバレッジ100%とする。
```

```sh:flutter test --coverage
100%
```

これだと表面上はすばらしいですが、カバレッジ計測の意図的には効果が薄い気がするのも事実。。。

<br>

「[Google Testing Blog / Code Coverage Best Practices](https://testing.googleblog.com/2020/08/code-coverage-best-practices.html)」 の記載内容を参考にすると、
「**テストによる最低限の動作保証がされて"いない"のはどの部分なのか、が明らかになること**」が、カバレッジ測定の 1 つのメリットだと読み取れます。

> **In fact a lot of the value of code coverage data is to highlight not what’s covered, but what’s not covered.**

<br>

しかし、デフォルトの設定では、未テストのファイルがカバレッジレポートに含まれないため、どのファイルが未テストなのかが把握しづらい状況に。
（もちろん、「実装時に必ずテストを書く」といったルールが徹底されている場合は、デフォルト設定でも十分かもしれません。）

そこで、`flutter test --coverage` 実行時に、参照されない `.dart` ファイルも測定対象に含める方法を検討してみます。

<br>

## やったこと 1

https://github.com/flutter/flutter/issues/27997#issuecomment-1644224366

上記の Issue コメントの通り、ダミーのテストファイルを作成し、その中で `main.dart` を参照する方法です。
「**テストファイル上で main.dart を参照 ≒ main.dart から直接的および間接的に参照されるファイル全てを参照**」となり、参照される全てのファイルがカバレッジ測定対象となります。

※ 逆に言うと、`main.dart` から参照してる必要があるので、参照されていないファイルは、カバレッジ測定対象外となります。

```dart: fake_test.dart
// ignore: unused_import
import 'package:<app>>/main.dart';

void main() {
  // Fake test in order to make each file reachable by the coverage
}
```

<br>

本来は、unit テストだけでなく、Widget テストや view ファイルのテストも、全部網羅できているのが理想だと思います。
ですが、プロジェクト方針や都合上、「上記のような特定の種類のテストを一時的に見送る」という場合もあるかと思います。

その場合、上記だと参照している全てのファイルが対象となるので、カバレッジが恐ろしく低くなる可能性があります。

`flutter test --coverage` コマンドのオプションには、`ignore` 設定ができるオプションがないので、以下のような対応が別途必要となるかと思います。

### case1: Codecov でのみカバレッジレポートを使用する場合

Github Actions 上でカバレッジ測定し、その結果を Codecov に送信する。
というお馴染みのパターンの場合、Codecov の`codecov.yml` で ignore 設定を行うことで、必要なファイルだけを 表示・計測対象とすることができます。

```yaml: codecov.yml
ignore:
  - "lib/widgets/**"
  - "**/widgets/**"
  - "**_screen.dart"
  - etc.
```

<br>

### case2: カバレッジレポートから指定のファイルを除外する

詳細に以下試せていないが、カバレッジレポートから指定のファイルを除外してから、Codecov に送るでも良さそう。

https://stackoverflow.com/questions/53649089/how-to-exclude-file-in-flutter-test-coverage
https://pub.dev/packages/remove_from_coverage

<br>

### case3: 除外設定をした上で、カバレッジレポートを生成する

（記事作成時の）2 日前に公開されたばかりのライブラリだが、こういったものもあるみたいなので、使い勝手次第では使ってみても良さそう。

https://pub.dev/packages/discover
https://github.com/flutter/flutter/issues/27997#issuecomment-2765655878

<br>

## やったこと 2

基本的には上記の main.dart を参照する方法で問題ないかと思いますが、別の方法でもカバレッジ測定対象外のファイルをカバレッジレポートに含める方法を検討してみました。

※ 自力でカバレッジファイルをいじれば、やりたいように加工できるよ。って程度の内容です。
備忘録程度に記載しておきます。

::::details 【記載詳細】

Github Actions 上でのカバレッジ計測を前提で、一連の処理を記載します。
もちろんターミナル上で順番に手動実行しても同様の結果が得られます。

```yaml
- name: Run Flutter tests with coverage
  run: flutter test --coverage

# lcov をインストール
- name: Install lcov
  run: sudo apt-get update && sudo apt-get install -y lcov

# lib配下のカバレッジ計測対象外の.dartも、カバレッジに含まれるように加工。
- name: Ensure all lib files are included in coverage
  run: |
    # 既存のカバレッジ対象ファイル一覧を取得（`SF:` の後にあるファイルパスを抽出）
    grep 'SF:' coverage/lcov.info | sed 's/SF://' | sort > coverage/existing_files.txt

    # 全て lib 配下の Dart ファイル一覧を取得
    find lib -name "*.dart" | sort > coverage/all_files.txt

    # `lcov.info` に記載されて`いない`ファイルのみを取得
    comm -23 coverage/all_files.txt coverage/existing_files.txt > coverage/missing_files.txt

    # `missing_files.txt` にあるファイルのみ「Tracked lines: 1, coverage: 0%」となる結果データを作成
    while read file; do
      echo "SF:$file"
      awk 'BEGIN {print "DA:1,0\nend_of_record"}'
    done < coverage/missing_files.txt > coverage/lib-files.info

    # 既存の `lcov.info` に追加
    cat coverage/lcov.info coverage/lib-files.info > coverage/combined.info
    mv coverage/combined.info coverage/lcov.info

    # 自動生成ファイルを除外
    lcov --remove coverage/lcov.info 'lib/**/*.g.dart' 'lib/**/*.freezed.dart' 'lib/**/*.gen.dart' 'lib/**/*.mocks.dart' -o coverage/lcov.info --ignore-errors unused

    # 確認用
    lcov --list coverage/lcov.info

# Codecov に結果を送信
- name: Upload coverage to Codecov
  run: # 以降割愛
```

<br>

対応した主な内容としては以下です。
測定対象外のファイルに対して、「Tracked lines: 1, coverage: 0%」となるダミーの測定データを生成し、`lcov.info` に追加しています。

こうすることで、カバレッジ計測対象外のファイルが、カバレッジレポートに含まれるようになり、どのファイルが未テストなのかを確認しやすくなりました。

ここでは、「Total: 1」と記載のファイルが、カバレッジ計測対象外のファイルとなります。

<br>

※ 以下は、上記記載の「フォルダ構成例」のように、`sample1.dart` のみにテストコードが書かれてる場合の例です。
html で出力するために以下のコマンドで表示しています。

```sh
genhtml coverage/lcov.info -o coverage/html
open coverage/html/index.html
```

【before】
![](https://storage.googleapis.com/zenn-user-upload/85a1e31195d3-20250318.png)
【after】
![](https://storage.googleapis.com/zenn-user-upload/c209e88ec4e2-20250318.png)

<br>

**【備考・補足】**

もちろん、このやり方での課題はいくつか出てきます。
「**テストが実施されていない部分を明確にする**」 という観点を主目的とするならば、選択肢の 1 つとしては有りかと思いました。

- ダミーデータであること
  - `flutter test --coverage` 本来の実行結果とは異なるカバレッジレポートが生成されます。
    - ただし、「1 ファイルにつき 1 行のダミーデータ」が追加されるだけであり、全体のカバレッジ率への影響は小さいと考えられます。
- 全ての `.dart` ファイルがカバレッジ対象となる
  - 上記のスクリプトだと、 「lib 配下の全ての`.dart`ファイル」が対象となります。
    - `enum` 、`style` 、`const` などの定義のみのファイルも、テスト対象としてカバレッジレポートに含まれます。
      - スクリプトを改善し、適切なファイルのみを選別できるようにすれば、より精度の高いカバレッジレポートを得られるとは思います。
      - 結局、レポート作成後に何かしらの除外対応は必要となります。上記パターン 1 と同様に。

::::

## おわり

ディレクトリ構成が整理されていれば、不要なディレクトリを適切に除外することで、実用的なカバレッジレポートを得ることができるかと思います。
運用を続ける中で、より適切な設定や改善点が見えてくるはずなので、必要に応じて適宜調整していきます。
