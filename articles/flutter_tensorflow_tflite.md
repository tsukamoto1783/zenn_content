---
title: "【Flutter】学習モデルを組み込んで推論実行する"
emoji: "🦾"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter,Dart,TensorFlow, TensorFlowLite, tflite]
published: true
publication_name: ncdc
---

Flutterで学習モデルを用いた推論実行してみたい。
ということで、Flutterで学習モデルを推論実行する方法を調べてみました。

&nbsp;
Flutterで学習モデル扱うために、今回は[tflite_flutter](https://pub.dev/packages/tflite_flutter)を使用して実行していきます。
機械学習でメジャーなTensorFlowが開発の大本（さらにその大本はGoogle）なので、かなり信頼できるライブラリかと思います。

ただ、このライブラリはここ2年近く開発が止まっていました。
![](https://storage.googleapis.com/zenn-user-upload/110ad773842b-20230527.png =400x)
*※ 2023/5/27 時点*

そのため、tflite_flutterを扱っている技術記事も少なく、Flutterのアップデートにライブラリがついていけていない状況でした。

また、大本であるTensorFlowや、FirebaseのML Kitなどの公式ドキュメントでも、Flutterに関する記述は現状ほとんどありません。ほとんどがAndroidやiOSの記述です。

そんな中、最近ライブラリの更新が再開されアップデートが活発になっており、今後さらに発展していきそうなので早速試してみたいと思います。

&nbsp;
:::message
記事作成時のライブラリバージョンは**0.10.0**です。
:::

&nbsp;
# サンプルアプリを動かして確認してみる
まずはライブラリが公開しているサンプルアプリを動かして、学習モデルを使用するとどういったことができるか確認してみます。

現在公開されているサンプルは以下4つです。
- image_classification_mobilenet
- style_transfer
- super_resolution_esrgan
- text_classification

&nbsp;
### 【image_classification_mobilenet】
王道な画像推論。画像から何が写っているかを判定します。
実機なら写真撮って推論することも可能です。
![](https://storage.googleapis.com/zenn-user-upload/5b30b7f422a2-20230527.gif =250x)

&nbsp;
### 【style_transfer】
選択した画像のスタイルを変換します。
![](https://storage.googleapis.com/zenn-user-upload/7fe3db8154e9-20230527.gif =250x)

&nbsp;
### 【super_resolution_esrgan】
画像の解像度を変更します。
サンプルでは50×50の画像を200×200に拡大しています。
![](https://storage.googleapis.com/zenn-user-upload/b252d4cbb7da-20230527.gif =250x)

&nbsp;
### 【text_classification】
入力した文章を分類します。
※何を基準に分類している、、？
![](https://storage.googleapis.com/zenn-user-upload/8597a3dba2a5-20230527.gif =250x)
*[Text Classification using TensorFlow Lite Plugin for Flutter](https://medium.com/@am15hg/text-classification-using-tensorflow-lite-plugin-for-flutter-3b92f6655982)より引用*

&nbsp;
# 自前の学習モデルに差し替えて使用してみる
サンプルアプリでどういったことができるか確認できました。
次は自前の学習モデルを使用して推論を実行してみます。

上記サンプルで確認した「image_classification_mobilenet」を基に、今回は画像推論の入門としてメジャーな**MNIST**を使用して推論実行するアプリを作成してみます。


&nbsp;
変更後のMNISTを実行するアプリはこんな感じ。
手描き数字の画像から、どの数字が記載されているかを判定できていることが確認できます。
![](https://storage.googleapis.com/zenn-user-upload/a7c5ea437354-20230528.gif =250x)


&nbsp;
※MNISTのモデルの作成については、以下の記事を参考にしてみてください。
[「~コピペでOK~ 実際に動かして体験する機械学習入門」](https://zenn.dev/tsukatsuka1783/articles/introduction_machine_learning)



&nbsp;
# モデル差し替え時に詰まったポイント
コードの変更量自体は大したことないですが、差し替え時に詰まった点を記載します。

### 1.【インタプリタの初期化時のエラー】
差し替えた.tflite形式のモデルを用いてインタプリタを初期化する際に、
"TensorFlowのオプションがサポートされてないやらなんやら" でエラーが発生。

```dart
interpreter = await Interpreter.fromAsset(modelName, options: _interpreterOptions);
```

```txt
ERROR: 
Select TensorFlow op(s), included in the given model, is(are) not supported by this interpreter. 
Make sure you apply/link the Flex delegate before inference. 
For the Android, it can be resolved by adding "org.tensorflow:tensorflow-lite-select-tf-ops" dependency. 
See instructions: https://www.tensorflow.org/lite/guide/ops_select
```

&nbsp;
.pbファイルから.tfliteファイルに変換する際に、エラーに記載があるURL内のドキュメントの通りに変換すると解決しました。
**※.pb形式以外のモデルから.tflite形式に変換する際は要注意。**

```python: pythonでの変換コード
converter = tf.lite.TFLiteConverter.from_saved_model(input_dir)
converter.target_spec.supported_ops = [
  tf.lite.OpsSet.TFLITE_BUILTINS, # enable TensorFlow Lite ops.
  tf.lite.OpsSet.SELECT_TF_OPS # enable TensorFlow ops.
]
tflite_model = converter.convert()
open("converted_model.tflite", "wb").write(tflite_model)
```

&nbsp;
### 2.【インタプリタのデリゲートオプションによるエラー】
モデルをセットして推論を実行すると、
"モデルにGPU delegateでサポートされていない演算が含まれてるから推論できないとかなんとか" でエラーが発生。

```dart
  // Load model
  Future<void> loadModel() async {
    final options = InterpreterOptions();

    // 省略

    // Use Metal Delegate
    if (Platform.isIOS) {
      options.addDelegate(GpuDelegate());
    }
    
    // Load model from assets
    interpreter = await Interpreter.fromAsset(modelPath, options: options);

    // 省略
  }

  // Process picked image
  Future<void> processImage() async {
    // 省略

    // Run model inference
    runInference(imageMatrix);
}
```

```txt
Following operations are not supported by GPU delegate:
MUL: MUL requires one tensor that not less than second in all dimensions.
25 operations will run on the GPU, and the remaining 40 operations will run on the CPU.
TfLiteMetalDelegate Prepare: Failed to allocate id<MTLBuffer>
Node number 65 (TfLiteMetalDelegate) failed to prepare.
Restored original execution plan after delegate application failure.
[VERBOSE-2:dart_vm_initializer.cc(41)] Unhandled Exception: Bad state: failed precondition
```

&nbsp;
モデル学習時の設定の問題でもあるかと思いますが、ここではGpuDelegateのオプション設定を無効にすることで解決。

※GpuDelegateのオプション設定を無効にすると、推論処理がCPUで実行されるようになるので、推論速度等が低下する恐れがあること注意。
※GpuDelegateのオプション設定を無効にすることが非推奨なのかどうか等は現状ドキュメント等に記載がない。この辺のオプション設定について今後明記されるかもしれないので要確認。

```diff dart
  // Load model
  Future<void> loadModel() async {
    final options = InterpreterOptions();

    // 省略

    // Use Metal Delegate
    if (Platform.isIOS) {
-     options.addDelegate(GpuDelegate());
+     // options.addDelegate(GpuDelegate());
    }

    // Load model from assets
    interpreter = await Interpreter.fromAsset(modelPath, options: options);

    // 省略
  }

  // Process picked image
  Future<void> processImage() async {
    // 省略

    // Run model inference
    runInference(imageMatrix);
}
```

&nbsp;
**【補足：デリゲートとは】**
デリゲートをざっくりまとめると、
「TensorFlow Liteが提供するツールで、スマートフォンなどのデバイス上で機械学習モデルの計算を高速化したり電力を節約したりするための、特殊なハードウェア（GPUやDSPなど）を活用する機能のこと。」

なので、デリゲート設定を有効にしない場合は、CPU上でモデルを実行することになる。
参照:[「TensorFlow Lite Delegates」](https://www.tensorflow.org/lite/performance/delegates)

※特殊なハードウェア（GPUやDSPなど）：アクセラレータ




&nbsp;
### 3.【モデルが求めるテンソルの形状に変換】
モデルが求めるテンソル情報を基に、画像を変換する必要があります。
サンプルアプリだと、モデルが求めているinput画像の形式は、
「1枚の画像を受けつけ、224x224の3チャンネル（RGB）の画像」ということが分かります。

```dart
// Load model from assets
interpreter = await Interpreter.fromAsset(modelPath, options: options);

inputTensor = interpreter.getInputTensors().first;
print(inputTensor);

outputTensor = interpreter.getOutputTensors().first;
print(outputTensor);
```

```
flutter: Tensor{_tensor: Pointer: address=0x1299bac80, name: input, type: uint8, shape: [1, 224, 224, 3], data: 150528}

flutter: Tensor{_tensor: Pointer: address=0x1299bac10, name: MobilenetV1/Predictions/Reshape_1, type: uint8, shape: [1, 1001], data: 1001}
```

&nbsp;
&nbsp;
MNISTのモデルに差し替えると、モデルが求めているinput画像の形式は、
「1枚の画像を受けつけ、28x28のグレースケール画像」ということが分かります。

```
flutter: Tensor{_tensor: Pointer: address=0x12f8a0c00, name: serving_default_input_1:0, type: float32, shape: [1, 28, 28], data: 3136}

flutter: Tensor{_tensor: Pointer: address=0x12f8a14c0, name: StatefulPartitionedCall:0, type: float32, shape: [1, 10], data: 40}
```

&nbsp;
上記の変更点を踏まえて、画像変換処理部分を修正します。


::::details コード変更箇所
※変更のあるコード箇所だけ記載します。変更ない部分は記載省略。
※今回はtypeがuint8ではなくfloat32になっていることにも注意。
※読み込むlabels.txtもMNISTに合わして変更してます。

```diff dart
class _HomeState extends State<Home> {
-  static const modelPath = 'assets/mobilenet/mobilenet_v1_1.0_224_quant.tflite';
+  static const modelPath = 'assets/mnist/mnist.tflite';

-  static const labelsPath = 'assets/mobilenet/labels.txt';
+  static const labelsPath = 'assets/mnist/labels.txt';

-  Map<String, int>? classification;
+  Map<String, double>? classification;


  // Process picked image
  Future<void> processImage() async {
    if (imagePath != null) {

      // Resize image for model input
      final imageInput = img.copyResize(
        image!,
-        width: 224,
-        height: 224,
+        width: 28,
+        height: 28,
      );

      // Get image matrix representation
      final imageMatrix = List.generate(
        imageInput.height,
        (y) => List.generate(
          imageInput.width,
          (x) {
            final pixel = imageInput.getPixel(x, y);
-            return [pixel.r, pixel.g, pixel.b];
+            return (pixel.r + pixel.g + pixel.b) / 3;
          },
        ),
      );

      // Run model inference
      runInference(imageMatrix);
    }
  }

  // Run inference
  Future<void> runInference(
-    List<List<List<num>>> imageMatrix,
+    List<List<num>> imageMatrix,
  ) async {
    // Set tensor input
    final input = [imageMatrix];
    // Set tensor output
-    final output = [List<int>.filled(1001, 0)];
+    final output = [List<double>.filled(10, 0)];

    // Run inference
    interpreter.run(input, output);

    // Get first output tensor
    final result = output.first;
    print(result);

    // Set classification map {label: points}
-    classification = <String, int>{};
+    classification = <String, double>{};

  }
}
```
```txt: labels.txt
0
1
2
3
4
5
6
7
8
9
```
::::

&nbsp;
# ライブラリのアップデートに伴う変更点
以前のバージョン（[tflite_flutter(0.9.0)](https://pub.dev/packages/tflite_flutter/versions/0.9.0#important-initial-setup--add-dynamic-libraries-to-your-app)）からの変更点をざっとまとめると、
- 「ライブラリの導入が簡単に」
以前はライブラリ導入する際に「.sh」を走らせたりしないといけなかったが、それらの導入処理が無くなりました。
- 「FlutterやTensorFlowのバージョンアップに追従」
- 「画像の前処理などのをサポートするヘルパーライブラリが非推奨に」
以前のヘルパーライブラリを代替するための新しい開発が進行中で、2023年8月末までに広くサポートされる予定とのことです。
- etc.

&nbsp;
# おわり
冒頭にも記載しましたが、このライブラリは記事記載時点ではまだまだ開発途中です。

issueのコメントで、このライブラリの更新を望む声がたくさん上がっていて、開発者もそれに応えてくれているので、これからの更新が非常に楽しみなライブラリです。
（開発が2年ぐらい止まっている間も、ForkしてFlutterのバージョンアップに対応してくれている人がいたり、長い間支えてくれていてありがたい限りです。。）

[tflite_flutter](https://pub.dev/packages/tflite_flutter)のバージョンアップに付随して、TensorFlowや、FirebaseのML KitのドキュメントにもFlutterの記載が充実することを期待しています。

