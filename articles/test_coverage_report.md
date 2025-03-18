---
title: "カバレッジ測定対象外のファイルもカバレッジレポートに含めたい"
emoji: "📊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, dart, coverage, test, codecov]
published: false
# publication_name: ncdc
---

:::message
本記事の test 部分は Flutter で書いてますが、カバレッジに関する内容なので、他の言語でも基本的な考え方は共通な認識です。
jest などでも同様のことが検討できるかと思います。
:::

## はじめに

お馴染みの`flutter test --coverage` コマンド。
デフォルトでは、`test/`配下のテストコードから、直接または間接的に参照されるファイルのみがカバレッジ計測対象となります。

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
  // testコマンド自体は成功するが、テスト結果としてはカバレッジ計測は何もされない。= lcov.infoの中身が空。
  test('sample1', () {
    expect(1, 1);
  });
}
```

<br>

であれば極論、
「テストコードが import1 つだけの独立した 1 つのファイル」かつ「そのファイルに関してはテストが網羅してある場合」、パット見は PJ のカバレッジは 100%であるように見えてしまいます。

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

## やったこと

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

## 備考・補足

もちろん、このやり方での課題はいくつか出てきます。

1. **ダミーデータ追加による影響**
   - `flutter test --coverage` 本来の実行結果とは異なるカバレッジレポートが生成されます。
     - ただし、「1 ファイルにつき 1 行のダミーデータ」が追加されるだけであり、全体のカバレッジ率への影響は小さいと考えられます。
     - 「テストが実施されていない部分を明確にする」 という部分をカバレッジ測定の主な目的とするならば、影響は小さいと考えられます。
2. **全ての `.dart` ファイルがカバレッジ対象となる**
   - デフォルトの `flutter test` では、カバレッジ対象となる `.dart` ファイルを自動で選別してくれますが、上記のスクリプトは 「lib 配下の全ての`.dart`ファイル」が対象となります。
     - `enum` 、`style` 、`const` などの定義のみのファイルも、テスト対象としてカバレッジレポートに含まれます。
       - ※ スクリプトを改善し、適切なファイルのみを選別できるようにすれば、より精度の高いカバレッジレポートを得られるとは思います。
3. **不要なファイルの除外**
   - 不要なファイルをそのまま許容する場合
     - カバレッジの主な目的を 「テストされていない部分を可視化すること」 とするならば、定数やスタイル定義のファイルがカバレッジレポートに含まれてても良しとするのも一つの選択肢です。
   - 不要なファイルを除外する場合
     - ディレクトリ構成を考慮し、適切に除外設定を行うことで、不要なファイルがカバレッジレポートに含まれるのを防げます。
     - Codecov などのカバレッジレポートツールを使用する場合、`codecov.yml` で除外設定を記載。
       - 例えば、`constant/`や`theme/`、view ファイル命名規則の`**_screen.dart`を除外する設定、など。
       ```yaml: codecov.yml
       ignore:
         - "**/constant/**"
         - "**/theme/**"
         - "**_screen.dart"
       ```

<br>

## おわり

ディレクトリ構成が整理されていれば、不要なディレクトリを適切に除外することで、実用的なカバレッジレポートを得ることができるかと思います。
運用を続ける中で、より適切な設定や改善点が見えてくるはずなので、必要に応じて適宜調整していきます。
