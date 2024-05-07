---
title: "【Flutter】fastlaneでTestFlightを配信する"
emoji: "📤"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, dart, fastlane, TestFlight, ios]
published: true
publication_name: ncdc
---

タイトル通り、fastlane で TestFlight を配信できるようにしていきます。

<br>

# はじめに

fastlane の導入方法等については、この記事では記載しません。
fastlane の導入方法等は、こちらの記事も参照してください。
https://zenn.dev/ncdc/articles/fastlane_match#fastlane-match-%E3%81%AE%E5%B0%8E%E5%85%A5

<br>

# 全体像

```ruby: Fastfile
platform :ios do
  desc "Push to TestFlight"
  lane :upload_testflight do
    # 1.実行ブランチ設定
    # 特定のbranch以外で実行された場合はエラーを出力。
    current_branch = `git rev-parse --abbrev-ref HEAD`.strip
    if current_branch != "<branch名>"
      UI.user_error!("This lane can only be run from the '<branch名>' branch.")
    end

    # 2.API Key設定
    # ここではenvファイルを使用して環境変数を管理。
    app_store_connect_api_key(
      key_id: ENV["ASC_KEY_ID"],
      issuer_id: ENV["ASC_ISSUER_ID"],
      key_filepath: ENV["ASC_KEY_FILEPATH"],
      duration: 1200,
      in_house: false
    )

    # 3.ビルド番号設定
    # TestFlightにアップロードされている最新のビルド番号を取得してインクリメント
    increment_build_number({
      build_number: latest_testflight_build_number + 1
    })

    # 4.flutterビルド
    sh( "fvm","flutter","clean" )
    sh( "fvm","flutter","build","ios","--dart-define-from-file=dart_defines/dev.json" )

    # 5.xcodeビルド(Archive実行)
    build_app(
      clean: true,
      silent: true,
      workspace: "Runner.xcworkspace", # 各自プロジェクトに合わせて変更
      scheme: "Runner", # 各自プロジェクトに合わせて変更
      output_directory: "<出力先pathを指定>",
      output_name: "<<出力されるファイル名>.ipa>",
      export_method: "app-store",
      export_options: {
        provisioningProfiles: {
          "<identifer名>" => "<provisioning profile名>",
        }
      },
    )

    # 6.TestFlightにアップロード
    upload_to_testflight
  end
end

```

<br>

# 各処理の詳細、補足

### 1.実行ブランチ設定

開発途中のブランチとかで間違って実行しないよう、特定のブランチでしか実行できないようにしました。
主には`production`や`develop`ブランチとかになるのかと思います。

<br>

### 2.API Key 設定

AppStoreConnect の API キーを設定します。
env ファイルに環境変数を設定して、それを読み込むようにしています。
env ファイルの設定詳細は、[「はじめに」](#はじめに)で記載した記事を参照してください。

<br>

## 3.ビルド番号設定

ビルド番号は、TestFlight にアップロードされている最新のビルド番号を取得してインクリメントするようにしました。
こうすることで、`pubspec.yaml`の`version`の値に依存せずで、良いかなと。

※ ビルド番号の管理は android 側と合わせたいとかあると思いますので、お好みの管理方法で。

<br>

## 4.flutter ビルド

flutter コマンドを`sh`で実行します。

<br>

**【補足】**
fastlane の関数の[build_app](#build_app)と、flutter コマンドのビルドとの違いがよく分からず、
「どっちも build するのでどっちかの処理だけでよいのでは？build_app だけで`--dart-define`も適用されるようにできないのか？」
などと色々と詰まりました。。。

→ 違いは以下の記事で大方理解できました。（言われたら確かに、xcode でビルドした際はビルド後に Archive 実行してたなと、、）
https://medium.com/flutter-jp/ipa-e176de0276c6

<br>

なので、`flutter build`と`build_app`の違いが理解できたら、`build_app`を使用せずに`flutter build ipa`だけの方がパフォーマンス高いのではと思ったので、以下のようなコードでも良さそうだなと思いました。

```ruby: Fastfile
    sh( "fvm","flutter","clean" )
    sh( "fvm","flutter","build","ipa","--dart-define-from-file=dart_defines/dev.json" )

    upload_to_testflight(
        ipa: "../build/ios/ipa/xxx.ipa",
    )
```

※ `build_app`でやるメリットしては、「色々な設定を明示的に記載でき、チーム間で共有しやすい」とかはあるかもしれないなと。

※ `--dart-define-from-file`は Flutter3.17 以降で非推奨（無効？廃止？）となっているので、今後は別の方法で対応する方がいいのかもしれない。（が、調べた感じベストプラクティスがまだ無さそう？）
https://github.com/flutter/flutter/issues/138793

<br>

## 5.xcode ビルド

build_app 関数は、Xcode で言う`Archive`実行に該当。
[build_app の Document](https://docs.fastlane.tools/actions/build_ios_app/)には沢山のパラメータが用意されているので、必要に応じて設定してください。

<br>

## 6.TestFlight にアップロード

作成された ipa ファイルを TestFlight にアップロードする処理です。
今回はパラメータの指定は何も行っていないが、必要に応じて`ipa:`や`changelog:`など、各種設定してください。

[upload_to_testflight：Document](https://docs.fastlane.tools/actions/upload_to_testflight/)

<br>

# 終わり

- 今回使用した fastlane 提供の関数はエイリアス[^1]が提供されているので、そちらを使用しても問題なく実行できます。

  - `build_app` は `gym`、`upload_to_testflight`は`pilot`に変更しても同じ結果となります。
  - ドキュメント的にはエイリアスされた方がメイン？の記載になっているので、エイリアス版を使用する方が良いかもです。
    ![](https://storage.googleapis.com/zenn-user-upload/13996d7b2b4c-20240507.png)

- ios だけのリリースだと XcodeCloud でも良さそうだけど、android も fastlane 処理を組んでいるなら fastlane で統一した方がよいか。とかとか難しいところです。。。(試したいことは他にも沢山。)

<br>

[^1]: エイリアス：一般的にはあるオブジェクト、コマンド、関数などに別の名前を割り当てることを指します。この別名を使用することで、より覚えやすかったり、タイプしやすかったりする名前で同じ操作を実行できるようになります。エイリアスは複雑なコマンドや長いコードを短くシンプルにするために使われます。
