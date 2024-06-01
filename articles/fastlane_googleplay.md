---
title: "【Flutter】fastlaneでGooglePlayの内部テストを配信する"
emoji: "⛳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, dart, fastlane, PlayStore, android]
published: true
# publication_name: ncdc
---

タイトル通り、fastlane で GooglePlay の内部テストを配信できるようにしていきます。

<br>

# はじめに

fastlane の導入方法等については、この記事では記載しません。
fastlane の導入方法等は、こちらの記事も参照してください。
https://docs.flutter.dev/deployment/cd#fastlane
https://zenn.dev/ncdc/articles/fastlane_match#fastlane-match-%E3%81%AE%E5%B0%8E%E5%85%A5

<br>

# 全体像

```yaml:使用するフォルダ構成
project(root)
└─ android
   └─ fastlane
       ├─ Appfile
       ├─ Fastfile
       └─ <key>.json
```

```ruby: Appfile
# 1.keyやpackageNameの設定
json_key_file("<jsonKey path>") # Path to the json secret file
package_name("<package名>") # e.g. com.krausefx.app
```

```ruby: Fastfile
default_platform(:android)

platform :android do
  desc "buildとGooglePlay（内部テスト）へのアップロード"
  lane :upload_play_store do
    # 2.実行ブランチ設定
    # 特定のbranch以外で実行された場合はエラーを出力。
    current_branch = `git rev-parse --abbrev-ref HEAD`.strip
    if current_branch != "<branch名>"
      UI.user_error!("This lane can only be run from the '<branch名>' branch.")
    end

    # 3.local.propertiesからバージョン名を取得
    properties_path = "../local.properties"
    version_name = ""

    if File.exist?(properties_path)
      File.open(properties_path, 'r') do |file|
        file.each_line do |line|
          if line.include?('flutter.versionName')
            # flutter.versionName の値を取得
            version_name = line.split('=')[1].strip
          end
        end
      end
    else
      UI.error("local.properties ファイルが見つかりません。パスを確認してください。")
    end

    # 4.GooglePlayにアップロードされている最新のビルド番号を取得してインクリメント
    previous_build_number = google_play_track_version_codes(
      track: "internal", # internal（内部テスト）を指定
    )[0]
    new_version_code = previous_build_number + 1

    # 5.flutterビルド
    sh( "fvm","flutter","clean" )
    sh( "fvm","flutter","build","appbundle","--dart-define-from-file=dart_defines/dev.json","--build-number=#{new_version_code}" )

    # 6.ビルドされたaabファイルをGooglePlay（内部テスト）にアップロード
    upload_to_play_store(
      track: 'internal', # internal（内部テスト）を指定
      version_name: "#{new_version_code}(#{version_name})",
      aab: "../build/app/outputs/bundle/release/app-release.aab",
      release_status: "draft", # draft（下書き状態）を指定
      skip_upload_apk: true,
      skip_upload_metadata: true,
      skip_upload_images: true,
      skip_upload_screenshots: true
    )
  end
end
```

<br>

# 各処理の詳細、補足

### 1.store の共通設定

AppFile に key 情報やパッケージ名などの共通設定を記載しておくと、Fastfile で fastlane の関数を呼び出す際に毎回指定しなくて済むようになります。

jsonKey に関しては、GoogleCloud のコンソールで API の設定をして jsonKey を取得し、取得した json ファイルパスを指定します。
GoogleCloud コンソールでの設定方法はドキュメント参照。
https://docs.fastlane.tools/actions/upload_to_play_store/

<br>

### 2.実行ブランチ設定

開発途中のブランチとかで間違って実行しないよう、特定のブランチでしか実行できないようにしました。
主には`production`や`develop`ブランチとかになるのかと思います。

<br>

### 3.local.properties からバージョン名を取得

内部テストにアップロードする際の「リリース名」を、手動でアップロードする時と同じ形式にしたかったため、`local.properties`からバージョン名を取得する処理をいれました。

見出し[「6.GooglePlay にアップロード」](#6googleplay-にアップロード)でも記載していますが、upload_to_play_store のデフォルト設定だとバージョン名だけ（ex. 0.1.0）となるため。
→ 添付のように`ビルド番号（バージョン名）`としたかったため。

![](https://storage.googleapis.com/zenn-user-upload/c04f1b54bbcd-20240507.png)

<br>

### 4.ビルド番号設定

[latest_testflight_build_number: Doc](https://docs.fastlane.tools/actions/latest_testflight_build_number/)
ビルド番号は、GooglePlay の内部テスト にアップロードされている最新のビルド番号を取得して、インクリメントするようにしました。

**【補足】**
`track`の値の違い。
[参照：Google Play Developer API](https://developers.google.com/android-publisher/tracks?hl=ja#adding_and_modifying_apks)

- `internal`：内部テスト
- `alpha`：クローズドテスト
- `beta`：オープンテスト
- `production`：製品版リリース

ビルド番号の管理方法について、色々と考え用はありそうなのでお好みでです。
:::details 【参考記事】
https://zenn.dev/miyaken12/articles/57c2a141d318d3
https://backport.net/blog/2019/09/01/flutter_android_app_version/
https://koyari.jp/it/version-code-name/1115/
https://github.com/fastlane/fastlane/issues/878
https://spin.atomicobject.com/version-fastlane/
:::

<br>

## 5.flutter ビルド

flutter コマンドを`sh`で実行します。

**【補足】**
fastlane のアクションの[gradle](https://docs.fastlane.tools/actions/gradle/)と、flutter コマンドのビルドとの違いがよく分からず。

`gradle`アクションだと`--dart-define-from-file`がうまく適用させることができなかったので、`sh`で flutter コマンドを実行するようにしました。
うまいことやれば`gradle`アクションでも適用させれるるかもです。

※ `--dart-define-from-file`は Flutter3.17 以降で非推奨（無効？廃止？）となっているので、今後は別の方法で対応する方がいいのかもしれない。（が、調べた感じベストプラクティスがまだ無さそう？）
https://github.com/flutter/flutter/issues/138793

<br>

## 6.GooglePlay にアップロード

作成された aab ファイルを GooglePlay にアップロードする処理です。
ドキュメントには他にもプロパティの記載があるので、必要に応じて設定してください。
https://docs.fastlane.tools/actions/upload_to_play_store/

<br>

**【注意】**
現状の記載だと、`release_status`が`draft`以外だと以下のエラーになる。
**＝ 内部テストが下書き状態でしかリリースできない。**

```txt
[!] Google Api Error: Invalid request - Only releases with status draft may be created on draft app.
```

原因が不明。自動化したいための fastlane だが、なぜか TestFlight のように配信までが fastlane コマンドから実施できない。最後まで自動化する方法がわからない。

https://support.google.com/googleplay/android-developer/thread/240224208/google-play-api-how-can-i-make-a-draft-release-available-to-internal-testers?hl=en

https://github.com/fastlane/fastlane/discussions/18293

> 初期アルファ版を Play ストアコンソールで直接リリースしていないことを意味します。

初期リリースもしてるはずだけど、エラーは消えない。

https://jonathancardoso.com/en/blog/automated-release-publish-deployment-react-native-android-apps-using-fastlane-part-1-play-store/#sending-alpha-version-to-the-play-store-using-fastlane

→ 引き続き調査。

<br>

**【備考】**

- release_status の値の違い。
  [参照：Google Play Developer API](https://developers.google.com/android-publisher/tracks?hl=ja#apk_workflow_example)

  - `draft`：未公開（下書き状態）、aab ファイルをアップロードはするが、リリースまではしない。
  - `halted`：公開停止
  - `inProgress`：段階的な公開
  - `completed`：即時公開

- リリース名、リリースノートの指定方法
  [GitHub / How to set release name and release note when using upload_to_play_store?](https://github.com/fastlane/fastlane/discussions/21100)

<br>

# 備考

- fastlane を実行する際に、java のバージョンで怒られることがあります。Java と Gradle のバージョンが違うと。
  私は`SDKMAN`というツールで java のバージョン管理をして解決しました。
  参考：
  [「mac で SDKMAN を使って Java のバージョンを管理する」](https://kurukuruway.com/kaihatsu/mac%E3%81%A7sdkman%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6java%E3%81%AE%E3%83%90%E3%83%BC%E3%82%B8%E3%83%A7%E3%83%B3%E3%82%92%E7%AE%A1%E7%90%86%E3%81%99%E3%82%8B/)
  [「AndroidStudio Java のバージョンによるエラーの解決法」](https://qiita.com/TT-RR/items/6866be865c4ab2118030)

  ```txt
  ┌─ Flutter Fix ─────────────────────────────────────────────────────────────────────────────────┐
  │ [!] Your project's Gradle version is incompatible with the Java version that Flutter is using │
  │ for Gradle.                                                                                   │
  │                                                                                               │
  │ To fix this issue, first, check the Java version used by Flutter by running `flutter doctor   │
  │ --verbose`.                                                                                   │
  ```

- 今回使用した fastlane 提供の関数はエイリアスが提供されているので、そちらを使用しても問題なく実行できます。

  - `upload_to_play_store` は `supply`に変更しても同じ結果となります。
  - ドキュメント的にはエイリアスされた方がメイン？の記載になっているので、エイリアス版を使用する方が良いかもです。
    ![](https://storage.googleapis.com/zenn-user-upload/332382fa13c0-20240508.png)

- android の gradle における`assemble`と`bundle`の違い
  - assemble: **apk** ファイルを生成する
  - bundle: **aab** ファイルを生成する
- `fastlane supply init`が実施しようとすると失敗する。
  - init するには本番リリースを一度は実施しておく必要がありそう？
  - 検証中は`fastlane supply init`せずとも構築できた（aab ファイルのアップロードだけなので、metadata とかを使用しなかった）ので、未解決。
  - 参考：
    - https://github.com/fastlane/fastlane/issues/21529
    - https://github.com/fastlane/fastlane/issues/21530
- iOS の TestFlight に関しては、以下の記事参照。
  - https://zenn.dev/ncdc/articles/fastlane_testflight

<br>
<br>

### 参考記事

:::details 【参考記事】

https://qiita.com/mangano-ito/items/c8de5d602928101238ca
https://developers.play.jp/entry/2023/11/24/134115
https://qiita.com/SUGYKEN/items/a1f6b5581d187346d01b#3-fastfile%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B
https://deku.posstree.com/flutter/fastlane/
https://note.com/himaratsu/n/nea4a8cabc3bd
https://developers.play.jp/entry/2022/12/23/144717
https://zenn.dev/aeonpeople/articles/6a2da4ed6e3446
https://isub.co.jp/flutter/flutter-dev-vscode-build-release-deploy-tester/
https://zenn.dev/attomicgm/articles/how_to_use_flutter_fastlane
https://qiita.com/sekitaka_1214/items/9af55223086f29a7d127

:::
