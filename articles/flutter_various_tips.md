---
title: "【Flutter】理解できてそうで理解できてなかったFlutterのあれこれ"
emoji: "🐦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, Dart, late, implements, abstract]
published: true
---
<!-- staticの特徴に関して、良いコードを読んでから記載したい -->

理解したつもりでなんとなく使用していたFlutterのあれこれについて、改めて調べてみた。

```txt :検証環境
// dart --version
Dart SDK version: 2.19.6 (stable) on "macos_arm64"
```


# super
ボリュームが多いため、別記事に分けました。
[【Flutter】コンストラクタとsuperについて](https://zenn.dev/tsukatsuka1783/articles/flutter_constract_super)


&nbsp;
# late
変数宣言時に使用される修飾子。
変数を宣言する際に初期値を与えずに、後で初期化することができる。

**【lateの特徴】**
- 変数の初期化を遅らせることで、柔軟なコードを書くことができる。
- lateを記載することで、後ほど初期化することを明示的に示すことができる。
- lateを記載することで、初期化によって値が入る事が保証されているため、意味の無いnull変数が存在しなくなる。また、nullチェックも不要になる。

**【lateの注意点】**
- 必ず初期化で値を入れる必要あり。(初期化しないとエラーになる)
- lateを使用すると、null安全の恩恵を受けることができない。
- 柔軟性がある一方、使用箇所が多くなると初期化処理箇所を追うのが大変になり、可読性が下がる可能性あり。


**【lateの使用例】**
```dart
class Coffee {
  // late装飾子での変数宣言
  late String _temperature;

  void heat() {
    _temperature = 'hot';
  }

  void chill() {
    _temperature = 'iced';
  }

  String serve() => '$_temperature coffee';
}

main() {
  var coffee = Coffee();
  coffee.heat();
  print(coffee.serve());
}
```
```txt :dart実行結果
hot coffee
```
https://dart.dev/null-safety/understanding-null-safety#late-variables


&nbsp;
# factory
ボリュームが多いため、別記事に分けました。
[【Flutter】factory について](https://zenn.dev/tsukatsuka1783/articles/flutter_factory)



&nbsp;
# static
**【staticの特徴】**
クラスのメソッドや変数に対して使用される装飾子。
クラスのインスタンスを生成せずに直接呼び出すことができる。
また、インスタンスを生成した場合でも、staticなメソッドや変数は共有される。

**【staticの注意点】**
基本的にはstaticメソッドや変数は使用しない方向で考えた方が良いかと。
どこからでも呼び出せるため、意図しない値の変更や、依存関係が不明確になったりするため。
使用する場合は、ログ出力メソッドなど、依存関係が無いorインスタンス化する必要が無いメソッドが有効か。

**【staticメソッド例】**
```dart
class MathUtils {
  static int sum(int a, int b) {
    return a + b;
  }
}

void main() {
  // インスタンスかせずに呼び出せる
  int result = MathUtils.sum(3, 5);
  
  print(result); 
}
```
```txt :dart実行結果
8
```

**【static変数例】**
```dart
class Counter {
  static int count = 0;

  void increment() {
    count++;
  }
}

void main() {
  Counter counter1 = Counter();
  Counter counter2 = Counter();

  // インスタンスを生成してもstatic変数は共有される
  // また、インスタンスを生成しなくても、直接static変数を呼び出せる
  counter1.increment();
  print(Counter.count);

  counter2.increment();
  print(Counter.count);
}
```
```txt :dart実行結果
1
2
```
https://dart.dev/language/classes





&nbsp;
# @override
親クラスのメソッドを上書きするためのアノテーション。
@overrideをつける事で、親クラスのメソッドの振る舞いをカスタマイズしたり、特定の機能を追加したりすることができる。

**【@override例】**
```dart
class Animal {
  void walk() {
    print('動物は歩く');
  }
}

class Cat extends Animal {
  @override
  void walk() {
    print('猫は四足歩行で歩く');
  }
}

void main() {
  var cat = Cat();
  cat.walk();
}
```
```txt :dart実行結果
猫は四足歩行で歩く
```
https://dart.dev/language/extend#overriding-members


&nbsp;
# abstract
クラスやメソッドを抽象化するための装飾子。
抽象(abstract)クラスや抽象(abstract)メソッドを作成することができる。

**※以降、abstractクラスとabstractメソッドを、「抽象クラス」「抽象メソッド」と表記する。**

**【abstractの特徴 ＆ 注意点】**
- 他のクラスに継承されることを意図しているため、抽象クラス自体はインスタンス化できない。抽象クラスを継承した子クラスを作成して使用する必要あり。
- 抽象クラスは、継承された子クラスで機能の追加を提供するために使用される。
  また、共通の機能やメソッドを持つクラスを表現するために使用される。
- 抽象メソッドは、実装の中身が無いメソッドで、継承した子クラスで必ずoverrideして実装する必要がある。
- 抽象クラスは、実装のないメソッド（抽象メソッド）と、具体的なメソッド（具象メソッド）と混在して持つことができる。
- **抽象クラスは、クラス間のコードの再利用や拡張性を高めるために強力な反面、適切な設計や使用がなされないと可読性や保守性が下がる可能性があるので注意。**

**【abstract使用例】**
```dart
// 抽象クラス
abstract class Animal {
  // 抽象メソッド
  void walkingType();

  // 具象メソッド
  void walk() {
    print('動物は歩く');
  }
}

class Cat extends Animal {
  // 抽象メソッドの実装
  // 継承した親クラスに抽象メソッドがある場合は、必ずoverrideして実装する必要あり。
  @override
  void walkingType() {
    print('猫は四足歩行');
  }
}

void main() {
  var cat = Cat();
  cat.walkingType();
  cat.walk();
}
```
```txt :dart実行結果
猫は四足歩行
動物は歩く
```
https://dart.dev/language#interfaces-and-abstract-classes


&nbsp;
# implements
特定のインターフェースを実装する際に使用される。

&nbsp;
**【注意点】**
**「インターフェース」と「抽象クラス」との違い（概念の違い？）を知ることが一つポイント。**
**ややこしいことに、Dartのコード上ではどちらも「abstract class」で表現されるが、目的や使用方法が違うことに注意。**


&nbsp;
**【インターフェース】**
インターフェースは、クラスがどのようなメソッドやプロパティを提供するべきかを定義したもの。
インターフェース定義内のメソッドやプロパティは、具体的な実装を一切含んでいない。（抽象メソッドなどの宣言のみで構成される）
インターフェースを実装するクラスは、インターフェースで宣言されたメソッドやプロパティを全て使用する必要がある。
```dart
// インターフェース
abstract class Animal {
  // 具体的な実装を一切含まない（ここでは抽象メソッドのみの記載）
  void walkingType();
  void walk();
  void sleep();
}

// インターフェースの実装
class Dog implements Animal {
  // インターフェースで定義されているメソッドを全て実装しないとErrorとなる。
  @override
  void walkingType() {
    print('四足歩行');
  }

  @override
  void walk() {
    print('歩く');
  }


  @override
  void sleep() {
    print('Zzz...');
  }

  // インターフェースで定義されていないものは、自由に追加可能。
  void makeSound() {
    print('Woof!');
  }
}

```

&nbsp;
**【抽象クラス】**
抽象クラスは、上記見出しの「**abstract**」で説明した通り、実装のないメソッド（抽象メソッド）と、具体的なメソッド（具象メソッド）と混在して持つことができる。
```dart
// 抽象クラス
abstract class Animal {
  // 具象と抽象の両方のメソッドを持つことができる

  // 抽象メソッド
  void walkingType();

  // 具象メソッド
  void walk() {
    print('動物は歩く');
  }

  // 具象メソッド
  void sleep() {
    print('動物は眠る');
  }
}

// 継承
class Dog extends Animal {
  // 抽象クラスを継承した際は、抽象メソッドだけは必ずオーバーライドする必要あり。
  @override
  void walkingType() {
    print('四足歩行');
  }

  // 抽象クラスで定義されていないものは、自由に追加可能。
  void makeSound() {
    print('Woof!');
  }
}
```


&nbsp;
# .. （カスケード演算子）
同じオブジェクトに対して一連の操作を行うことができる演算子。
一時的な変数を作成せずにアクセスや呼び出しができ、コードを簡潔に記述することができる。


```dart
class Cat {
  late String name;
  late int age;

  void introduction() {
    print('$name、$age歳');
  }

  void type() {
    print('ミケ猫');
  }

  void meal() {
    print('チュール');
  }
}

void main() {
  Cat cat = Cat()
    ..name = 'tom'
    ..age = 5;

  cat
    ..introduction()
    ..type()
    ..meal();
}
```
```txt :dart実行結果
tom、5歳
ミケ猫
チュール
```
https://dart.dev/language/operators#cascade-notation


<!-- # 無名関数 -->
