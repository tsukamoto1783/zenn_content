---
title: "【Android/iOS】AR機能についての基礎知識"
emoji: "👓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Android, ARCore, AR, iOS, ARKit]
published: false
# published: true
# publication_name: ncdc
---

**AR：拡張現実（Augmented Reality）**

Android や iOS アプリで AR 機能を実装するにあたり、基礎知識や関係性について調べた内容を記載してみました。
あまり馴染みのない技術知識が詰まった分野なので、ここではざっくり全体理解を目標として記載しています。

主なトピックとしては以下です。

- **ARCore** / **ARKit**
- **SceneView** / **RealityKit**
- **Scene Viewer** / **AR Quick Look**
- **Unity との関係性**

<br>

## ARCore / ARKit

https://developers.google.com/ar?hl=ja

https://developer.apple.com/jp/augmented-reality/arkit/

こちらは AR 機能を実装するための**基盤・土台**となる大元の機能です。

スマートフォンのカメラやセンサーを用いた**モーショントラッキング**や**環境認識**を用いて、現実世界に仮想オブジェクトを配置したり、ユーザーの動きに応じてインタラクティブな体験を提供するための機能を提供しています。

以下は [ARCore のドキュメント](https://developers.google.com/ar/develop)からの抜粋ですが、ARKit も基本的には同様の機能を提供していると考えて良いと思います。（iOS のドキュメントは抽象的な印象なので Google の方が読みやすいです。）

> ARCore は、次の 3 つの主要な機能を使用して仮想コンテンツをスマートフォンのカメラで見た現実世界と統合します。
>
> - **モーショントラッキング**を使用すると、スマートフォンは世界からの相対位置を認識して追跡できます。
> - **環境認識**により、スマートフォンは、水平面、垂直面、地面、コーヒー テーブル、壁などのあらゆる種類の表面のサイズと位置を検出できます。
> - **光の推定**では、スマートフォンの現在の明るさの状態を推定できます。
>
> ======================
>
> 基本的に、ARCore は 2 つのことを行います。**モバイルデバイスの移動による位置の追跡**と、**現実世界に対する独自の理解の構築**です。

<br>

ただし、ARCore や ARKit は、あくまでも AR 機能を実装するための **基盤・土台** であるため、仮想空間に表現する 3D コンテンツの用意や、拡張現実空間上にレンダリングするための制御は、別途用意・実装する必要があります。

<br>

### ARCore と ARKit の違い

基本的には同じような機能を提供してくれますが、その中でも違いや特徴を挙げると、以下のような点が挙げられます。

- **開発元**
  ARCore は Google / ARKit は Apple
- **Target Platform**
  ARCore は マルチプラットフォーム / ARKit は iOS のみ
- **対応デバイス**
  ARCore は幅広い Android デバイスに対応 / ARKit は iOS デバイスに限定
  → リーチできるユーザー数に影響
- **ハイスペック端末の機能**

  - Android: 飛行時間（ToF）ハードウェア深度センサー搭載の端末

    - ARCore 機能自体は基本的にすべての Android 端末で実現可能\*だが、上記のセンサー搭載の端末では、高機能センサー情報も加味されるようになり、精度が向上
      （\*: Depth API は 89%のアクティブなデバイスでサポートされていると[記載](https://developers.google.com/ar/devices?#google_play_devices)）
    - 参照：[ARCore / Use Raw Depth in your Android app](https://developers.google.com/ar/develop/java/depth/raw-depth#device_compatibility)

  - iOS: iPhone 12 以降の **Pro 機種**
    - iPhone / iPad の "**Pro 機種のみ**"、 LiDAR スキャナ と呼ばれる高性能のスキャン機能が有効となり、LiDAR スキャナ搭載端末にのみ有効な ARKit 機能が提供される（Pro 機種にデフォルトで入ってる「測定アプリ」など）
    - 参照：[ARKit](https://developer.apple.com/jp/augmented-reality/arkit/)
    - 参照：[LiDAR スキャナ](https://support.apple.com/ja-jp/102468)

<br>

## SceneView / RealityKit

ARCore や ARKit で構築した基盤に 3D コンテンツを描画するためのメジャーな選択肢となるのが [SceneView](https://github.com/SceneView/sceneview-android) や [RealityKit](https://developer.apple.com/jp/augmented-reality/realitykit/) です。

こちらについては違いがいくつか出てきます。

- **SceneView**
  - シンプルなレンダリング機能に特化
  - 非公式の OSS ライブラリ
- **RealityKit**
  - レンダリング機能
  - 物理シミュレーションやアニメーションなど高度なレンダリングシミュレーションが可能
  - Apple 公式

<br>

### SceneView (Android/Kotlin)

https://github.com/SceneView/sceneview-android
SceneView は、Kotlin での Android アプリにおいて、ARCore で構築した仮想空間上に 3D モデルなどのコンテンツを**簡単にレンダリングするためのライブラリ**です。

Kotlin での Android アプリに SceneView を利用することで、ARCore の機能を活用しつつ、3D モデルの表示やアニメーション、インタラクションなどの処理を簡単に実装できます。

この**レンダリングを ARCore だけで実現しようとすると、レンダリング時の座標調整など必要となるため、かなり複雑となります**。
こだわった表現を実現したい場合を除き、SceneView のようなライブラリを利用することで簡略化にもつながり、基本的な表現であれば必要十分な機能を提供してくれます。

<br>

**【特徴】**

- OSS のライブラリ(Google 非公式)
- 3D モデルのレンダリングに特化
- 外部ツール(Blender 等)で作成したモデルを読み込んで表示

**【備考・背景】**

元々は Google 公式の **Sceneform** というレンダリングライブラリがありましたが、2020 年にサポートが終了しています。

Issue のコメントを見る限り、公式側としては Sceneform の代替策として 3D モデル等の作成には「Unity ARCore SDK for Android」の使用を推奨しています。

しかし、Unity の使用について、Unity が非ゲームアプリ開発に適さない「肥大化したソフトウェア」であり、Android のモダンな開発手法と相性が悪いといった議論が上がったようです。
このような背景から、リポジトリがフォークされ、現在もコミュニティによって開発が続けられているみたいです。

※ 直訳のざっくり要約なので、詳細は Issue のやり取りをご参照ください。
https://github.com/google-ar/arcore-android-sdk/issues/1049

<br>

### RealityKit (iOS)

https://developer.apple.com/jp/augmented-reality/realitykit/
RealityKit は、ARKit で構築した仮想空間上に 3D モデルなどのコンテンツをレンダリングするためのツールキットです。
単にレンダリングするだけでなく、物理演算やアニメーション、空間オーディオなど、高度なレンダリングシミュレーションを実現するための機能も備えています。

<br>

**【特徴】**

- iOS 公式のフレームワーク
- 物理演算、アニメーション、空間オーディオなど、高度なレンダリングシミュレーションが可能
- [Reality Composer](https://developer.apple.com/jp/augmented-reality/tools/) という GUI ツールとの統合により、非エンジニアでも直感的にカスタマイズが可能

**【備考】**

RealityKit や Reality Composer は、iOS に最適化された強力なツールであり、Android 側にはない包括的な機能を提供しています。これは Apple ならではの差別化ポイントと言えます。
ただし、Apple の純正ツールということで、ある程度使いこなせるようになるまでに学習コストがかかる点は一つ考慮ポイントです。

また、特にクロスプラットフォーム開発を前提とする場合はいくつか考慮が必要です。

シンプルに 3D モデルを呼び出すぐらいのアプリであれば、SceneView / RealityKit をそれぞれ使用すれば問題ありませんが、Android アプリで高度なレンダリング設定や高度なアニメーション対応などを行いたい場合は、Unity などの他の外部ツールを使用することになります。
その場合、Unity はクロスプラットフォーム対応しているため、iOS 側も Unity に統一する方が開発効率が良いケースもありうると考えられます。
開発リソースやプラットフォーム戦略に応じて、適切なツールを選択することが重要です。

<br>

## Scene Viewer / AR Quick Look

https://developers.google.com/ar/develop/scene-viewer?hl=ja
https://developer.apple.com/jp/augmented-reality/quick-look/

これらは、**Web ブラウザ**や**アプリ**から AR コンテンツを表示できる、Android / iOS 端末の標準搭載の機能です。

（※ "AR プレビュー"、"WebAR"、"3D モデルビューアー" などとも文脈によっては言われています。）

これらの技術は、フル機能の AR アプリを開発するコストをかけずに、手軽に AR コンテンツを提供したい場合に有効です。

意識せずに使ったこともある機能かもしれませんが、ブラウザ検索結果などにも利用されている機能です。

- **Scene Viewer** (Android)
  - Google が提供
  - Android 端末デフォルトの"Google"アプリ上で動作
  - glb / glTF ファイルに対応
- **AR Quick Look** (iOS)
  - Apple が提供
  - iOS 端末デフォルトの"Safari"アプリ上で起動
  - USDZ ファイルに対応

|                                       Android                                        |                                         iOS                                          |
| :----------------------------------------------------------------------------------: | :----------------------------------------------------------------------------------: |
| ![](https://storage.googleapis.com/zenn-user-upload/68fde4ab471a-20251027.gif =200x) | ![](https://storage.googleapis.com/zenn-user-upload/4f222fbf621d-20251027.gif =200x) |

<br>

## Unity

https://unity.com/ja

AR アプリについて検索をかけると、Unity に関する情報や話題が多く出てきます。
Unity とネイティブアプリの関係性が、初見ではややこしい部分だったので、関係性を整理しておきます。

Unity はゲーム開発に特化した統合開発環境（≒IDE）であり、2D や 3D のゲームやアプリケーションに特化した開発環境を提供しています。

クロスプラットフォーム対応の Flutter や React Native のように、Unity で開発したアプリケーションは、iOS や Android など複数のプラットフォームに対応した形でビルド・配布が可能です。

そのため、関係性としては、
「Kotlin 言語を用いた Android 向けアプリ開発」と、
「Unity（C#言語）を用いた Android 向けアプリ開発」
というように、同じ Android アプリ開発であっても、使用する開発環境や言語が異なる、というイメージです。iOS でも同様です。

<br>

### Unity での AR 開発

Unity での AR 開発では、[AR Foundation](https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.3/manual/index.html) という公式パッケージを使用するのが一般的です。

AR Foundation は、ARCore と ARKit の両方に対応したクロスプラットフォームのフレームワークで、同じコードで iOS と Android の両方に対応した AR アプリを開発できます。

- 単一のコードベースで iOS/Android 両対応
- ARCore と ARKit の機能を統一された API で利用可能
- 高度な 3D 表現やゲーム要素を実装しやすい

ただし、**Unity はあくまでもゲーム特化の開発環境であるため、基本的にはゲームアプリ開発に向いており、ゲーム以外の一般的なモダンなモバイルアプリ開発にはあまり向いていないとされます。**

一般的なモバイルアプリ開発においては、Kotlin や Swift を用いたネイティブ開発が主流であり、Unity は 2D・3D(ゲーム)開発に特化した選択肢として位置付けられています。

（※ Unity 開発にはあまり詳しくないので、語弊があればご指摘ください）

<br>

## 整理

以上を踏まえると、AR 機能をネイティブアプリに実装する場合、以下のような選択肢が一般的に採用されます。

### 1. Kotlin + ARCore + SceneView

主な AR 機能としては、あらかじめ用意された 3D モデルを現実空間に表示する、ユーザーの動きに応じて 3D モデルを操作する、といった基本的な AR 体験を提供するアプリを開発できます。
前述の通り、表示するコンテンツ自体は呼び出す(読み込む)だけなので、比較的シンプルな実装で済みます。

### 2. Swift + ARKit + RealityKit

こちらは iOS 純正のツールを用いた構成例です。
どれも iOS に特化した作りであり、高性能なアプリが作れる反面、専門的な知識が求められます。
もちろん、上記 1.のように簡単なモデルを呼び出すぐらいであれば、比較的シンプルに実装は可能です。

### 3. Unity + AR Foundation (ARCore/ARKit)

基本的に AR でのゲームアプリ開発などで用いられます。
より高度な 3D 表現やインタラクションを実現したい場合に適しています。
AR Foundation を使用することで、単一のコードベースで iOS/Android 両対応が可能です。
一般的なモバイルアプリをわざわざ Unity で開発するケースは少ないですが、ゲーム要素を含む AR アプリには最適かと思われます。

### 4. Kotlin/Swift + Unity

Unity で開発した AR コンテンツをネイティブアプリに組み込む場合に採用されます。
ネイティブアプリ上の一部だけ、高度な AR コンテンツや AR ゲーム要素を組み込みたい場合に有効な手段となります。

AR 機能は Unity 側で実装することになるため、ネイティブアプリ側では、特段 AR 機能を意識する必要はありません。
そのため、「ネイティブアプリチーム」と「Unity AR アプリチーム」のように開発を分担することも可能です。

ただし、Unity で開発したコンテンツ（export ファイル）を、ネイティブアプリに組み込むためには、各種設定やパフォーマンスの最適化など諸々の追加対応が必要となるため、開発側としてはかなり複雑になります。

<br>

## まとめ

ざっくりと全体の関係性を整理してみると、以下のようなイメージです。

![](https://storage.googleapis.com/zenn-user-upload/8029518c251f-20251027.png)

<br>

- **ARCore(Android) / ARKit(iOS)**
  モーショントラッキングや環境認識といった AR の基盤・土台となる機能を提供
- **SceneView(Android) / RealityKit(iOS)**
  上記の基盤上に 3D コンテンツを描画・レンダリングするための技術(ツール)
  - **SceneView**
    シンプルな 3D モデル表示に特化した OSS ライブラリ
    OSS ライブラリなので、今後コミュニティが衰退したり、他の有力ライブラリが出てきた場合のリスク等を考慮する必要あり
  - **RealityKit**
    物理演算やアニメーションなど高度な表現が可能な iOS 公式フレームワーク
- **Scene Viewer(Android) / AR Quick Look(iOS)**
  端末標準搭載の機能であり、フル機能の AR アプリを開発せずに手軽に AR 体験を提供したい場合に有効
  Web ブラウザからでも 3D モデルの AR 表示が可能なため、開発コストを抑えつつユーザーに AR 体験が提供可能
- **Unity**
  より高度な AR 表現やゲーム要素を含む開発では Unity が選択肢に
  クロスプラットフォームであり、AR Foundation を使用することで、ARCore/ARKit の両対応が可能
  ゲーム開発に特化した環境であるため、一般的なモバイルアプリ開発には必ずしも適しているわけではないことに注意
- **開発方針の選択**
  実現したい AR 体験の複雑さ、対応プラットフォーム、開発リソース、保守性などを総合的に考慮することが重要
  シンプルな 3D モデル表示であればネイティブ開発 + レンダリングライブラリで十分ですが、高度なゲーム要素や複雑な物理演算が必要な場合は Unity の採用も検討

<br>

## 参考資料

:::details 【その他参考リンク】

https://press.monaca.io/atsushi/20234
https://github.com/KhronosGroup/glTF-Sample-Assets/blob/main/Models/Models-showcase.md
https://tech.cygames.co.jp/archives/3597/
:::
