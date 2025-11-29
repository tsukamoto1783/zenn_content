---
title: "【Android】KotlinネイティブアプリからUnityアプリを呼び出してみた"
emoji: "🤝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Android, Kotlin, Unity]
published: true
publication_name: ncdc
---

Kotlin のネイティブアプリから Unity アプリを呼び出してみた内容を記載します。

※ 備忘録 & とりあえずの動作検証であったため、細かい部分の解説はありません。かなり "えいやっっ!" で動作させてみた内容です。

<br>

**【検証環境】**

- Unity 6.2（6000.2.9f1）
- Android Studio Narwhal Feature Drop | 2025.1.2
  - Kotlin:: 2.0.2
- Android, Unity Export: build.gradle
  - compileSdk: 36
  - minSdk: 31
  - targetSdk: 36

<br>

**【目次】**

1. Unity アプリの Export
2. Kotlin アプリの作成
3. Kotlin アプリと Unity アプリの連携

<br>

## Unity アプリの Export

Unity をネイティブアプリから呼び出すには、Unity 公式の[Unity as a Library](https://unity.com/ja/features/unity-as-a-library)機能を使用します。

https://unity.com/ja/features/unity-as-a-library

<br>

今回はシンプルに Unity の`AR Mobile`テンプレートでアプリを新規作成して、Kotlin アプリから呼び出す形とします。
![](https://storage.googleapis.com/zenn-user-upload/cd460b45dc70-20251028.png =400x)

テンプレアプリを作成できたら、以下の設定を確認します。

1.  `Edit` > `Project Settings` > `Player` > `Configuration`

    - Scriptiing Backend が `IL2CPP` になっているか
    - Target Architectures に `ARM64` がチェックされているか
    - ※ 新しめの Unity バージョンだとデフォルト設定されてそう。
      ![](https://storage.googleapis.com/zenn-user-upload/5659725854d0-20251128.png)

1.  `File` > `Build Profiles`

    - Platforms の Android を有効化する
    - Export Project にチェックを入れる
      ![](https://storage.googleapis.com/zenn-user-upload/878194b28ec9-20251128.png)

1.  Export ボタンを押下して実行
    - Export 先のフォルダを選択できるので、"NewFolder"で適当に Export 用のフォルダを作成し、"Choose"を押下。
      ![](https://storage.googleapis.com/zenn-user-upload/d3d1a2e6dbdd-20251128.png)
      ![](https://storage.googleapis.com/zenn-user-upload/6817e0eba82f-20251128.png)_↑Export 後の中身_

<br>

※ いろんな記事を参考に実施しました。他に必要そうな設定があれば適宜設定してください。
https://qiita.com/norihirosunada/items/f26e77c7555b0f38ac8b
https://qiita.com/cha84rakanal/items/4a1e4b242ec40e3e8121
https://zenn.dev/arsaga/articles/ede728a794a553

<br>

## Android プロジェクトの立ち上げ

こっちはシンプルに Android Studio で新規プロジェクトを立ち上げるだけです。

AndroidStudio で `New Project / Empty Activity` で作成したテンプレ新規状態のアプリで試していきます。
![](https://storage.googleapis.com/zenn-user-upload/32c327814acd-20251128.png)

後ほど差分を確認したいので、このまま一旦コミットしておきます。

<br>

## Kotlin アプリと Unity アプリの連携

ここからが本題です。
先ほど Export した Unity アプリの中身を、Kotlin アプリに組み込んでいきます。

ここの対応が記事を参考にしても中々うまくいかず & 最適解が分からず、結局最後は AI による力技で動かせるところまで持っていきました。

1. Export した Unity アプリの`unityLibrary`と`Shared`フォルダを、Kotlin アプリのホームディレクトリに配置
   - `Shared`も配置しないと build 時に怒られました。
2. `unityLibrary`をそのままコミット履歴に含めると、"3K+" もの差分になってしまうので、[こちらの記事](https://zenn.dev/arsaga/articles/ede728a794a553#android%E5%81%B4%E3%81%AE%E8%A8%AD%E5%AE%9A%E4%BD%9C%E6%A5%AD)を参考に、`unityLibrary`をコミット履歴から対象外に設定
   - unityLibrary/.gitignore
     - /build
     - /src
     - /symbols
     - /xrmanifest.androidlib/build
3. Kotlin アプリの画面レイアウトを Unity を呼び出せるように修正

   - 今回はシンプルに呼び出すだけの画面構成としました。既存の UI 部分の定義は全て削除

```diff kotlin : MainActivity.kt
+ package com.example.hoge
+ import android.os.Bundle
+ import com.unity3d.player.UnityPlayerGameActivity
+
+ class MainActivity : UnityPlayerGameActivity() {
+     override fun onCreate(savedInstanceState: Bundle?) {
+         super.onCreate(savedInstanceState)
+     }
+ }

- class MainActivity : ComponentActivity() {
-     override fun onCreate(savedInstanceState: Bundle?) {
-         super.onCreate(savedInstanceState)
-         enableEdgeToEdge()
-         setContent {
-             UnityImportTheme {
-                 Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
-                     Greeting(
-                         name = "Android",
-                         modifier = Modifier.padding(innerPadding)
-                     )
-                 }
-             }
-         }
-     }
- }-
- @Composable
- fun Greeting(name: String, modifier: Modifier = Modifier) {
-     Text(
-         text = "Hello $name!",
-         modifier = modifier
-     )
- }-
- @Preview(showBackground = true)
- @Composable
- fun GreetingPreview() {
-     UnityImportTheme {
-         Greeting("Android")
-     }
- }
```

4. 以降、記事を参考に色々設定したがうまくビルドできなかったので、ClaudeCode にぶん投げました。
   変更点などの詳細は ClaudeCode に別途まとめてもらったので、気になる方はご確認ください。

   ↓ 変更量としてはこれぐらいです。
   ![](https://storage.googleapis.com/zenn-user-upload/24205fd259f1-20251128.png =400x)

::::details 【変更点詳細: ClaudeCode 作】

# Unity Export を Android（Kotlin）プロジェクトに統合する手順

## 概要

本ドキュメントは、Unity から Export した Android プロジェクト（unityLibrary）を既存の Kotlin + Jetpack Compose プロジェクトに統合し、ビルドを成功させるまでの手順をまとめたものです。

### 前提条件

- Unity 6000.2.x（IL2CPP バックエンド使用）
- Android Studio
- Gradle 8.13 + Android Gradle Plugin 8.12.3
- Kotlin 2.0.21
- Compile SDK: 36

## 1. プロジェクト構成

```txt
AndroidCallUnity/
├── app/           # メインアプリケーションモジュール
├── unityLibrary/  # Unity から Export したライブラリ
│ ├── libs/        # unity-classes.jar, AAR ファイル
│ ├── src/main/
│ │ ├── Il2CppOutputProject/  # IL2CPP ソースコード
│ │ ├── jniLibs/              # ネイティブライブラリ (.so)
│ │ └── java/                 # Unity Player Java クラス
│ └── xrmanifest.androidlib/  # XR マニフェストモジュール
├── shared/                   # 共有 Gradle スクリプト
├── settings.gradle.kts
└── local.properties
```

## 2. 変更が必要なファイル

### 2.1 settings.gradle.kts

Unity モジュールを Gradle プロジェクトに追加します。

```diff kotlin
dependencyResolutionManagement {
+    repositoriesMode.set(RepositoriesMode.PREFER_SETTINGS)  // FAIL_ON_PROJECT_REPOSから変更
    repositories {
        google()
        mavenCentral()
+        flatDir {
+            dirs("unityLibrary/libs")  // AARファイル用のフラットディレクトリを追加
        }
    }
}

rootProject.name = "UnityCallSample"
+ include(":app", ":unityLibrary", ":unityLibrary:xrmanifest.androidlib")
```

**ポイント:**

- `repositoriesMode`を`PREFER_SETTINGS`に変更（unityLibrary 内の flatDir リポジトリを許可）
- `flatDir`で AAR ファイルのディレクトリを指定
- `unityLibrary`と`xrmanifest.androidlib`を include

### 2.2 gradle/libs.versions.toml

android-library プラグインを追加します。

```diff toml
[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
+ android-library = { id = "com.android.library", version.ref = "agp" }  # 追加
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
```

### 2.3 build.gradle.kts（ルート）

```diff kotlin
plugins {
    alias(libs.plugins.android.application) apply false
+     alias(libs.plugins.android.library) apply false  // 追加
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.compose) apply false
}
```

### 2.4 app/build.gradle.kts

unityLibrary 依存関係を追加します。

```diff kotlin
dependencies {
+     implementation(project(":unityLibrary"))  // 追加
    // その他の依存関係...
}
```

### 2.5 gradle.properties

Unity Streaming Assets 設定を追加します。

```diff properties
+ # Unity Streaming Assets
+ unityStreamingAssets=.unity3d
```

### 2.6 local.properties

Unity SDK/NDK パスを設定します。

```diff properties
+ sdk.dir=/Users/xxx/Library/Android/sdk
+ unity.androidSdkPath=/Users/xxx/Library/Android/sdk
+ unity.androidNdkPath=/Applications/Unity/Hub/Editor/6000.2.9f1/+ PlaybackEngines/AndroidPlayer/NDK
```

## 3. unityLibrary/build.gradle の修正

### 3.1 フラットディレクトリリポジトリの追加

```gradle
repositories {
    flatDir {
        dirs 'libs'
    }
}
```

### 3.2 依存関係を`api`に変更

app モジュールから Unity クラスにアクセスできるよう、`implementation`を`api`に変更します。

```gradle
dependencies {
    api fileTree(dir: 'libs', include: ['*.jar'])
    api(name: 'arcore_client', ext:'aar')
    api(name: 'ARPresto', ext:'aar')
    api(name: 'UnityARCore', ext:'aar')
    api(name: 'unityandroidpermissions', ext:'aar')
    api project(':unityLibrary:xrmanifest.androidlib')
    api 'androidx.appcompat:appcompat:1.6.1'
    api 'androidx.core:core:1.9.0'
    api 'androidx.games:games-activity:3.0.5'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
}
```

### 3.3 local.properties からプロパティを読み込む関数を追加

```gradle
def getLocalProperty(String key) {
    Properties local = new Properties()
    local.load(new FileInputStream("${rootDir}/local.properties"))
    return local.getProperty(key)
}
```

### 3.4 IL2CPP ビルド時のプロパティ参照を修正

`getProperty()`を`getLocalProperty()`に変更します。

```gradle
commandLineArgs.add("--tool-chain-path=" + getLocalProperty("unity.androidNdkPath"))
// ...
execCommand(workingDir, [command, *commandLineArgs], [
    "ANDROID_SDK_ROOT": getLocalProperty("unity.androidSdkPath"),
    "ANDROID_NDK_ROOT": getLocalProperty("unity.androidNdkPath"),
    "NDK_ROOT": getLocalProperty("unity.androidNdkPath"),
    "ANDROID_NDK_HOME": getLocalProperty("unity.androidNdkPath"),
])
```

### 3.5 IL2CPP ビルドエラー修正（ファイル存在チェック）

シンボルファイルが存在しない場合のエラーを回避するため、存在チェックを追加します。

```gradle
def fileToRemove = file("${workingDir}/src/main/jniLibs/${abi}/libil2cpp${extensionToRemove}")
if (fileToRemove.exists()) {
    delete "${workingDir}/src/main/jniLibs/${abi}/libil2cpp${extensionToRemove}"
}

def fileToMove = file("${workingDir}/src/main/jniLibs/${abi}/libil2cpp${extensionToKeep}")
if (fileToMove.exists()) {
    ant.move(file: "${workingDir}/src/main/jniLibs/${abi}/libil2cpp${extensionToKeep}",
             tofile: "${workingDir}/symbols/${abi}/libil2cpp.so")
}
```

## 4. xrmanifest.androidlib/AndroidManifest.xml の修正

`package`属性を削除します（namespace は build.gradle で指定）。

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
  <application>
    <activity android:name="com.unity3d.player.UnityPlayerGameActivity" />
  </application>
</manifest>
```

## 5. app/AndroidManifest.xml の Unity 対応設定

```xml
<activity
    android:name=".MainActivity"
    android:exported="true"
    android:label="@string/app_name"
    android:theme="@style/BaseUnityGameActivityTheme"
    android:configChanges="mcc|mnc|locale|touchscreen|keyboard|keyboardHidden|navigation|orientation|screenLayout|uiMode|screenSize|smallestScreenSize|fontScale|layoutDirection|density"
    android:hardwareAccelerated="false"
    android:launchMode="singleTask"
    android:screenOrientation="portrait">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
    <meta-data android:name="unityplayer.UnityActivity" android:value="true" />
    <meta-data android:name="android.app.lib_name" android:value="game" />
</activity>
```

**重要な設定:**

- `android:theme="@style/BaseUnityGameActivityTheme"` - Unity 専用テーマ
- `android:configChanges` - 画面回転等で Activity を再生成しない
- `android:hardwareAccelerated="false"` - Unity が OpenGL ES を直接使用するため
- `android:launchMode="singleTask"` - 単一インスタンスで管理
- `meta-data` - Unity プレイヤーとしての Activity 宣言

## 8. トラブルシューティング

### 8.1 AAR ファイルが見つからない

**エラー:** `Could not find :arcore_client:`

**解決策:**

- settings.gradle.kts に`flatDir`を追加
- `repositoriesMode`を`PREFER_SETTINGS`に変更

### 8.2 Unity クラスにアクセスできない

**エラー:** `Cannot access 'UnityPlayerGameActivity' which is a supertype...`

**解決策:**

- unityLibrary/build.gradle の依存関係を`implementation`から`api`に変更

### 8.3 unity.androidNdkPath が見つからない

**エラー:** `Could not get unknown property 'unity.androidNdkPath'`

**解決策:**

- `getLocalProperty()`関数を追加
- local.properties に`unity.androidNdkPath`を設定

### 8.4 libil2cpp.sym.so が見つからない

**エラー:** IL2CPP ビルド中にファイルが見つからない

**解決策:**

- ファイル存在チェックを追加（本ドキュメント 3.5 参照）

## 9. 生成物

ビルド成功後、以下のファイルが生成されます：

- **APK:** `app/build/outputs/apk/debug/app-debug.apk`

**含まれるネイティブライブラリ:**

- `libil2cpp.so` - Unity IL2CPP ランタイム
- `libunity.so` - Unity エンジンコア
- `libgame.so` - ゲームロジック
- `libmain.so` - メインエントリーポイント
- `lib_burst_generated.so` - Burst Compiler による最適化コード

::::

<br>

最終的に添付のように、Android アプリから Unity アプリが呼び出せることが確認できました。

![](https://storage.googleapis.com/zenn-user-upload/5328f54c12a0-20251128.gif)
_↑Android アプリアイコンを Tap して起動_

<br>

## 終わり

テンプレ状態の Unity アプリと Android アプリだとしても、連携するにはややこしい設定対応が必要になりました。

これが実際の開発しているプロジェクト同士だと、もっと複雑な設定や競合が発生し、build できるまでが大変そうだなぁ。という印象を受けました。

とはいえ、ネイティブアプリに Unity アプリを組み込めるのは場合によっては強力な手段となると思うので、次は少し作り込んだアプリ同士で連携を試してみたいと思いました。

<br>

### 備考

※ 実案件で`Unity as a Library`を使用するのかどうか、みたいな観点は以下の記事も参考にしてみてください。

https://zenn.dev/mirrativ_blog/articles/f1e9dc9e55d43c
https://tech.cygames.co.jp/archives/3597/

<br>

※ ネイティブ完結の AR 機能等についてはこちらもよければご参照ください。
https://zenn.dev/ncdc/articles/android_ar_sample
