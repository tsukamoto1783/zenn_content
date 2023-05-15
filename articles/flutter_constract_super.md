---
title: "【Flutter】コンストラクタとsuperについて"
emoji: "🏗️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, Dart, コンストラクタ, super, assert]
published: true
---

誰もが目にするFlutterのお馴染みのこれ。

```dart
class MyHomePage extends StatefulWidget {
  MyHomePage({super.key, required this.title});


// ========== Dart2.17以前 ==========
class MyHomePage extends StatefulWidget {
  MyHomePage({Key? key, required this.title}) : super(key: key);
```

おまじないのように毎回使用していたけど、コンストラクタとsuperの関係性をいまいち理解せずに使っていたので、改めちゃんと調べてみた。

--------------
**コンストラクタ：** クラスのインスタンスが生成される際に、自動的に呼び出されるメソッドや初期化処理のこと。

**super：** 親（スーパー）クラスのメソッドやコンストラクタを呼び出すために使用される。コンストラクタ処理で使用されることが一般的。

--------------

まずは親子関係の無いクラスから順に見ていく。

# 親子関係の無いクラス

### 「引数無し」「デフォルトコンストラクタ定義無し」
```dart
class Person {
    // 中身何も無し
}

void main() {
  // Personのインスタンス生成
  Person person = Person();
}
```
```txt :dart実行結果
// 何も処理は実行されない
```

&nbsp;
### 「引数無し」「デフォルトコンストラクタ定義あり」
```dart
class Person {
    Person() {
        print('Personクラスのインスタンスが生成されました');
    }
}

void main() {
  Person person = Person();
}
```
```txt :dart実行結果
Personクラスのインスタンスが生成されました
```


&nbsp;
### 「引数あり」「デフォルトコンストラクタ定義無し」
```dart
class Person {
  String? name;

  Person(this.name);
}

void main() {
  Person person = Person('山田');
  print(person.name);
}
```
```txt :dart実行結果
山田
```

&nbsp;
### 「引数あり」「デフォルトコンストラクタ定義あり」
```dart
class Person {
  String? name;

  Person(this.name) {
    print('私は$nameです');
  }
}

void main() {
  Person person = Person('山田');
}
```
```txt :dart実行結果
私は山田です
```


&nbsp;

-------------
任意のコンストラクタ処理を入れたい時は、名前付きコンストラクタを使用できる。

### 「引数無し」「名前付きコンストラクタ定義あり」
```dart
class Person {
  Person.feature1() {
    print('人は二足歩行です');
  }

  Person.feature2() {
    print('人は哺乳類です');
  }
}

void main() {
  // 名前付きにする事でPersonクラスのインスタンス生成時に、意図したコンストラクタを選択できる
  Person person1 = Person.feature1();
  Person person2 = Person.feature2();
}
```
```txt :dart実行結果
人は二足歩行です
人は哺乳類です
```


&nbsp;
### 「引数あり」「名前付きコンストラクタ定義あり」
```dart
class Person {
  String? name;
  String? nationality;
  String? gender;

  Person({this.name, this.nationality, this.gender}) {
    print('Personクラスのインスタンスが生成されました');
  }

  Person.nationality(this.name, this.nationality) {
    print('$nameは$nationalityです');
  }

  Person.gender(this.name, this.gender) {
    print('$nameは$genderです');
  }
}

void main() {
  // 名前付きにする事でPersonクラスのインスタンス生成時に、意図したコンストラクタを選択できる
  Person person = Person();
  Person person1 = Person.nationality('山田', '日本人');
  Person person2 = Person.gender('山田', '男性');
}
```
```txt :dart実行結果
Personクラスのインスタンスが生成されました
山田は日本人です
山田は男性です
```

&nbsp;
# 親クラスを継承するクラスの場合
親クラスを継承したクラスを作成する場合、子クラスのコンストラクタで親クラスのコンストラクタを呼び出す必要がある。

### パターン①
```dart
// 親クラス
class Person {
  Person() {
    print('Personクラスのインスタンスが生成されました');
  }
}

// 子クラス
class Yamada extends Person {
  Yamada() : super() { 
    print('Yamadaクラスのインスタンスが生成されました');
  }
  
  // 親クラスが「引数無し」かつ「デフォルトコンストラクタ」なら、super()は省略可能。以下の記載でもOK
  // Yamada() {
  //   print('Yamadaクラスのインスタンスが生成されました');
  // }
}

void main() {
  Yamada yamada = Yamada();
}
```
```txt :dart実行結果
Personクラスのインスタンスが生成されました
Yamadaクラスのインスタンスが生成されました
```

&nbsp;
### パターン②
```dart
// 親クラス
class Person {
  String? name;

  Person(this.name) {
    print('私は$nameです');
  }
}

// 子クラス
class Yamada extends Person {
  Yamada() : super('山田') {
    print('Yamadaクラスのインスタンスが生成されました');
    // 継承した親クラスの変数が使える
    print('親クラスのnameの値は「$name」です');
  }
}

void main() {
  Yamada yamada = Yamada();
}
```
```txt :dart実行結果
私は山田です
Yamadaクラスのインスタンスが生成されました
親クラスのnameの値は「山田」です
```

&nbsp;
### パターン③
```dart
// 親クラス
class Person {
  String? name;

  Person(this.name) {
    print('私は$nameです');
  }
}

// 子クラス
class Yamada extends Person {
  // 子クラスの引数の値を親クラス継承時に渡せます  
  Yamada(name) : super(name) {
    print('Yamadaクラスのインスタンスが生成されました');
  }

  // 子クラスの引数の値を親クラス継承時に渡す場合は、下記のように記載省略可能。 
  // Yamada(super.name){
  //   print('Yamadaクラスのインスタンスが生成されました');
  // }

}

void main() {
  Yamada yamada = Yamada('山田');
}
```
```txt :dart実行結果
私は山田です
Yamadaクラスのインスタンスが生成されました
```

&nbsp;
### パターン④
superを使って親クラスのメソッドを呼び出すこともできる。
```dart
// 親クラス
class Person {
  Person() {
    print('Personクラスのインスタンスが生成されました');
  }

  void speakLanguages() {
    print('私は日本語を喋ります');
  }
}

// 子クラス
class Yamada extends Person {
  Yamada() : super() {
    print('Yamadaクラスのインスタンスが生成されました');
    super.speakLanguages();
  }
}

void main() {
  Yamada yamada = Yamada();
}
```
```txt :dart実行結果
Personクラスのインスタンスが生成されました
Yamadaクラスのインスタンスが生成されました
私は日本語を喋ります
```

&nbsp;
# Flutterで使用する場合
Flutterで使用する場合の一例。

![](https://storage.googleapis.com/zenn-user-upload/3880245bf2a9-20230515.png =200x)

```dart
void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'sample',
      home: Scaffold(
        appBar: AppBar(
          title: const Text('sample app'),
        ),
        body: MyCustomContainer(
          child: const MyCustomText('Hello, Flutter!'),
        ),
      ),
    );
  }
}

class MyCustomContainer extends Container {
  MyCustomContainer({
    super.key,
    super.child,
  }) : super(
          margin: const EdgeInsets.all(20.0),
          padding: const EdgeInsets.all(20.0),
          color: Colors.blue,
        );

  // superの書き方省略しない場合
  // MyCustomContainer({Key? key, Widget? child})
  //     : super(
  //           key: key,
  //           margin: const EdgeInsets.all(20.0),
  //           padding: const EdgeInsets.all(20.0),
  //           color: Colors.blue,
  //           child: child);
}

class MyCustomText extends Text {
  const MyCustomText(
    super.data, {
    super.key,
  }) : super(
          style: const TextStyle(
            fontWeight: FontWeight.bold,
            fontSize: 30.0,
            fontStyle: FontStyle.italic,
            color: Colors.purple,
          ),
        );

  // superの書き方省略しない場合
  // const MyCustomText(String data, {Key? key})
  //     : super(data,
  //           key: key,
  //           style: const TextStyle(
  //             fontWeight: FontWeight.bold,
  //             fontSize: 30.0,
  //             fontStyle: FontStyle.italic,
  //             color: Colors.purple,
  // ));
}
```

# assert
assertはコンストラクタ内で使用でき、**デバック時にのみ有効**で、条件式がfalseの場合にエラーを発生させる。
リリースビルドではassertは無視されるため、エラーハンドリングやバリデーションには使用できないこと注意。

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'sample',
      home: Scaffold(
        appBar: AppBar(
          title: const Text('sample app'),
        ),
        body: MyCustomContainer(
          // 検証用にnullに変更
          child: null,
          // child: const MyCustomText('Hello, Flutter!'),
        ),
      ),
    );
  }
}

class MyCustomContainer extends Container {
  MyCustomContainer({
    super.key,
    super.child,
  })  
  // assetを追加
  : assert(child != null), // childがnullの場合は_AssertionErrorとなる
        super(
          margin: const EdgeInsets.all(20.0),
          padding: const EdgeInsets.all(20.0),
          color: Colors.blue,
        );
}

class MyCustomText extends Text {
  const MyCustomText(
    super.data, {
    super.key,
  }) : super(
          style: const TextStyle(
            fontWeight: FontWeight.bold,
            fontSize: 30.0,
            fontStyle: FontStyle.italic,
            color: Colors.purple,
          ),
        );
}

```

# 参考
https://dart.dev/language/constructors#invoking-a-non-default-superclass-constructor

https://codewithandrea.com/tips/dart-2.17-super-initializers/#automatic-migration-with-dart-fix

https://dart.dev/language/error-handling#assert