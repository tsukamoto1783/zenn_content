---
title: "【Android】cloneアプリのapplicationIdなどを変更して別アプリとして流用する"
emoji: "🏷️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Android, Kotlin, java, AndroidStudio, applicationId]
published: true
publication_name: ncdc
---

Android アプリ開発において、既存プロジェクトやテンプレートプロジェクトを clone して流用したい場面があると思います。
または、適当に applicationId を設定したものの、後から正式なものに変更したい。などの場合もあるあるかと思います。

そんな場合の applicationId や Project 名などを変更する方法についてざっとまとめてみました。

<br>

## 検証環境

- Android Studio Narwhal Feature Drop | 2025.1.2
- 検証前の applicationId は `com.example.test.app` と仮定します。

※ 普段 VSCode を主に使用しています。AndroidStudio の操作や表示に慣れておらず、VSCode を併用した記載となります。
慣れている方は AndroidStudio だけでも問題無いと思います。

<br>

## やること

### 0. ディレクトリ表示設定の変更（任意）

添付の通り設定から`Compact Middle Packages`のチェックを外す。
チェックを外すことで、VSCode などのエディタで見慣れたディレクトリ構造の表示となります。

![](https://storage.googleapis.com/zenn-user-upload/20fdd7fc3c6a-20250901.png)

※ 見慣れた使いやすい形式で各自設定をお願いします。

|                                  チェック有り                                  |                                  チェック無し                                  |
| :----------------------------------------------------------------------------: | :----------------------------------------------------------------------------: |
| ![](https://storage.googleapis.com/zenn-user-upload/f89cf714cd7f-20250901.png) | ![](https://storage.googleapis.com/zenn-user-upload/4db21f28ba87-20250901.png) |

<br>

### 1. applicationId

まずは新しい `applicationId` を考えましょう。

**`applicationId` とは、アプリを識別するための一意の ID です。**

ユーザーが直接目にすることは基本は無い設定値とはなりますが、Google Play ストアでアプリを公開する際の重要な設定値となります。

今回は新たに `jp.co.sample.newApp` とします。

VSCode で`app/build.gradle.kts`を開いて変更します。

```diff: app/build.gradle.kts
android {
    // ===== 省略 =====
    defaultConfig {
-        applicationId = "com.example.test.app"
+        applicationId = "jp.co.sample.newApp"
    // ====== 省略 =====
    }
```

<br>

### 2. namespace

`applicationId`の変更に伴い、`namespace`の値も変更します。

`namespace`についてはドキュメントを見てもややこしいところなので、ざっくりまとめると、
**自動生成されるコードを、どこに配置するかを指定する設定値です**
この値によって自動生成されるクラスのパッケージ構造（ディレクトリ構造）が決まります。

こちらの設定値も、ユーザーが直接目にすることは基本無い設定値となります。

通常は`applicationId`と同じ値にすることが推奨されています。
新規プロジェクト作成時も同じ値が設定されます。

なので今回も同じ値に変更します。

VSCode で`app/build.gradle.kts`を開いて変更します。

```diff: app/build.gradle.kts
android {
-    namespace = "com.example.test.app"
+    namespace = "jp.co.sample.newApp"
    // ===== 省略 =====
```

※ `namespace`についての詳細については以下参照ください。
https://developer.android.com/build/configure-app-module?hl=ja
https://medium.com/@MahabubKarim/understanding-namespace-and-applicationid-in-android-gradle-718c685db3d2

<br>

### 3. rootProject.name

続いて`rootProject.name`を変更します。

**`rootProject.name`とは、開発環境（Android Studio や Gradle）内でプロジェクトを識別するための名前です。**
**アプリ自体の動作や配布には影響しない内部的な設定値です。**（具体的には以下）

- namespace
- applicationId
- app_name

Android Studio のウィンドウ上部に表示されるプロジェクト名にもなります。

必ず`applicationId`と同じ値にする必要はありませんが、たいたい似たような名前にすることが多いかと思います。

VSCode で`settings.gradle.kts`を開き、今回は`SampleNewApp`として変更します。

```diff: settings.gradle.kts
- rootProject.name = "com-example-testApp"
+ rootProject.name = "SampleNewApp"
```

<br>

### 4. アプリ名（app_name）

Android 端末にインストールされたアプリアイコンの下に表示されるアプリ名を変更します。
`res/values/strings.xml`に定義されている`app_name`を変更します。

```diff: string.xml
-    <string name="app_name">com-example-testApp</string>
+    <string name="app_name">TestApp</string>
```

<br>

### 5. 自動生成ファイルなどの削除

このままだと上記の設定値の変更が上手く反映されない可能性が高いので、キャッシュファイルなどの自動生成ファイルを一度削除します。

削除するフォルダは、`.idea/`（IDE 設定類）、`.gradle/`（キャッシュ類） です。

上記２つのファイルを VSCode 上から削除します。

![](https://storage.googleapis.com/zenn-user-upload/e6bc81fe7fa2-20250901.png)

ここでようやく AndroidStudio を起動します。

![](https://storage.googleapis.com/zenn-user-upload/746efbd01c56-20250901.png)

※ 赤線のように、ローカルのフォルダ名と`rootProject.name`がそれぞれ表示されます。
※ 上記の「.フォルダ類」を消さないと`rootProject.name`などが反映されない場合があります。

<br>

### 6. ディレクトリ構造の修正

`namespace`にあわせて、アプリのディレクトリ構造を修正します。
こちらの修正は AndroidStudio 上の機能で修正したいと思います。

変更したいフォルダを右クリックで、`Rename…`を選択。
`All Directories`を選択して適宜命名を変更します。
これで紐づく箇所を自動で変更してくれます。

|                                       -                                        |                                       -                                        |                                       -                                        |
| :----------------------------------------------------------------------------: | :----------------------------------------------------------------------------: | :----------------------------------------------------------------------------: |
| ![](https://storage.googleapis.com/zenn-user-upload/1d82a129bf18-20250902.png) | ![](https://storage.googleapis.com/zenn-user-upload/e48b81f3e84c-20250902.png) | ![](https://storage.googleapis.com/zenn-user-upload/a55e9f50cf7a-20250902.png) |

以降、配下のフォルダについても、同様に `namespace` にあわせてフォルダ名を変更します。
こちらの対応で、一通りの変更対応は完了です。

|                                     before                                     |                                     after                                      |
| :----------------------------------------------------------------------------: | :----------------------------------------------------------------------------: |
| ![](https://storage.googleapis.com/zenn-user-upload/860cef4c6e30-20250902.png) | ![](https://storage.googleapis.com/zenn-user-upload/d75fc765824d-20250902.png) |

https://developer.android.com/build/configure-app-module?hl=ja
https://kotlinlang.org/docs/coding-conventions.html#source-code-organization

<br>

### 7. ドキュメントファイルの記載などの修正 + 修正漏れチェック

上記の Rename 処理は、.md ファイルなどのドキュメントファイル内の記載は変更してくれません。（import などのひも付きとかは無いので。）
また、同様に他の設定ファイルについても、漏れがないか確認します。

VSCode の検索で、変更前の applicationId や rootProject.name、ディレクトリ構造が記載されている箇所を検索して、必要に応じて修正・置換します。

検索例：

- `com.example.test.app`
- `com/example/test/app`
- etc.

<br>

### 8. ビルド・実行確認

ここまできたら、最後に build して実行してみましょう。
正常に動作すれば完了です。

<br>

## 備考

他記事や過去記事でよく目にする `package名` について。

以前は、`AndroidManifest.xml`にて手動で`package`プロパティを指定する必要がありました。
しかし、「AGP（Android Gradle Plugin）: 8.0」以降、手動設定が廃止され、ビルド後に`applicationId`が自動で`package`に設定されるようになったみたいです。

なので、大まかには 「`package名` ≒ `applicationId`」 という認識で良いかと思っています。

https://developer.android.com/build/configure-app-module
https://developer.android.com/build/releases/past-releases/agp-8-0-0-release-notes
