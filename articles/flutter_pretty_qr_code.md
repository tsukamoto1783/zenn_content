---
title: "【Flutter】QRコードを生成して表示する"
emoji: "🎼"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, Dart, QRコード, PrettyQrCode, QR]
published: true
publication_name: ncdc
---

## はじめに

「QR コードを生成したい。」

ということで、[pretty_qr_code](https://pub.dev/packages/pretty_qr_code)ライブラリを用いて、QR コードを生成してみました。

`pretty_qr_code`では、QR コード生成のデモが用意されているので、まずはそちらでできることの全体像をご確認ください。

[pretty_qr_code デモ: Live Preview](https://promops.github.io/flutter_pretty_qr/)

<br>

【検証時の環境】

```log
flutter doctor
[✓] Flutter (Channel stable, 3.29.2, on macOS 15.5 24F74 darwin-arm64, locale ja-JP)
[✓] Android toolchain - develop for Android devices (Android SDK version 33.0.2)
[✓] Xcode - develop for iOS and macOS (Xcode 16.3)
[✓] Chrome - develop for the web
[✓] Android Studio (version 2024.1)
[✓] VS Code (version 1.101.1)
[✓] VS Code (version 1.100.0-insider)
[✓] Connected device (7 available)
[✓] Network resources
```

<br>

## QR コードに関する用語

QR コードについての知識や用語が分からないと適切な設定ができないので、まずは最低限ライブラリで使用する用語について調べてみました。

### クワイエットゾーン

QR コード周囲の余白（マージン）のこと。
正確に QR コードを読み取るために必要なスペースです。

### 誤り訂正レベル

QR コードが一部破損していたり汚れていたとしても、QR コード自身でデータを復元する機能のこと。
この機能のレベルは、L（Low）、M（Medium）、Q（Quartile）、H（High）の 4 段階があります。
レベルが高いほど復元可能能力は向上しますが、データ量も増えます。どのような環境で QR コードを使用するかによって、適切なレベルを選択する必要があります。

※ スマホ表示であれば、破損や汚れ等は基本的に発生しないので、L（Low）で十分かもしれません。デフォルトも L（Low）なことより。

### QR コードのバージョン

QR コードのサイズやデータ容量を決定する値。
バージョンは 1 から 40 まであります。
バージョンが大きいほど、より多くのデータを格納できます。

→ QR コードに載せるデータ量（文字数）によって、適切な値を指定する必要があります。

### モジュール

QR コードを構成する 1 つの正方形のこと。セルとも呼ばれます。

`QR コードバージョン：1` を例に見ると、
21 x 21 のモジュールで構成されている、すなわち、１つの白黒の正方形（1 モジュール）が 21 x 21 で並んでいるイメージです。

コードのバージョンを大きくすると、並ぶモジュール数も多くなり、結果的に多くの情報を埋め込めることになります。

<br>

## シンプルな QR コードの生成

まずはシンプルな QR コードの生成から見てみます。
一行書くだけで QR コードが Widget として表示されます。

```dart
PrettyQrView.data(data: 'https://flutter.dev')
```

生成された QR コードをスマホから読み込むと、指定した URL が反映されていることが確認できます。
（お手元のスマホで読み取ると認識されることが確認できます。）

![](https://storage.googleapis.com/zenn-user-upload/75c0aaf1930b-20250625.png =200x)

<br>

## QR コードのカスタマイズ

`PrettyQrView.data`では、以下のパラメータを指定できます。

```dart
Widget data({
  Key? key,
  required String data,                          // QRコードに埋め込むデータ文字列
  PrettyQrDecoration? decoration,                // QRコードのレイアウト調整
  int errorCorrectLevel = QrErrorCorrectLevel.L, // QRコードの誤り訂正レベル（L, M, Q, H）
  Widget Function(BuildContext, Object, StackTrace?)? errorBuilder, // QRコード生成失敗時に代わりに表示するWidget
})
```

`PrettyQrDecoration`について、以下に設定できることを記載します。

```dart
PrettyQrDecoration PrettyQrDecoration({
  PrettyQrDecorationImage? image, // QRコードの中央に配置する画像設定
  PrettyQrQuietZone? quietZone,   // クワイエットゾーンの指定。特段指定がなければ、`PrettyQrQuietZone.standard` の指定で良さそう。直接指定したい場合は.pixels(num)も可。
  Color? background,              // 背景色
  PrettyQrShape shape,            // QRコードの形状設定
})

// ==================
const PrettyQrDecorationImage({
  required super.image,                     // 画像のパス
  super.scale = 0.2,                        // 比率。2.0以下が推奨
  super.onError,                            // 画像読み込み失敗時のコールバック
  super.colorFilter,                        // 画像の色合いの指定
  super.fit,                                // 画像のフィット方法（BoxFit）
  super.repeat = ImageRepeat.noRepeat,      // 画像が小さい場合の繰り返し方法
  super.matchTextDirection = false,         // 画像の水平反転？
  super.opacity = 1.0,                      // 画像の不透明度
  super.filterQuality = FilterQuality.low,  // 画像の拡大縮小する際の品質
  super.invertColors = false,               // 画像の色を反転
  super.isAntiAlias = false,                // 画像のエッジを滑らかにするか
  this.padding = EdgeInsets.zero,           // padding
  this.position = PrettyQrDecorationImagePosition.embedded, // 画像の位置
}) : assert(scale >= 0 && scale <= 1);

// ==================
const PrettyQrSmoothSymbol({
    this.roundFactor = 1,                 // QRコードのドットの角の丸み設定(0.0〜1.0)
    this.color = const Color(0xFF000000), // QRコードの色
})  : assert(roundFactor <= 1, 'roundFactor must be less than 1'),
    assert(roundFactor >= 0, 'roundFactor must be greater than 0');
```

| QR①                                                                                  | QR②                                                                                  |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| ![](https://storage.googleapis.com/zenn-user-upload/905ea33e54d9-20250626.png =200x) | ![](https://storage.googleapis.com/zenn-user-upload/76c648ce43ae-20250626.png =200x) |

:::details コードサンプル

```dart
// QR①
PrettyQrView.data(
    data: 'https://flutter.dev',
    decoration: const PrettyQrDecoration(
    image: PrettyQrDecorationImage(
        image: AssetImage('assets/images/flutter_icon.png'),
        scale: 1,
        padding: EdgeInsets.all(40),
        position: PrettyQrDecorationImagePosition.foreground,
    ),
    quietZone: PrettyQrQuietZone.standart,
    shape: PrettyQrSmoothSymbol(
        color: Colors.blue,
        roundFactor: 1,
    ),
    background: Colors.lightGreen,
    ),
    errorCorrectLevel: QrErrorCorrectLevel.H,
),

// QR②
PrettyQrView.data(
    data: 'https://flutter.dev',
    decoration: const PrettyQrDecoration(
    image: PrettyQrDecorationImage(
        image: AssetImage('assets/images/flutter_icon.png'),
        position: PrettyQrDecorationImagePosition.background,
        invertColors: true,
    ),
    quietZone: PrettyQrQuietZone.standart,
    shape: PrettyQrSmoothSymbol(
        color: Colors.deepPurple,
        roundFactor: 0,
    ),
    background: Colors.lightGreen,
    ),
    errorCorrectLevel: QrErrorCorrectLevel.L,
),
```

:::

<br>

文字列以外にバイナリーデータを QR に渡したい場合や、QR コードのバージョンを指定したい場合は、デフォルトの PrettyQrView コンストラクタを用いて指定できます。

```dart
final qrCode = QrCode(
    8,
    QrErrorCorrectLevel.H,
)..addData('lorem ipsum dolor sit amet');
// ..addByteData(byteData) // バイナリデータの場合

final qrImage = QrImage(qrCode);

// ....
@override
Widget build(BuildContext context) {
    return PrettyQrView(qrImage: qrImage)

// ===============
class QrCode {
  final int typeNumber;        // QRコードのバージョン（1〜40）
  final int errorCorrectLevel; // QRコードの誤り訂正レベル（L, M, Q, H）
  final int moduleCount;       // QRコードのモジュール数。「typeNumber * 4 + 17」で自動計算されるため、直接指定は不可。
}
```

> Note: Do not create QrImage inside the build method; otherwise, you may have an undesired jank in the UI thread.

※ build の中で `QrImage` を生成すると不具合が発生するおそれがあるみたいなので、注意が必要です。

![](https://storage.googleapis.com/zenn-user-upload/1aa9c12cf13b-20250626.png =200x)

:::details コードサンプル

```dart
final qrImage = QrImage(
    QrCode(
        12,
        QrErrorCorrectLevel.H,
    )..addData('lorem ipsum dolor sit amet'),
);

@override
Widget build(BuildContext context) {
    return PrettyQrView(
        qrImage: qrImage,
        decoration: const PrettyQrDecoration(
        quietZone: PrettyQrQuietZone.standart,
        shape: PrettyQrSmoothSymbol(
            roundFactor: 0,
        ),
        ),
    ),
```

:::

## 備考

画像の配置を`PrettyQrDecorationImagePosition.embedded`で配置すると、値が読み込めない現象が発生。
`QrErrorCorrectLevel`はデフォルトでは`Low`が指定されているため、誤り訂正レベルを上げて`QrErrorCorrectLevel.H`を指定することで、読み取り可能な QR コードが生成されるようになりました。

https://github.com/promops/flutter_pretty_qr/issues/43

| QrErrorCorrectLevel.L                                                                | QrErrorCorrectLevel.H                                                                |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| ![](https://storage.googleapis.com/zenn-user-upload/ee99c65b1fa6-20250626.png =200x) | ![](https://storage.googleapis.com/zenn-user-upload/c34676c08932-20250626.png =200x) |

## 参考リンク

:::details 参考リンク
https://www.mediaseek.co.jp/barcode/13614/
https://www.qrcode.com/about/error_correction.html
https://www.qrcode.com/about/version.html
https://media9.co.jp/m_tuhan/contents/article1/
:::
