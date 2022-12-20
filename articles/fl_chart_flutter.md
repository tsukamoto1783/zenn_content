---
title: "【Flutter】fl_chart：折れ線グラフで使用するプロパティ詳細"
emoji: "📈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter,Dart,flChart,graph]
published: true
---
# はじめに
fl_chartでシンプルな折れ線グラフを作成してみた。

公式のサンプルはこれ↓
[![altテキスト](/images/fl_chart_flutter/sample2.png)](https://github.com/imaNNeoFighT/fl_chart/blob/master/repo_files/documentations/line_chart.md#sample-2-source-code)


もうちょっとシンプルな仕様が好みなので、プロパティをいじってつくってみたのはこんな感じ↓
![hoge](/images/fl_chart_flutter/fl_chart_demo.png =300x)


# 各種プロパティを見る
[fl_chartのgithub](https://github.com/imaNNeoFighT/fl_chart/blob/master/repo_files/documentations/line_chart.md#sample-2-source-code)見るとわかると思うが、ドキュメントが充実していて非常にありがたい。
大概のことは実装できるかと思われる。

ただ、プロパティの数が多く、充実しているからこそ逆に欲しい情報を探すのがやや面倒だったり、、、
なので、主要なプロパティをざっくりな日本語にして記載していく。

※「プロパティ数がものすごく多い ＆ どこの表示に反映されるかよくわからんやつがある」 といった理由でよく使いそうなプロパティを厳選して記載


```dart
  LineChartData mainData() {
    return LineChartData(

      // タッチ操作時の設定
      lineTouchData: LineTouchData(
        handleBuiltInTouches: true,                        // タッチ時のアクションの有無
        getTouchedSpotIndicator: defaultTouchedIndicators, // インジケーターの設定
        touchTooltipData: LineTouchTooltipData(            // ツールチップの設定
          getTooltipItems: defaultLineTooltipItem,         // 表示文字設定
          tooltipBgColor: Colors.white,                    // 背景の色
          tooltipRoundedRadius: 2.0,                       // 角丸
        ),
      ),

      // 背景のグリッド線の設定
      gridData: FlGridData(
        show: true,                          // 背景のグリッド線の有無
        drawVerticalLine: true,              // 水平方向のグリッド線の有無
        horizontalInterval: 1.0,             // 背景グリッドの横線間隔
        verticalInterval: 1.0,               // 背景グリッドの縦線間隔
        getDrawingHorizontalLine: (value) {  // 背景グリッドの横線設定
          return FlLine(
            color: Color(0xff37434d),        // 背景横線の色
            strokeWidth: 1.0,                // 背景横線の太さ
          );
        },
        getDrawingVerticalLine: (value) {    // 背景グリッドの縦線設定
          return FlLine(
            color: Color(0xff37434d),        // 背景縦線の色
            strokeWidth: 1.0,                // 背景縦線の太さ
          );
        },
      ),

      // グラフのタイトル設定
      titlesData: FlTitlesData(
        show: true,                          // タイトルの有無
        bottomTitles: AxisTitles(            // 下側に表示するタイトル設定
          axisNameWidget: Text("【曜日】",      // タイトル名
            style: TextStyle(
              color: Color(0xff68737d),
            ),
          ),
          axisNameSize: 22.0,                     // タイトルの表示エリアの幅
          sideTitles: SideTitles(                 // サイドタイトル設定
            showTitles: true,                     // サイドタイトルの有無
            interval: 1.0,                        // サイドタイトルの表示間隔
            reservedSize: 40.0,                   // サイドタイトルの表示エリアの幅
            getTitlesWidget: bottomTitleWidgets,  // サイドタイトルの表示内容
          ),
        ),
        rightTitles: AxisTitles(),                // 上記と同じため割愛
        topTitles: AxisTitles(),
        leftTitles: AxisTitles()
      ),

      // グラフの外枠線
      borderData: FlBorderData(
        show: true,                   // 外枠線の有無
        border: Border.all(           // 外枠線の色
          color: Color(0xff37434d),
        ),
      ),

      // グラフのx軸y軸のの表示数
      minX: 0.0,
      maxX: 6.0,
      minY: 0.0,
      maxY: 6.0,

      // チャート線の設定
      lineBarsData: [
        LineChartBarData(
          spots: [                      // 表示する座標のリスト
            FlSpot(0.0, 3.0),
            FlSpot(1.0, 2.0),
            FlSpot(2.0, 5.0),
            FlSpot(3.0, 3.0),
            FlSpot(4.0, 4.0),
            FlSpot(5.0, 3.0),
            FlSpot(6.0, 4.0),
          ],
          isCurved: false,              // チャート線を曲線にするか折れ線にするか
          barWidth: 1.0,                // チャート線幅
          isStrokeCapRound: false,      // チャート線の開始と終了がQubicかRoundか（？）
          dotData: FlDotData(
            show: true,                 // 座標のドット表示の有無
            getDotPainter: (spot, percent, barData, index) =>
                FlDotCirclePainter(     // ドットの詳細設定
              radius: 2.0,
              color: Colors.blue,
              strokeWidth: 2.0,
              strokeColor: Colors.blue,
            ),
          ),
          belowBarData: BarAreaData(    // チャート線下部に色を付ける場合の設定
            show: false,                // チャート線下部の表示の有無
          ),
        ),
      ],
    );
  }

```


::::details コード全容
```dart
import 'package:fl_chart/fl_chart.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      title: 'fl_chart sample',
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  /// 色は単色が良かったので、グラデーションは使用しない
  // List<Color> gradientColors = [
  //   const Color(0xff23b6e6),
  //   const Color(0xff02d39a),
  // ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('fl_chart sample'),
      ),
      body: Center(
        // グラフ表示画面(大枠)の設定
        child: AspectRatio(
          aspectRatio: 1.414,
          child: DecoratedBox(
            decoration: const BoxDecoration(
              borderRadius: BorderRadius.all(
                Radius.circular(18),
              ),
              color: Color(0xff232d37),
            ),
            child: Padding(
              padding: const EdgeInsets.only(
                right: 18,
                left: 12,
                top: 24,
                bottom: 12,
              ),
              child: LineChart(
                mainData(),
              ),
            ),
          ),
        ),
      ),
    );
  }

  LineChartData mainData() {
    return LineChartData(
      // タッチ操作時の設定
      lineTouchData: LineTouchData(
        handleBuiltInTouches: true, // タッチ時のアクションの有無
        getTouchedSpotIndicator: defaultTouchedIndicators, // インジケーターの設定
        // ツールチップの設定
        touchTooltipData: LineTouchTooltipData(
          getTooltipItems: defaultLineTooltipItem, // 表示文字設定
          tooltipBgColor: Colors.white, // 背景の色
          tooltipRoundedRadius: 2.0, // 角丸
        ),
      ),

      // 背景のグリッド線の設定
      gridData: FlGridData(
        show: true, // 背景のグリッド線の有無
        drawVerticalLine: true, // 水平方向のグリッド線の有無
        horizontalInterval: 1.0, // 背景グリッドの横線間隔
        verticalInterval: 1.0, // 背景グリッドの縦線間隔
        // 背景グリッドの横線設定
        getDrawingHorizontalLine: (value) {
          return FlLine(
            color: const Color(0xff37434d),
            strokeWidth: 1.0, // 線の太さ
          );
        },
        // 背景グリッドの縦線設定
        getDrawingVerticalLine: (value) {
          return FlLine(
            color: const Color(0xff37434d),
            strokeWidth: 1.0, // 線の太さ
          );
        },
      ),

      // グラフのタイトル設定
      titlesData: FlTitlesData(
        show: true, // タイトルの有無
        rightTitles: AxisTitles(),
        topTitles: AxisTitles(),
        // 下側のタイトル設定
        bottomTitles: AxisTitles(
          // タイトル名
          axisNameWidget: const Text(
            "[曜日]",
            style: TextStyle(
              color: Color(0xff68737d),
            ),
          ),
          axisNameSize: 22.0, //タイトルの表示エリアの幅
          // サイドタイトルの設定
          sideTitles: SideTitles(
            showTitles: true, // サイドタイトルの有無
            interval: 1.0, // サイドタイトルの表示間隔
            reservedSize: 40.0, // サイドタイトルの表示エリアの幅
            getTitlesWidget: bottomTitleWidgets, // サイドタイトルの表示内容
          ),
        ),
        leftTitles: AxisTitles(
          axisNameWidget: const Text(
            "【値】",
            style: TextStyle(
              color: Color(0xff68737d),
            ),
          ),
          axisNameSize: 28.0, //タイトルの表示エリアの幅
          sideTitles: SideTitles(
            showTitles: true, // サイドタイトルの表示・非表示
            interval: 1.0, // サイドタイトルの表示間隔
            reservedSize: 42.0, // サイドタイトルの表示エリアの幅
            getTitlesWidget: leftTitleWidgets,
          ),
        ),
      ),

      // グラフの外枠線
      borderData: FlBorderData(
        show: true, // 外枠線の有無
        border: Border.all(
          color: const Color(0xff37434d),
        ),
      ),

      // グラフのx軸y軸のの表示数(最大値)
      minX: 0.0,
      maxX: 6.0,
      minY: 0.0,
      maxY: 6.0,

      // チャート線の設定
      lineBarsData: [
        LineChartBarData(
          // 表示する座標のリスト
          spots: const [
            FlSpot(0.0, 3.0),
            FlSpot(1.0, 2.0),
            FlSpot(2.0, 5.0),
            FlSpot(3.0, 3.0),
            FlSpot(4.0, 4.0),
            FlSpot(5.0, 3.0),
            FlSpot(6.0, 4.0),
          ],
          isCurved: false, // チャート線を曲線にするか折れ線にするか
          /// グラデーションは使用しない
          // gradient: LinearGradient(
          //   colors: gradientColors,
          // ),
          barWidth: 1.0, // チャート線幅
          isStrokeCapRound: false, // チャート線の開始と終了がQubicかRoundか（？）
          dotData: FlDotData(
            show: true, // 座標にドット表示の有無
            // ドットの詳細設定
            getDotPainter: (spot, percent, barData, index) =>
                // ドットの詳細設定
                FlDotCirclePainter(
              radius: 2.0,
              color: Colors.blue,
              strokeWidth: 2.0,
              strokeColor: Colors.blue,
            ),
          ),
          // チャート線下部に色を付ける場合の設定
          belowBarData: BarAreaData(
            show: false, // チャート線下部の表示の有無

            /// グラデーションは使用しない
            // gradient: LinearGradient(
            //   colors: gradientColors
            //       .map((color) => color.withOpacity(0.3))
            //       .toList(),
            // ),
          ),
        ),
      ],
    );
  }

  Widget bottomTitleWidgets(double value, TitleMeta meta) {
    const style = TextStyle(
      color: Color(0xff68737d),
      fontWeight: FontWeight.bold,
      fontSize: 16.0,
    );
    Widget text;
    switch (value.toInt()) {
      case 0:
        text = const Text('月', style: style);
        break;
      case 1:
        text = const Text('火', style: style);
        break;
      case 2:
        text = const Text('水', style: style);
        break;
      case 3:
        text = const Text('木', style: style);
        break;
      case 4:
        text = const Text('金', style: style);
        break;
      case 5:
        text = const Text('土', style: style);
        break;
      case 6:
        text = const Text('日', style: style);
        break;
      default:
        text = const Text('', style: style);
        break;
    }

    return SideTitleWidget(
      axisSide: meta.axisSide,
      child: text,
    );
  }

  Widget leftTitleWidgets(double value, TitleMeta meta) {
    const style = TextStyle(
      color: Color(0xff68737d),
      fontWeight: FontWeight.bold,
      fontSize: 15.0,
    );
    String text;
    switch (value.toInt()) {
      case 1:
        text = '10';
        break;
      case 3:
        text = '30';
        break;
      case 5:
        text = '50';
        break;
      default:
        return Container();
    }

    return Text(text, style: style, textAlign: TextAlign.left);
  }
```
::::

# Firestoreなどから値を取得してグラフ表示したい場合はデータ変換必須
表示する座標を設定する以下のコード箇所について、
実際に使用する場合は、座標を固定値で定義するよりは、FirestoreなどのDBから値を取得して、その値を座標として表示することになるかと思う。
```dart
      // チャート線の設定
      lineBarsData: [
        LineChartBarData(
          // 表示する座標のリスト
          spots: [
            FlSpot(0.0, 3.0),
            FlSpot(1.0, 2.0),
            FlSpot(2.0, 5.0),
            FlSpot(3.0, 3.0),
            FlSpot(4.0, 4.0),
            FlSpot(5.0, 3.0),
            FlSpot(6.0, 4.0),
          ],
```
値を取得した後、座標表示形式（FlSpot()のList）に変換する処理は必須になるかと。


# おわり
flutterのグラフパッケージの中では１番like数が多くてメジャーなパッケージかと。

ただ、現在はアプデが止まっている状態。(2022/10月頃から)
ですが、ドキュメントが充実していて使い勝手がいいので、今のところ採用しても全然アリかと個人的には思ってます。

また今度、fl_chart内の他の種類のグラフも試してみようと思います。(comming soon ...)


## ※おまけ
他のグラフパッケージも軽く紹介だけ。

[charts_flutter](https://pub.dev/packages/charts_flutter)
like数が1.3Kのグラフパッケージ。
最終アプデが1年以上経っていて、DISCONTINUEDになっているのが少しネック？
（実際に触ったことなくて中身を知らないので、なんとも言えないが、、）

[graphic](https://pub.dev/packages/graphic)
軽くいじってみてシンプルなグラフは実装できた。
fl_chartに比べるとドキュメントがあまり充実していない印象で、細かい調整は難しいかも？
シンプルで好みのグラフが作成できそうなので、もうちょっと詳しく触ってみたい。

・[syncfusion_flutter_charts](https://pub.dev/packages/syncfusion_flutter_charts)
fl_chartの次にlike数が多いグラフパッケージ。
ライセンスが必要ですが、個人で使う分には無料なので、今度軽く使ってみたい。
有料だけあって中身は充実はしてるらしいので。


## 参考記事
::::details 参考記事
https://zenn.dev/corrupt/articles/cfe587c87f101f

https://afgprogrammer.com/blog/how-to-use-charts-in-flutter/

https://dev.classmethod.jp/articles/flutter-implementing-line-chart-line-chart-in-fl-chart/

::::