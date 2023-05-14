---
title: "~コピペでOK~ 実際に動かして体験する機械学習入門"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Python, MNIST, TensorFlow, TensorFlowLite, 機械学習]
published: true
---

# この記事の目的
「機械学習とは結局なんぞや、何ができるんや。」
「こまけぇことはいいからとりあえずサンプルを動かして挙動確認してえ。」

といった、**機械学習という言葉は知っているが、実際の動作イメージがついていない人** が実際に動作確認して体験してみよう。
といった記事です。

今回は**MNIST**という手書き数字の画像データを使って、数字を推論するプロブラムを作成します。

「機械学習ってこういったことができるんやなぁ。」となんとなく掴んで頂けたら儲けもんです。

※ とりあえず機械学習を体験してみることが目的なので、技術的に詳しいことは書いてない＆参考になるようなコードは書いてません。あしからず。

# 「機械学習」とは
ざっくり言うと、**コンピューター自らがデータから学習する仕組み。また、そのデータを元に予測等を行うこと。**

（備考）
「人工知能（AI）」を実現するための一つの手法が「機械学習」であり、「ディープラーニング」はその機械学習の手法の一つです。（機械学習の派生、強化版みたいなイメージです。）

# MNISTとは
MNISTは、機械学習の手法の一つである「ニューラルネットワーク」を使用した代表的な**手書き数字の画像データセット**です。

機械学習のチュートリアルなどでよく使用される有名なデータセットで、手書き数字の画像を読み込ませて数字を当てる。といった一度は見たことあるあれです。
![](https://storage.googleapis.com/zenn-user-upload/6ced053f7c01-20230512.png =150x)

※「ニューラルネットワーク」にも「畳み込みニューラルネットワーク（CNN）」とか種類があるが、ややこしいのでここでは割愛。

# MNISTの学習モデルを作成してみる
MNISTのデータセットを取得し、学習するコードを以下に記載します。
コピペしたらPython実行するだけで学習モデルの作成完了です。

::::details 学習モデル作成コード
```python
# 必要なライブラリは各自installしてください。
# 以下のサンプルコードでは、作成される学習モデルはTensorFlowLite形式に変換してます。後にモバイル等に組み込むことを想定していたため。

import tensorflow as tf
from tensorflow import keras
import numpy as np
import matplotlib.pyplot as plt
import random

print(tf.__version__)

# 2つの入力パラメータが一致するかどうかに応じて、'red'/'black'を返すヘルパー関数
def get_label_color(val1, val2):
  if val1 == val2:
    return 'black'
  else:
    return 'red'

# "テスト"データセットを使用してTF Liteモデルを評価するためのヘルパー関数。
def evaluate_tflite_model(tflite_model):
  # モデルを使用してTFLiteインタープリタを初期化する。
  interpreter = tf.lite.Interpreter(model_content=tflite_model)
  interpreter.allocate_tensors()
  input_tensor_index = interpreter.get_input_details()[0]["index"]
  output = interpreter.tensor(interpreter.get_output_details()[0]["index"])

  # "テスト"データセットのすべての画像で予測を実行する。
  prediction_digits = []
  for test_image in test_images:
    # 前処理：バッチ次元を追加し、モデルの入力データ形式に合わせてfloat32に変換する。
    test_image = np.expand_dims(test_image, axis=0).astype(np.float32)
    interpreter.set_tensor(input_tensor_index, test_image)

    # 推論を実行する。
    interpreter.invoke()

    # 後処理：バッチ次元を削除し、
    # 最も確率が高い値を見つけるために、最大の確率を持つ数字を取得する。
    digit = np.argmax(output()[0])
    prediction_digits.append(digit)

  # 予測結果を正解ラベルと比較して正確性を計算する。
  accurate_count = 0
  for index in range(len(prediction_digits)):
    if prediction_digits[index] == test_labels[index]:
      accurate_count += 1
  accuracy = accurate_count * 1.0 / len(prediction_digits)

  return accuracy
# =============================================

# kerasのAPIを使用してMNISTデータセットをダウンロード。「トレーニング」データセットと「テスト」データセットに分割する。
mnist = keras.datasets.mnist
(train_images, train_labels), (test_images, test_labels) = mnist.load_data()

# 入力画像を正規化して、各ピクセルの値を0から1の間になるようにする。
train_images = train_images / 255.0
test_images = test_images / 255.0

# トレーニングデータセットの最初の25枚の画像を表示する。確認用に。
plt.figure(figsize=(10,10))
for i in range(25):
  plt.subplot(5,5,i+1)
  plt.xticks([])
  plt.yticks([])
  plt.grid(False)
  plt.imshow(train_images[i], cmap=plt.cm.gray)
  plt.xlabel(train_labels[i])
plt.show()


# モデルのアーキテクチャを定義する
model = keras.Sequential([
  keras.layers.InputLayer(input_shape=(28, 28)),
  keras.layers.Reshape(target_shape=(28, 28, 1)),
  keras.layers.Conv2D(filters=32, kernel_size=(3, 3), activation=tf.nn.relu),
  keras.layers.Conv2D(filters=64, kernel_size=(3, 3), activation=tf.nn.relu),
  keras.layers.MaxPooling2D(pool_size=(2, 2)),
  keras.layers.Dropout(0.25),
  keras.layers.Flatten(),
  keras.layers.Dense(10)
])

# モデルの訓練方法を定義する
model.compile(optimizer='adam',
              loss=tf.keras.losses.SparseCategoricalCrossentropy(from_logits=True),
              metrics=['accuracy'])

# 数字の分類モデルを訓練する
model.fit(train_images, train_labels, epochs=5)

model.summary()

# テストデータセットのすべての画像を使用してモデルを評価する。
test_loss, test_acc = model.evaluate(test_images, test_labels)
print('Test accuracy:', test_acc)

# テストデータセットの数字画像のラベルを予測する。
predictions = model.predict(test_images)

# モデルは、入力画像が0から9の数字である確率を表す10個の浮動小数点数を出力するため、
# 最も確率が高い値を見つけるために、予測値の配列から最大値のインデックスを取得する。
prediction_digits = np.argmax(predictions, axis=1)

# 100枚のランダムなテスト画像と予測されたラベルをプロットする。
# 予測結果が「テスト」データセットの提供されたラベルと異なる場合は、赤色で強調表示する。
plt.figure(figsize=(18, 18))
for i in range(100):
  ax = plt.subplot(10, 10, i+1)
  plt.xticks([])
  plt.yticks([])
  plt.grid(False)
  image_index = random.randint(0, len(prediction_digits))
  plt.imshow(test_images[image_index], cmap=plt.cm.gray)
  ax.xaxis.label.set_color(get_label_color(prediction_digits[image_index],\
                                           test_labels[image_index]))
  plt.xlabel('Predicted: %d' % prediction_digits[image_index])
plt.show()

# KerasモデルをTF Lite形式に変換する。
converter = tf.lite.TFLiteConverter.from_keras_model(model)
tflite_float_model = converter.convert()

# モデルのサイズをKB単位で表示する。
float_model_size = len(tflite_float_model) / 1024
print('Float model size = %dKBs.' % float_model_size)

# 量子化を使用してモデルをTF Liteに再変換する。
converter.optimizations = [tf.lite.Optimize.DEFAULT]
tflite_quantized_model = converter.convert()

# モデルのサイズをKB単位で表示する。
quantized_model_size = len(tflite_quantized_model) / 1024
print('Quantized model size = %dKBs,' % quantized_model_size)
print('which is about %d%% of the float model size.'\
      % (quantized_model_size * 100 / float_model_size))

# TF Lite floatモデルを評価する。オリジナルのTF（Keras）モデルと同じ正確性が得られる。
float_accuracy = evaluate_tflite_model(tflite_float_model)
print('Float model accuracy = %.4f' % float_accuracy)

# TF Lite quantizedモデルを評価する。
# 量子化モデルの正確性がオリジナルのfloatモデルよりも高い場合がある
quantized_accuracy = evaluate_tflite_model(tflite_quantized_model)
print('Quantized model accuracy = %.4f' % quantized_accuracy)
print('Accuracy drop = %.4f' % (float_accuracy - quantized_accuracy))


# 量子化されたモデルをDownloadsディレクトリに保存する
f = open('mnist.tflite', "wb")
f.write(tflite_quantized_model)
f.close()
```
::::

# 作成した学習モデルを推論してみる 
上記で作成した学習モデルを使用して、推論を実行するコードを以下に記載します。
::::details 推論コード
```python
import numpy as np
import tensorflow as tf
from PIL import Image

# MNISTモデルのパスと画像のパスを指定する
model_path = '<学習モデルのパス>'
image_path = '<入力する画像のパス>'

def load_image(image_path):
    # 画像を開き、グレースケールに変換する
    image = Image.open(image_path).convert('L')
    # 画像のサイズを28x28に変更し、numpy配列に変換する
    image = np.array(image.resize((28, 28)), dtype=np.float32)
    # 画像を反転させる
    image = 255.0 - image
    # ピクセル値を0から1の範囲に正規化する
    image /= 255.0

    print("Image type:", type(image))
    return image


def run_inference(tflite_model_path, image_path):
    # TF Liteモデルの読み込みと初期化処理
    interpreter = tf.lite.Interpreter(model_path=tflite_model_path)
    interpreter.allocate_tensors()

    # 入力と出力の詳細を取得する
    input_details = interpreter.get_input_details()
    output_details = interpreter.get_output_details()
    print(input_details)
    print(output_details)

    # 画像を読み込む
    image = load_image(image_path)
    print(image)

    # 入力テンソルの形状に画像を整形する
    target_shape = input_details[0]['shape']
    image = np.reshape(image, target_shape)

    # 入力テンソルに画像を設定する
    interpreter.set_tensor(input_details[0]['index'], image)
    print(input_details)

    # 推論を実行する
    interpreter.invoke()
    output_data = interpreter.get_tensor(output_details[0]['index'])

    # 出力結果を確率に変換し、パーセンテージに変換する
    probabilities = tf.nn.softmax(output_data).numpy()
    percentages = probabilities * 100
    return percentages.flatten()
# =============================================

# 画像の分類推論を実行してパーセンテージを取得する
percentages = run_inference(tflite_model_path, image_path)

for i, percentage in enumerate(percentages):
    print(f"Class {i}: {percentage:.2f}%")
```
::::

試しに以下の"**0**"の手書き画像を用いて推論を実行すると、以下のような実行結果が表示されます。
![](https://storage.googleapis.com/zenn-user-upload/79c200f53b5c-20230512.png =100x)

```
Class 0: 99.02%
Class 1: 0.00%
Class 2: 0.00%
Class 3: 0.83%
Class 4: 0.00%
Class 5: 0.01%
Class 6: 0.03%
Class 7: 0.00%
Class 8: 0.11%
Class 9: 0.00%
```
出力結果より、入力した画像は99%の確率で"**0**"と予測されたことがわかります。

# おわり
機械学習といったらすごく高度なことをしていて、簡単には手を出せないイメージですが、代表的な学習モデルであれば無料で公開されていて意外と簡単に実装できたりします。
無料で公開されているデータセットが他にも数多くあるので、他にも色々と試してみてください。

## 備考
[TensorFlow](https://www.tensorflow.org/?hl=ja): Googleが開発した機械学習ライブラリ。
[TensorFlowLite](https://www.tensorflow.org/lite?hl=ja): TensorFlowのモデルをモバイルや組み込みデバイスで実行するための軽量化されたライブラリ。
[Keras](https://keras.io/ja/): ニューラルネットワークライブラリ。簡易な記述でニューラルネットワークを構築できる。TensorFlowのラッパーライブラリ。
