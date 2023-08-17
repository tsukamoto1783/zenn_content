---
title: "【Flutter】statefulWidgetとflutter_hooksでアニメーション実装"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter,dart,animation,statefulWidget,hooks]
published: true
---

flutterのstatefulWidgetとflutter_hooksで、基本的なアニメーション実装してみる。

アニメーションの実装は少しめんどくさい印象があって、個人的に食わず嫌いなところがあったが、今回色々と試しながら実装してみた。
animationに関しては便利なパッケージもいくつかあるが、今回は根本的な挙動確認も行ないため、パッケージは使用しないものとする。

SNSのいいね機能でありそうなアニメーションを例に実装してみる。

![](https://storage.googleapis.com/zenn-user-upload/fe33c325ee13-20230817.gif =200x)

-----------------
<br>

# statefulパターン
```dart
void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
    // ...statefulのおまじない部分は記載省略
  State createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget>
    with SingleTickerProviderStateMixin {
  // with SingleTickerProviderStateMixin を忘れずに

  late AnimationController controller;
  late Animation<double> scaleAnimation;
  late Animation<double> fadeAnimation;
  Color color = Colors.grey;

  @override
  void initState() {
    super.initState();

    controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 500),
    )..addStatusListener((status) {
        if (status == AnimationStatus.completed) {
          // animationの再生が終了すると、色を変更してanimationの状態をリセットする
          setState(() {
            color = Colors.grey;
          });
          controller.reset();
        }
      });

    scaleAnimation = Tween<double>(
      begin: 1.0, // アニメーション開始時のスケール
      end: 2.0, // アニメーション終了時のスケール
    ).animate(controller);

    fadeAnimation = Tween<double>(
      begin: 1.0, // アニメーション開始時のスケール
      end: 0.0, // アニメーション終了時のスケール
    ).animate(controller);
  }

  @override
  void dispose() {
    controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Animation Sample'),
      ),
      body: Center(
        child: Stack(
          children: [
            Center(
              child: FadeTransition(
                opacity: fadeAnimation,
                child: ScaleTransition(
                  scale: scaleAnimation,
                  child: const Icon(
                    Icons.favorite,
                    size: 100.0,
                    color: Colors.red,
                  ),
                ),
              ),
            ),
            Center(
              child: Icon(
                Icons.favorite,
                size: 100.0,
                color: color,
              ),
            ),
            Center(
              child: Padding(
                padding: const EdgeInsets.only(top: 300.0),
                child: IconButton(
                  onPressed: () {
                    setState(() {
                      color = Colors.red;
                    });
                    // アニメーション再生
                    controller.forward();
                  },
                  icon: const Icon(Icons.play_arrow),
                  iconSize: 60,
                ),
              ),
            )
          ],
        ),
      ),
    );
  }
}
```

### ポイント1：SingleTickerProviderStateMixin
`AnimationController`を使用する際には、必須で`vsync`を指定する必要がある。
おまじない的に`vsync:this`とすることが一般的だが、thisを指定するためには、
`with SingleTickerProviderStateMixin`を付与してあげる必要がある。

※`vsync:this`や`SingleTickerProviderStateMixin`についての詳細はここでは割愛。
<!-- 詳細を調べて備考欄として末尾に書きたい。 -->

```dart
class _MyWidgetState extends State<MyWidget>
    with SingleTickerProviderStateMixin {
    // ...省略

    controller = AnimationController(
      vsync: this,
      // ...省略
```

### ポイント2：アニメーションの制御・検知
アニメーションの再生や停止、リセットなどを検知するには、`AnimationController`に対して`addStatusListener`を使用すると簡単に制御できる。
アニメーションの状態がstatusに入ってくるので、それを元に処理を行う。
<!-- 他の便利なメソッドがないかも調べる -->
<!-- statusの種類も調べる -->

```dart
controller = AnimationController(
  vsync: this,
  duration: const Duration(milliseconds: 500),
)..addStatusListener((status) {
    if (status == AnimationStatus.completed) {
      // animationの再生が終了すると、色を変更してanimationの状態をリセットする
      setState(() {
        color = Colors.grey;
      });
      controller.reset();
    }
  });
```
以下のようにも書ける。
```dart
    void statusListener(AnimationStatus status) {
      if (status == AnimationStatus.completed) {
        // animationの再生が終了すると、色を変更してanimationの状態をリセットする
        setState(() {
          color = Colors.grey;
        });
        controller.reset();
      }
    }

    controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 500),
    );

    controller.addStatusListener(statusListener);

```

### ポイント3：コントローラーの破棄
`AnimationController`は、`dispose`を呼ばないとメモリリークするので、`dispose`をオーバーライドして破棄する。
また、コントローラーに紐づいたリスナーについては、コントローラーの破棄と一緒にリスナーも破棄されるため、別途リスナーを破棄する処理の記述は不要。
```dart
  @override
  void dispose() {
    controller.dispose();

    super.dispose();
  }
```

-----------------
<br>

# hooksパターン
```dart
void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: MyWidget(),
    );
  }
}

class MyWidget extends HookWidget {
  const MyWidget({super.key});

  @override
  Widget build(BuildContext context) {
    final controller = useAnimationController(
      duration: const Duration(milliseconds: 500),
    );

    final scaleAnimation = Tween<double>(
      begin: 1.0,
      end: 2.0,
    ).animate(controller);

    final fadeAnimation = Tween<double>(
      begin: 1.0,
      end: 0.0,
    ).animate(controller);

    final color = useState(Colors.grey);

    // 画面読み込み時に一度だけ実行される
    useEffect(() {
      void statusListener(AnimationStatus status) {
        if (status == AnimationStatus.completed) {
          color.value = Colors.grey;
          controller.reset();
        }
      }

      // アニメーションの状態を監視する
      controller.addStatusListener(statusListener);

      // 画面破棄時に実行される
      return () {
        // 状態監視をしていたリスナーを削除する
        // MEMO: controllerの破棄は、useAnimationControllerが自動で行ってくれる
        controller.removeStatusListener(statusListener);
      };
    }, []);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Animation Sample'),
      ),
      body: Center(
        child: Stack(
          children: [
            Center(
              child: FadeTransition(
                opacity: fadeAnimation,
                child: ScaleTransition(
                  scale: scaleAnimation,
                  child: const Icon(
                    Icons.favorite,
                    size: 100.0,
                    color: Colors.red,
                  ),
                ),
              ),
            ),
            Center(
              child: Icon(
                Icons.favorite,
                size: 100.0,
                color: color.value,
              ),
            ),
            Center(
              child: Padding(
                padding: const EdgeInsets.only(top: 300.0),
                child: IconButton(
                  onPressed: () {
                    color.value = Colors.red;
                    controller.forward();
                  },
                  icon: const Icon(Icons.play_arrow),
                  iconSize: 60,
                ),
              ),
            )
          ],
        ),
      ),
    );
  }
}
```

### ポイント1：useEffect
`useEffect`を使用すると、statefulでいう`initState`や`dispose`のような処理をまとめて管理できる。

※詳細な`useEffect`についての書き方や挙動については、この記事では割愛。

```dart
// 画面読み込み時に一度だけ実行される
useEffect(() {
  void statusListener(AnimationStatus status) {
    if (status == AnimationStatus.completed) {
      color.value = Colors.grey;
      controller.reset();
    }
  }
  // アニメーションの状態を監視する
  controller.addStatusListener(statusListener);

  // return 以下は、画面破棄時に実行される
  return () {
    // 状態監視をしていたリスナーを削除する
    controller.removeStatusListener(statusListener);
    
    // MEMO: controllerの破棄は、useAnimationControllerが自動で行ってくれる
  };
}, []);
```


### ポイント2：useAnimationController
flutter_hooksには`useAnimationController`というhooksが用意されている。
このhooksを使用すると、`vsync`を指定しなくても`AnimationController`のインスタンスを生成できる。
また、hooksを使用すると、ライフサイクルも自動で管理してくれるため、statefulでいう`@override void dispose()`の記述も不要となる。

```dart
    final controller = useAnimationController(
      duration: const Duration(milliseconds: 500));
```

ただ、hooksで作成したコントローラーにリスナーを追加した場合は、リスナー破棄の処理が必要となる。（hooksが自動でリスナーまでは破棄してくれない。。）
[上記ポイント1](#ポイント1：useeffect)のコード部分参照。

-----------------
<br>

# おわり
すごくシンプルなアニメーションの実装を、statefulWidgetとflutter_hooksのそれぞれのパターンで実装してみた。
まだまだ理解が浅い部分があるので、もっとよい実装があれば随時更新していきたい。

flutter_hooksの方に関しては、useEffectの使い方や挙動の理解が重要になってくるので、別途挙動調査して理解を深める。

[調査中...]()