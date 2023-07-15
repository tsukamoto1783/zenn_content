---
title: "【Flutter】チュートリアルなどで使用するコーチマーク実装"
emoji: "🔍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter ,dart ,tutorial ,tutorialCoachMark]
published: true
---

[tutorial_coach_mark](https://pub.dev/packages/tutorial_coach_mark)パッケージを用いて、チュートリアルなどで使用するコーチマークを実装する。

どういったことができるのかは[tutorial_coach_mark](https://pub.dev/packages/tutorial_coach_mark)のREADMEの動画を見るとイメージがつきやすい。

<br>

## コーチマークとは
コーチマークとは、アプリの使い方を視覚的に教えるヒントのこと。
短く、際立たせたい機能に焦点を当てて必要最低限の情報だけを視覚的に説明する機能のこと。
コーチマークがあることによって、ユーザーはアプリの特定の機能の使い方を簡単に理解できるようになる。

参考：[Nielsen Norman Group / Instructional Overlays and Coach Marks for Mobile Apps](https://www.nngroup.com/articles/mobile-instructional-overlay/)

<br>

## チュートリアルとコーチマークの違い
どちらもアプリの使用方法を教えるような似た機能を提供するが、以下の観点で少し違いがある。

**チュートリアル：** アプリの全体的な使い方や流れを詳細に教える（伝える）もの。割と長尺になることもある。

**コーチマーク：** 特定の機能に対して、その機能の使い方を視覚的に端的に教える（伝える）もの。より短く端的にが基本。

<br>


## 実装ポイント
ライブラリを使用する上で、ポイントとなるclassは以下の３つ。

:::message
- TutorialCoachMark
- TargetFocus
- TargetContent
:::

以下にそれぞれのclassで使用できるプロパティやメソッドを記載する。

<br>


### TutorialCoachMark
TutorialCoachMarkは、チュートリアルの全体的な動作と見た目を設定する。
各ステップの管理や、アニメーションの設定、スキップボタンの動作、チュートリアルのオーバーレイの見た目など。

※TutorialCoachMarkについてはREADMEに記載がないので、ソースコードを読んで記載のあったものを記載。


| プロパティ                   | 型                                                    | 説明                                                                        |
| ---------------------------- | ----------------------------------------------------- | --------------------------------------------------------------------------- |
| targets                      | List<TargetFocus>                                     | 各ステップ（各ターゲット）の中身・定義                                      |
| onClickTarget                | FutureOr<void> Function(TargetFocus)?                 | 各ターゲットがクリックされたときのコールバック                              |
| onClickTargetWithTapPosition | FutureOr<void> Function(TargetFocus, TapDownDetails)? | 各ターゲットが特定の位置でクリックされたときのコールバック                  |
| onClickOverlay               | FutureOr<void> Function(TargetFocus)?                 | オーバーレイへのTap有効時に、オーバーレイがクリックされたときのコールバック |
| onFinish                     | Function()?                                           | チュートリアルが終了したときのコールバック                                  |
| paddingFocus                 | double                                                | フォーカスエリアのパディング                                                |
| onSkip                       | Function()?                                           | スキップボタンがクリックされたときのコールバック                            |
| alignSkip                    | AlignmentGeometry                                     | スキップボタンの位置                                                        |
| textSkip                     | String                                                | スキップボタンのテキスト                                                    |
| textStyleSkip                | TextStyle                                             | スキップボタンのテキストスタイル                                            |
| hideSkip                     | bool                                                  | スキップボタンを隠すかどうか                                                |
| colorShadow                  | Color                                                 | チュートリアルのオーバーレイの色                                            |
| opacityShadow                | double                                                | オーバーレイの透明度                                                        |
| focusAnimationDuration       | Duration                                              | フォーカスアニメーションの期間                                              |
| unFocusAnimationDuration     | Duration                                              | フォーカス解除アニメーションの期間                                          |
| pulseAnimationDuration       | Duration                                              | パルスアニメーションの期間                                                  |
| pulseEnable                  | bool                                                  | パルスアニメーションを有効にするかどうか                                    |
| skipWidget                   | Widget?                                               | スキップボタンのカスタムウィジェット                                        |
| showSkipInLastTarget         | bool                                                  | 最後のターゲットでスキップボタンを表示するかどうか                          |
| imageFilter                  | ImageFilter?                                          | イメージフィルター                                                          |

<br>

| メソッド                  | 説明                                                                             |
| ------------------------- | -------------------------------------------------------------------------------- |
| show                      | チュートリアルを表示                                                             |
| showWithNavigatorStateKey | `navigatorKey`を使用してチュートリアルを表示                                     |
| showWithOverlayState      | `OverlayState`を使用してチュートリアルを表示                                     |
| finish                    | チュートリアルを終了して、`onFinish`コールバックを呼び出し、オーバーレイを削除   |
| skip                      | チュートリアルをスキップして、`onSkip`コールバックを呼び出し、オーバーレイを削除 |
| isShowing                 | チュートリアルが表示中かどうかの判定                                             |
| next                      | 次のチュートリアルステップに進む                                                 |
| previous                  | 前のチュートリアルステップに戻る                                                 |

<br>

### TargetFocus
TargetFocusは、フォーカス後に表示される内容を設定する。
  
| プロパティ               | 型              | 説明                                                                                    |
| ------------------------ | --------------- | --------------------------------------------------------------------------------------- |
| identify                 | dynamic         | 識別用                                                                                  |
| keyTarget                | GlobalKey       | フォーカスしたいウィジェットのGlobalKey                                                 |
| targetPosition           | TargetPosition  | GlobalKeyを使用しない場合、フォーカスする場所を決定するためにTargetPositionを作成できる |
| contents                 | ContentTarget[] | ウィジェットにフォーカスした後に表示したいコンテンツのリスト                            |
| shape                    | ShapeLightFocus | Circle or RRect（丸 or 角丸四角）                                                       |
| radius                   | double          | shapeがRRectの場合に使用                                                                |
| borderSide               | BorderSide      | フォーカス枠の設定                                                                      |
| color                    | Color           | ターゲットのカスタムカラー                                                              |
| enableOverlayTab         | bool            | オーバーレイ（全画面）をタップして次のステップを呼び出すことを有効にする                |
| enableTargetTab          | bool            | ターゲットをクリックして次のステップを呼び出すことを有効にする                          |
| alignSkip                | Alignment       | ターゲット内のスキップボタンの位置を指定                                                |
| paddingFocus             | Alignment       | ターゲット内のフォーカスのパディング設定                                                |
| focusAnimationDuration   | Duration        | フォーカス開始時のアニメーションの時間を指定                                            |
| unFocusAnimationDuration | Duration        | フォーカス終了時のアニメーションの時間を指定                                            |
| pulseVariation           | Tween           | パルスアニメーションの間隔を指定                                                        |

<br>

### TargetContent
ContentTargetは、TargetFocus{contents:}のListに追加するコンテンツを設定する。
ウィジェットにフォーカスを当てた後、何が表示され、どのように表示されるかを設定する。
  
| プロパティ     | 型                          | 説明                                                                                         |
| -------------- | --------------------------- | -------------------------------------------------------------------------------------------- |
| align          | AlignContent                | フォーカスしたウィジェットに対して、どの領域にコンテンツを表示するかを指定（上、下、左、右） |
| padding        | EdgeInsets                  | コンテンツのパディング                                                                       |
| child          | Widget                      | 静的なコンテンツを表示                                                                       |
| builder        | Widget                      | 動的なコンテンツを表示                                                                       |
| customPosition | CustomTargetContentPosition | alignがAlignContent.customの場合にカスタム位置を追加                                         |

<br>

## 実装時に詰まった点
### aline設定による領域の違い
フォーカスしたウィジェットに対して、思い通りに表示するのに少し手間取った。
それぞれの階層でalineを適切に指定してあげる必要あり。

```dart: 確認用
    List<TargetFocus> targets = [];

    targets.add(
      TargetFocus(
        keyTarget: keyButton2,
        contents: [
          TargetContent(
            align: ContentAlign.bottom,
            builder: (context, controller) => Stack(
              // alignment: Alignment.topCenter,
              children: [
                Container(
                  color: Colors.red,
                  width: MediaQuery.of(context).size.width,
                  height: MediaQuery.of(context).size.height,
                ),
                Container(
                  color: Colors.blue,
                  width: 50,
                  height: 50,
                ),
              ],
            ),
          ),
        ],
      ),
    );
```

| bottom                                                                               | right                                                                                | left                                                                                 | top                                                                                  |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| ![](https://storage.googleapis.com/zenn-user-upload/ac69fa5a4899-20230715.png =150x) | ![](https://storage.googleapis.com/zenn-user-upload/f828b657dc51-20230715.png =150x) | ![](https://storage.googleapis.com/zenn-user-upload/988ab9d9ad24-20230715.png =150x) | ![](https://storage.googleapis.com/zenn-user-upload/5994f99d7384-20230715.png =150x) |

<br>

### 適切な再描画
setState()などで、フォーカス後のコンテンツにを再描画する際に少し詰まった。
→再描画させたい範囲と、buildの範囲との関係性をしっかりと見直すことで解決。


| before                                                                               | after                                                                                |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| ![](https://storage.googleapis.com/zenn-user-upload/f1ade791a367-20230715.gif =150x) | ![](https://storage.googleapis.com/zenn-user-upload/8c53bc53d8c0-20230715.gif =150x) |

<br>

※以下のサンプルコードは、イマイチいけてないコードですが確認用のためご愛嬌。riverpod等で実装するとよりすっきり書けるかと。

::::details before
```dart
import 'dart:ui';

import 'package:flutter/material.dart';
import 'package:tutorial_coach_mark/tutorial_coach_mark.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'TutorialCoachMark Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  MyHomePageState createState() => MyHomePageState();
}

class MyHomePageState extends State<MyHomePage> {
  late TutorialCoachMark tutorialCoachMark;

  // チェックボックス値
  bool isShow = false;
  GlobalKey key = GlobalKey();

  @override
  void initState() {
    createTutorial();
    Future.delayed(Duration.zero, showTutorial);
    super.initState();
  }

// ============================================================
// setState()によって、build以下が再描画される
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: Container(
        color: isShow == false ? Colors.white : Colors.grey,
        child: Center(
          child: Container(
            key: key,
            color: Colors.blue,
            width: 50,
            height: 50,
          ),
        ),
      ),
    );
  }
// ============================================================

  void showTutorial() {
    tutorialCoachMark.show(context: context);
  }

  void createTutorial() {
    tutorialCoachMark = TutorialCoachMark(
      targets: _createTargets(),
      textSkip: "",
      paddingFocus: 10,
      opacityShadow: 0.5,
      imageFilter: ImageFilter.blur(sigmaX: 8, sigmaY: 8),
    );
  }

  List<TargetFocus> _createTargets() {
    List<TargetFocus> targets = [];

    targets.add(
      TargetFocus(
        keyTarget: key,
        contents: [
          TargetContent(
            align: ContentAlign.bottom,
// ============================================================
// initState()内のcreateTutorial()で定義されるCheckBoxは、build配下では無いため、isShowが変更されても再描画されない。
// 背景色のContainer()はbuild配下なので、isShowが変更されると再描画される。
            builder: (context, controller) => Padding(
              padding: const EdgeInsets.only(top: 40.0),
              child: Transform.scale(
                scale: 2,
                child: Checkbox(
                  value: isShow,
                  onChanged: (value) {
                    setState(
                      () {
                        isShow = !isShow;
                      },
                    );
                  },
                ),
              ),
            ),
// ============================================================
          ),
        ],
      ),
    );

    return targets;
  }
}

```
::::


::::details after
```dart
import 'dart:ui';

import 'package:flutter/material.dart';
import 'package:tutorial_coach_mark/tutorial_coach_mark.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'TutorialCoachMark Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  MyHomePageState createState() => MyHomePageState();
}

class MyHomePageState extends State<MyHomePage> {
  late TutorialCoachMark tutorialCoachMark;

// チェックボックス値
  bool isShow = false;
  GlobalKey key = GlobalKey();

  @override
  void initState() {
    createTutorial();
    Future.delayed(Duration.zero, showTutorial);
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: Container(
        color: isShow == false ? Colors.white : Colors.grey,
        child: Center(
          child: Container(
            key: key,
            color: Colors.blue,
            width: 50,
            height: 50,
          ),
        ),
      ),
    );
  }

  void showTutorial() {
    tutorialCoachMark.show(context: context);
  }

  void createTutorial() {
    tutorialCoachMark = TutorialCoachMark(
      targets: _createTargets(),
      textSkip: "",
      paddingFocus: 10,
      opacityShadow: 0.5,
      imageFilter: ImageFilter.blur(sigmaX: 8, sigmaY: 8),
    );
  }

  List<TargetFocus> _createTargets() {
    List<TargetFocus> targets = [];

    targets.add(
      TargetFocus(
        keyTarget: key,
        contents: [
          TargetContent(
            align: ContentAlign.bottom,
// ============================================================
// 背景色のContainer()がbuild配下なので、isShowが変更されると再描画される。
            builder: (context, controller) {
              return CustomCheckBox(
                (value) {
                  setState(() {
                    isShow = !isShow;
                  });
                },
              );
// ============================================================

            },
          ),
        ],
      ),
    );

    return targets;
  }
}

class CustomCheckBox extends StatefulWidget {
  const CustomCheckBox(this.onCheckedChanged, {super.key});

  final void Function(void) onCheckedChanged;

  @override
  CustomCheckBoxState createState() => CustomCheckBoxState();
}

class CustomCheckBoxState extends State<CustomCheckBox> {
  bool _isShow = false;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.only(top: 40.0),
      child: Transform.scale(
        scale: 2,
// ============================================================
// Container()とは別物として、Checkbox()だけのために状態を監視。
        child: Checkbox(
          value: _isShow,
          onChanged: (value) {
            widget.onCheckedChanged(value);
            setState(
              () {
                _isShow = !_isShow;
              },
            );
          },
        ),
// ============================================================
      ),
    );
  }
}
```
::::