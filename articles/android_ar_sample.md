---
title: "【Android/Kotlin】AR機能を色々なパターンで試してみる"
emoji: "🌎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Android, ARCore, AR, SceneView, SceneViewer]
published: true
publication_name: ncdc
---

## はじめに

この記事では Android 端末での AR 機能について、色々なパターンで動作検証してみた内容を記載しています。
主な検証パターンとしては以下です。

- SceneView：ARSceneView
- SceneView：SceneView
- Scene Viewer
- model-viewer
- Kotlin アプリから Unity 呼び出し

<br>

※ AR 機能についての大まかな全体像については、以下の記事にまとめていますので、そちらをご参照ください。

https://zenn.dev/ncdc/articles/ar_composition

### 動作環境・備考

- Android Studio Narwhal Feature Drop | 2025.1.2
- Kotlin + Jetpack Compose
- 検証端末: Pixel 8a (Android 14)
- サンプルコードでは View 部分の実装のみ記載です。
- 動作確認をしたいだけの実装であるため、コーディングのお作法的なところは御愛嬌ください。
- 動作検証用のアプリは、AndroidStudio で `New Project / Empty Activity` で作成したアプリをベースにしています。
  ![](https://storage.googleapis.com/zenn-user-upload/426ca89d9bec-20251028.png =500x)
- 動作確認に使用する 3D モデルは、以下のリポジトリのものを使用しています。
  https://github.com/KhronosGroup/glTF-Sample-Assets/tree/main/Models

<br>

## SceneView：ARSceneView

まずはアプリ内で AR 機能を完結させるパターンを試してみます。

3D コンテンツのレンダリングには、[SceneView ライブラリ](https://github.com/SceneView/sceneview-android)を使用します。

https://github.com/SceneView/sceneview-android

SceneView ライブラリは主に以下のコンポーネントを提供してくれます。
ここでは AR 機能を試したいので、**ARSceneView** コンポーネントを使用します。

> - **Sceneview**: 3D rendering capabilities using Filament
> - **ARSceneview**: Augmented Reality capabilities using Filament and ARCore

<br>

ARSceneView を使う際は、ライブラリ内部で ARCore を参照しているため、特段アプリ側で ARCore の import 等は不要となります。
ただ、内部的には ARCore を使用しているため、ARCore で必要となる権限設定はアプリ側でも行う必要があります。

https://developers.google.com/ar/develop/java/enable-arcore?hl=ja

```diff:toml
# libs.versions.toml
# build.gradleへの追記は省略
[versions]
+ lifecycleViewmodelCompose = "2.6.1"
+ arsceneview = "2.3.0"

[libraries]
+ arsceneview = { group = "io.github.sceneview", name = "arsceneview", version.ref = "arsceneview" }
```

```diff:xml
# AndroidManifest.xml
+ <uses-permission android:name="android.permission.CAMERA" />
+ <uses-feature android:name="android.hardware.camera.ar" android:required="true" />
+ <meta-data android:name="com.google.ar.core" android:value="required" tools:replace="android:value" />
```

<br>

続いて、[README の Basic Usage](https://github.com/SceneView/sceneview-android?tab=readme-ov-file#basic-usage) を参考に、アプリの View 箇所に、ARSceneView を使った AR 表示のコードを実装します。
3D モデルファイルについては、`assets/models/` に配置しておきます。

```kotlin
val engine = rememberEngine()
val modelLoader = rememberModelLoader(engine)

// 3Dモデルノードを一度だけ作成して保持
val modelNode = remember {
    ModelNode(
        // assetsフォルダから3DモデルファイルをロードしてModelInstanceを作成
        modelInstance = modelLoader.createModelInstance(
            assetFileLocation = "models/ABeautifulGame.glb"
        ),
        // モデルのスケール設定
        scaleToUnits = 0.5f
    ).apply {
        // モデルの初期位置を設定
        // x: 0f (左右中央)
        // y: -0.5f (中央からやや下)
        // z: -1f (カメラから1メートル前方)
        position = io.github.sceneview.math.Position(x = 0f, y = -0.5f, z = -1f)
    }
}

Box(modifier = Modifier.fillMaxSize()) {
    /**
     * AR設定と3Dモデルの表示
     *
     * - engine: Filamentレンダリングエンジン
     * - modelLoader: 3Dモデル(GLB/GLTF)をロードするためのローダー
     * - childNodes: シーンに配置する3Dモデルノードのリスト
     * - sessionConfiguration: ARCore セッションの設定
     */
    ARScene(
        modifier = Modifier.fillMaxSize(),
        engine = engine,
        modelLoader = modelLoader,
        childNodes = rememberNodes {
            add(modelNode)
        },
        sessionConfiguration = { session, config ->
            /**
             * デプスモード（深度認識）の設定
             *
             * デバイスがサポートしている場合は自動デプス検出を有効化
             * これにより3Dオブジェクトが現実世界の物体の前後関係を正しく認識可能
             */
            config.depthMode = when (session.isDepthModeSupported(Config.DepthMode.AUTOMATIC)) {
                true -> Config.DepthMode.AUTOMATIC
                else -> Config.DepthMode.DISABLED
            }
            /**
             * 光推定モードの設定
             *
             * 現実世界の照明条件を3Dモデルに反映し、よりリアルな描画を実現
             */
            config.lightEstimationMode = Config.LightEstimationMode.ENVIRONMENTAL_HDR
        }
    )
}
```

簡易設定ではありますが、これぐらいのコード量で最低限の AR 表示が実装できました。
画像ではわかりづらいですが、ARCore が提供する**環境認識**や**モーショントラッキング**も機能していることが確認できました。
→ 照明が暗い空間で AR を起動すると、3D モデルの明るさも連動して暗くなることが確認できました。

![](https://storage.googleapis.com/zenn-user-upload/8606d0fee1d8-20251028.png =200x)

<br>

## SceneView: SceneView

続いて、SceneView ライブラリのもう一つの主機能である **SceneView** コンポーネントを使用してみたいと思います。

SceneView コンポーネントは、AR 機能を使用しない純粋な 3D モデルのレンダリング機能を提供してくれます。
なので、アプリ側としても ARCore の導入は不要ですし、ライブラリ内部でも ARCore は参照されないため、ARCore で必要となる権限等の設定も不要です。

先ほど同様、[README の Basic Usage](https://github.com/SceneView/sceneview-android?tab=readme-ov-file#basic-usage) を参考に、アプリの View 箇所に、SceneView を使って 3D モデルをレンダリングするコードを実装します。

```kotlin
val engine = rememberEngine()
val modelLoader = rememberModelLoader(engine)

// 3Dモデルノードを一度だけ作成して保持
val modelNode = remember {
    ModelNode(
        // assetsフォルダから3DモデルファイルをロードしてModelInstanceを作成
        modelInstance = modelLoader.createModelInstance(
            assetFileLocation = "models/ABeautifulGame.glb"
        ),
        // モデルのスケール設定
        scaleToUnits = 0.5f
    ).apply {
        // モデルの初期位置を設定
        // x: 0f (左右中央)
        // y: -0.5f (中央からやや下)
        // z: -2f (カメラから2メートル前方)
        position = Position(x = 0f, y = -0.5f, z = -2f)
    }
}

Box(modifier = Modifier.fillMaxSize()) {
    /**
     * 3Dシーンの設定
     *
     * - engine: Filamentレンダリングエンジン
     * - modelLoader: 3Dモデル(GLB/GLTF)をロードするためのローダー
     * - childNodes: シーンに配置する3Dモデルノードのリスト
     */
    Scene(
        modifier = Modifier.fillMaxSize(),
        engine = engine,
        modelLoader = modelLoader,
        childNodes = rememberNodes {
            add(modelNode)
        }
    )
}
```

同じく簡易設定ですが、ライブラリを使用すると、比較的容易に 3D モデルのレンダリングが実装できました。

![](https://storage.googleapis.com/zenn-user-upload/9c4381186f6c-20251028.png =200x)

SceneView の利用の場合は、ARCore の特徴である**環境認識**や**モーショントラッキング**等は一切行われていないため、AR 機能ではなく、**単なる 3D モデルの描画**となります。

<br>

## Scene Viewer

今までの実装では、アプリ独自(アプリ内で完結)の機能として、AR 機能や 3D モデルレンダリング機能を実装するパターンでした。

次は Android 端末に標準搭載されている **Scene Viewer** 機能を使用して、AR プレビューを表示してみたいと思います。

https://developers.google.com/ar/develop/scene-viewer?hl=ja

<br>

Scene Viewer は、Android 端末に標準搭載されている"Google アプリ"上で動作する AR プレビュー機能です。

アプリ側で AR 機能を一から開発するコストをかけずに、手軽に AR コンテンツを提供したい場合に有効です。

![](https://storage.googleapis.com/zenn-user-upload/68fde4ab471a-20251027.gif =200x)
_Chrome から AR プレビューを起動_

<br>

Scene Viewer はブラウザからでも呼び出せますが、まずはブラウザからではなく、アプリから呼び出すパターンを試してみます。

```kotlin
@Composable
fun SceneViewerScreen() {
    // サンプルの3DモデルURL（glTF形式）
    val modelUrl = "https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Models/master/2.0/Avocado/glTF-Binary/Avocado.glb"

    // Scene Viewer の Intent URI を構築
    val sceneViewerUri = "https://arvr.google.com/scene-viewer/1.0".toUri()
    .buildUpon()
    .appendQueryParameter("file", modelUrl)
    .appendQueryParameter("mode", "ar_only")
    .build()

    // Scene Viewer を起動
    val intent = Intent(Intent.ACTION_VIEW).apply {
        data = sceneViewerUri
        setPackage("com.google.android.googlequicksearchbox")
    }

    val context = LocalContext.current
    context.startActivity(intent)
}
```

AR プレビューが、アプリからでも呼び出せることが確認できました。

![](https://storage.googleapis.com/zenn-user-upload/6a23d2448540-20251028.gif =200x)
_※ プレビュー画面で描画できるまでに時間がかかるため一部カットしています_

<br>

## model-viewer

Web ブラウザから AR プレビューを起動してみるパターンもせっかくなので試してみます。

ブラウザから AR プレビューを起動する場合、Google が提供している **model-viewer** ライブラリを使用すると良さそうです。

構成としては、アプリ内に WebView を配置し、その中で <model-viewer> を使用した HTML を適当にベタ書きして試してみました。

※ 実際にはモバイルアプリで model-viewer を使用するケースはあまりないかもしれませんが、一応動作確認のために試してみました。

https://modelviewer.dev/

<br>

上記のサメの Gif 動画と同様に、一連のプレビュー機能が機能することを確認できました。（画面右下の AR アイコン?を Tap すると AR プレビューも正常に起動しました。）
※ 注意点としては、AR プレビュー（Scene Viewer）は Web 機能であるため、通信環境が必要となる点は一つポイントです。

![](https://storage.googleapis.com/zenn-user-upload/2da8a48d0e59-20251028.gif =200x)

<br>

## Kotlin アプリから Unity 呼び出し

最後に、Kotlin で作成した Android アプリから Unity を呼び出すパターンを試してみます。

Unity 側で、**Unity as a Library** という機能が提供されており、これを使用すると、ネイティブ実装の Android アプリから Unity の機能を呼び出すことが可能となるみたいです。

https://unity.com/ja/features/unity-as-a-library

Unity についてはあまり詳しくないので、先人の知恵(記事)を参考にしつつ、とりあえずの動作確認はできました。

→ ここだけで結構の量になったので、別記事に切り出しました。

https://zenn.dev/ncdc/articles/android_call_unity

<br>

## まとめ

Android アプリでの超基本的な AR 機能について、色々と動作検証してみた内容を記載しました。

ライブラリを用いると、簡単な AR 機能であれば比較的容易に実装できることがわかりました。

ただ、ワールド座標やオブジェクト座標などの各種座標のカスタムや、AR のマーカー対応だったり、諸々の高度な実装を実現したい場合は、ARCore や AR 技術の知識が更に必要となってくるなと感じました。
必要に応じて ARCore や AR 自体の知識も深めていきたいところです。

## 参考

::: details 【参考リンク】
https://developers.google.com/ar/develop?hl=ja
https://zenn.dev/noby111/books/3f370e126df73b/viewer/d6cc56
https://1planet.co.jp/tech-blog/A1Tx78T4
https://www.techpit.jp/courses/80/curriculums/83/sections/628/parts/2183
https://press.monaca.io/atsushi/20234
https://press.monaca.io/atsushi/30565
https://crexgroup.com/ja/xr/ar/what-is-arcore/
https://engineers.safie.link/entry/how_to_use_sceneview_android
https://qiita.com/kodev13/items/16f2065513e19aac16f0
:::
