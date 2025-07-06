---
title: "【Swift】実行環境毎に異なる設定値を指定したい"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Swift, Xcode, iOS]
published: true
publication_name: ncdc
---

Xcode の Configuration 設定にて、Debug と Release とで BundleID やアイコン等、異なる値を設定する手順を記載&メモしていきます。

※ 検証時の Xcode バージョン: 16.2

![](https://storage.googleapis.com/zenn-user-upload/dd3fb72aa2b3-20250703.png)

## やったこと

1. 適当なフォルダ階層に、`.xcconfig` ファイルを実行環境毎に作成します。
   新しいファイルをテンプレートから作成します。検索で "config" と入力すれば出てきます。
   ![](https://storage.googleapis.com/zenn-user-upload/7a444c84d839-20250703.png)
   Debug と Release とで実行環境を分けたいので、それぞれの "xcconfig" ファイルを作成します。
   今回の例では、`Debug.xcconfig` と `Release.xcconfig` を作成するとします。
   ![](https://storage.googleapis.com/zenn-user-upload/c43c2c7247b1-20250703.png)

   <br>

   生成されるとコメントだけの空の "xcconfig" ファイルが生成されます。そこに実行環境毎に切り分けたい値を定義していきます。
   今回は 「BundleID、アイコン、アプリ名」を設定してみます。

   ```xcconfig
    // Debug.xcconfig（Release.xcconfig も同様に定義）

    DisplayName = DebugSample
    BUNDLE_IDENTIFIER = jp.co.hoge.sample.debug
    APPICON_NAME = AppIconDebug
   ```

   CocoaPods を使用している場合、CocoaPods が生成した "xcconfig" があれば、そちらも読み込むように Path を指定します。

   ```xcconfig Debug.xcconfig
    // Debug.xcconfig（Release.xcconfig も同様に定義）

    #include "Pods/Target Support Files/Pods-PJ_Name/Pods-PJ_Name.debug.xcconfig"

    DisplayName = DebugSample
    BUNDLE_IDENTIFIER = jp.co.hoge.sample.debug
    APPICON_NAME = AppIconDebug
   ```

   <br>

   ※ 変数名は自由です。困ったら実際に使用されている Key 名などがよいかもしれません。（以下`project.pbxproj`の差分）

   ```diff pbxproj
   33D9A3BE2E1506E400FA2DF6 /* Debug */ = {
        buildSettings = {
   -		INFOPLIST_KEY_CFBundleDisplayName = "jp.co.hoge.sample";
   +		INFOPLIST_KEY_CFBundleDisplayName = "$(DisplayName)";
                   ...
   -		ASSETCATALOG_COMPILER_APPICON_NAME = AppIcon;
   +		ASSETCATALOG_COMPILER_APPICON_NAME = "$(APPICON_NAME)";
                   ...
   +		PRODUCT_BUNDLE_IDENTIFIER = "$(BUNDLE_IDENTIFIER)";
   ```

   <br>

2. 作成した"xcconfig" ファイルを、Configurations に設定します。（今回は Test の Target は無視してます。）
   ![](https://storage.googleapis.com/zenn-user-upload/68308bc1f0b6-20250703.png)

   <br>

3. "xcconfig"ファイルで設定した環境変数それぞれを、対象 TARGET の BuildSettings 等から設定します。
   スクロールで見つけようとすると項目数が多くて大変なので、いい感じに検索すればヒットします。
   ![](https://storage.googleapis.com/zenn-user-upload/15fc4ea2c09c-20250706.png)

   <br>

4. Scheme 設定を実行環境毎に作成
   添付の箇所から新しい Scheme を作成します。（Scheme 名はわかりやすく`<PJ>_Debug`、`<PJ>_Release`としています。）
   ![](https://storage.googleapis.com/zenn-user-upload/77e2904c182b-20250706.png =400x)

   作成した Scheme を選択し、"Edit Scheme..." から Build Configuration をそれぞれの実行環境に設定します。
   ![](https://storage.googleapis.com/zenn-user-upload/56e61d1d3f2f-20250706.png)
   ![](https://storage.googleapis.com/zenn-user-upload/c5f990518199-20250706.png)

    <br>

5. Debug と Release の Scheme を選択してそれぞれビルドします。
   BundleID が違うので、別アプリとしてそれぞれシミュレーターにインストールされることが確認できました。
   ![](https://storage.googleapis.com/zenn-user-upload/3962471f0fc9-20250706.png)

## 参照記事

:::details 参照記事
https://zenn.dev/satococoa/articles/b873b96b6dd81d
https://qiita.com/hirothings/items/7f6943db609ff88c10be#cocoapods%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%81%84%E3%82%8B%E5%A0%B4%E5%90%88pods%E3%81%AE%E8%A8%AD%E5%AE%9A%E3%81%AEinclude%E3%81%8C%E5%BF%85%E8%A6%81
https://softmoco.com/basics/how-to-set-app-icon-in-xcode.php

:::
