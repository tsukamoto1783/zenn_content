---
title: "【Flutter】factory について"
emoji: "🏭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, Dart, factory, シングルトン, ]
published: true
---
# factory
クラスのコンストラクタ定義の際に使用される修飾子。
通常のコンストラクタは常に新しいオブジェクトを生成するが、factoryコンストラクタは新しいオブジェクトを生成せずに別のオブジェクトを返したり、既存のオブジェクトを再利用したりできる。

Dartのドキュメントには以下のように記載されている。

https://dart.dev/language/constructors#factory-constructors

> Use the factory keyword when implementing a constructor that doesn’t always create a new instance of its class. 
For example, a factory constructor might return an instance from a cache, or it might return an instance of a subtype.
Another use case for factory constructors is initializing a final variable using logic that can’t be handled in the initializer list.

> ファクトリーキーワードは、常にそのクラスの新しいインスタンスを作成するわけではないコンストラクタを実装する場合に使用します。
たとえば、**ファクトリーコンストラクタはキャッシュからインスタンスを返したり**、**サブタイプのインスタンスを返したり**することがあります。
ファクトリーコンストラクタのもう一つの使用例は、**イニシャライザーリストでは処理できないロジックを使用して最終変数を初期化する場合**です。

&nbsp;
ドキュメントの記載を元に、factoryコンストラクタの使用例を見ていく。
:::message
- シングルトン
- キャッシュからインスタンスを返却（再利用）
- サブタイプのインスタンス返却
- イニシャライザーリストでは処理できないロジックを使用してfinal変数を初期化する場合
:::

&nbsp;
※コンストラクタについては以下の記事参照
[【Flutter】コンストラクタとsuperについて](https://zenn.dev/tsukatsuka1783/articles/flutter_constract_super)

&nbsp;
```txt :検証環境
// dart --version
Dart SDK version: 2.19.6 (stable) on "macos_arm64"
```

&nbsp;
# シングルトン
factoryコンストラクタはシングルトンを実装する際に使用されることが多い。

シングルトンとは、**アプリケーション内で唯一のインスタンスを生成・保持するクラス**のこと。
factoryコンストラクタを使用することで、常に同じインスタンスを返すようにすることができる。

**【特徴】**
- グローバルな状態の管理・共有
必要なデータや機能にグローバルに簡単にアクセスでき、また、変更も簡単に適用できる。
- リソースの共有・効率化
唯一のインスタンスを保持してるので、「リソースの共有・パフォーマンスの向上・メモリの節約」などが可能に。特にDBへの接続やログの出力など、重い処理を必要とする場合に有効。
- データの一貫性・整合性
- etc.

**【注意点】**
シングルトンが有効な場合にのみ適用することが大事。
シングルトンはグローバルな状態を保持するので、逆に複雑さを増す可能性があるため。

※ "シングルトンが有効な場合"とは、「アプリケーション全体で1つのインスタンスのみが必要で、複数のインスタンスを作成しても意味がない場合。」
例えば、アプリケーション全体で共有する設定ファイルやログファイルなど。
アプリケーションのさまざまな部分で異なる動作を必要とするオブジェクトがある場合は、シングルトンパターンは適さない。

&nbsp;
```dart :シングルトンのサンプルコード
class Singleton {
  // シングルトンインスタンスを保持する静的な変数
  static final Singleton _instance = Singleton._internal();

  // インスタンスを取得するためのファクトリコンストラクタ
  // 常に同じインスタンスを返す
  factory Singleton() {
    return _instance;
  }

  // インスタンス生成時に使用されるプライベートな名前付きコンストラクタ
  Singleton._internal() {
    print('Singletonクラスのインスタンスが生成されました');
  }
}

void main() {
  var instance1 = Singleton();
  var instance2 = Singleton();

  // 同じインスタンスかどうかを確認
  print(identical(instance1, instance2));
}
```
```txt
Singletonクラスのインスタンスが生成されました
true
```

&nbsp;
# キャッシュからインスタンスを返却（再利用）
factoryコンストラクタは以前に作成したオブジェクトを返すことができる。これによってオブジェクトの再利用が可能となる。
重いリソースを必要とするオブジェクトがある場合や、同一のオブジェクトを何度も生成したくない場合になどに有効。
（ここは直感的に利点が分かりやすいかと思います）


```dart :再利用パターンのサンプルコード
class Document {
  // インスタンスをキャッシュするためのマップを作成
  static final Map<String, Document> _cache = <String, Document>{};

  // Document クラスのプロパティ
  final String id;
  String content;

  // ファクトリコンストラクタ定義。同じidを持つDocumentが既に存在する場合はそれを返し、
  // 存在しない場合は新たに Document を作成してキャッシュに保存する
  factory Document(String id, [String content = '']) {
    if (_cache.containsKey(id)) {
      return _cache[id]!;
    } else {
      final Document doc = Document._internal(id, content);
      _cache[id] = doc;
      return doc;
    }
  }

  // プライベートな名前付きコンストラクタ
  Document._internal(this.id, this.content) {
    print('Document インスタンスを作成しました。id: $id, content: $content');
  }
}

void main() {
  // インスタンス作成
  var doc1 = Document('doc1', 'Hello');
  // 同じidを持つインスタンスを作成。すでに存在するためキャッシュから取得される
  var doc2 = Document('doc1');

  // 同じインスタンスなので、同じ出力結果となる
  print(doc1.content);
  print(doc2.content);

  // doc2.contentを更新。これによりdoc1.contentも更新される
  doc2.content = 'Hello, world!';

  // 同じインスタンスなので、どちらも更新された値が出力される
  print(doc1.content);
  print(doc2.content);
}
```
```txt
Document インスタンスを作成しました。id: doc1, content: Hello
Hello
Hello
Hello, world!
Hello, world!
```


&nbsp;
# サブタイプのインスタンス返却
factoryコンストラクタは、スーバークラス（親クラス）のコンストラクタが、サブクラス（子クラス）のインスタンスを返すことができる。

このパターンでは、クライアントコード側ではサブタイプなどを気にせずに、適切なサブクラスのインスタンスを返すことができる。
（サブクラスのインスタンスを返すロジックは、スーパークラスのfactoryコンストラクタ部分が担うので、クライアントコード側でロジックを考える必要が無い。）

また、コード拡張時にも、スーパークラスのコンストラクタを変更するだけで済むため、クライアントコード側に影響が少ない。

```dart :サブタイプのインスタンス返却のサンプルコード
class Animal {
  final String name;

  Animal(this.name);

  factory Animal.fromJson(Map<String, dynamic> json) {
    // サブクラスを選択するロジックを記載
    if (json['type'] == 'cat') {
      print('Catインスタンスが生成されます');
      return Cat(json['name']);
    } else if (json['type'] == 'dog') {
      print('Dogインスタンスが生成されます');
      return Dog(json['name']);
    } else {
      throw Exception('Unknown animal type');
    }
  }
}

class Cat extends Animal {
  Cat(super.name);
}

class Dog extends Animal {
  Dog(super.name);
}

void main() {
  final animal = Animal.fromJson({'type': 'cat', 'name': 'タマ'});
  print(animal.name);
}
```
```txt
Catインスタンスが生成されます
タマ
```


&nbsp;
# イニシャライザーリストでは処理できないロジックを使用してfinal変数を初期化する場合
factoryコンストラクタだと、オブジェクトの初期化ロジックをより細かく制御することができる。

イニシャライザーリスト[*1]では、コンストラクターが呼び出されたときにすぐに実行される一連の式のみを実行できる。
一方、ファクトリコンストラクターでは、任意のロジックを実行できる。

言葉だけだとよく分からないので、サンプルコードで確認する。

&nbsp;
まずは通常のコンストラクタの場合。
```dart :通常のコンストラクタ①
class Initialization {
  final String initializedValue;

  Initialization(this.initializedValue) {
    print("インスタンスが生成されました");
  }
}

void main() {
  // インスタンス生成時に引数が必要
  var instance = Initialization('初期化する値');
}
```
```txt
インスタンスが生成されました
```


&nbsp;
```dart :通常のコンストラクタ②
class Initialization {
  final String initializedValue;

  // デフォルト値を設定しているので、引数指定しなくても実行可能
  Initialization([this.initializedValue = '']) {
    print("インスタンス生成されました");
  }
}

void main() {
  // インスタンス生成時に引数しなくても実行可能
  var instance = Initialization();
}
```
```txt
インスタンスが生成されました
```

&nbsp;
イニシャライザーリストに動的な値を入れようとするとエラーになる。
```dart : Errorになる例
class Initialization {
  final String initializedValue;

  // 以下のように動的にデフォルト値を設定はできない。
  // Error:The default value of an optional parameter must be constant.
  Initialization([this.initializedValue = DateTime.now().toString()]) {
    print("インスタンス生成されました");
  }
}

void main() {
  var instance = Initialization(); // NG
}
```

&nbsp;
以下のようにfactoryコンストラクタを使用すると、動的な値を設定することができる。
```dart :factoryコンストラクタ例
class Initialization {
  final String initializedValue;

  // factoryコンストラクタを使用することで、イニシャライザーリストでは処理できないロジックを実行して初期化できる
  factory Initialization() {
    // ここでは例として現在の日時を使用して初期化する
    final String currentDateTime = DateTime.now().toString();
    final String value = '初期化された時間：$currentDateTime';

    return Initialization._(value);
  }

  Initialization._(this.initializedValue) {
    print('インスタンスが生成されました');
  }
}

void main() {
  var instance = Initialization();
  print(instance.initializedValue);
}
```
```txt
インスタンスが生成されました
初期化された時間：2023-05-10 18:42:28.725200
```

&nbsp;
&nbsp;
[*1]: イニシャライザーリストは、以下の部分のことです。
```dart
class Person {
  final String name;
  final int age;

  // (this.name, this.age) 部分がイニシャライザーリスト
  Person(this.name, this.age);
}
```
&nbsp;
## 【補足】 late finalで初期化する場合との違いは？
factoryコンストラクタで初期化する場合と、late finalで初期化する場合との違いはざっくりと以下。

&nbsp;
**【初期化タイミング】**
factoryコンストラクタ： インスタンス生成時に初期化される。

late final： 初期化はインスタンス生成時でなくても良くて、初めてlate final変数にアクセスした時に初期化される。

&nbsp;
**【コードの記載】**
factoryコンストラクタ： コードは少し複雑になるが、より複雑な初期化ロジックを実装できたり、汎用性があるロジック（コンストラクタ）にできる。

late final： コードは簡潔になるが、比較的単純な初期化処理がとなり、汎用性はあまり期待できない。

&nbsp;
以下にそれぞれのサンプルコードを記載する。
（サンプルコードだと同じ結果となり、あまり違いはないように見えるが、上記の違いがあることを頭に入れておくと、いつか役にたつことがあるはず。。）

```dart :factoryコンストラクタ例
class Initialization {
  final String initializedValue;

  factory Initialization() {
    final String currentDateTime = DateTime.now().toString();
    final String value = '初期化された時間：$currentDateTime';

    return Initialization._(value);
  }

  // 初期化処理
  Initialization._(this.initializedValue) {
    print('インスタンスが生成されました');
  }
}

void main() {
  Initialization instance = Initialization();
  print(instance.initializedValue);
}
```
```dart :late final例
class Initialization {
  late final String initializedValue;

  Initialization() {
    print('インスタンスが生成されました');

    final String currentDateTime = DateTime.now().toString();

    // 初期化処理
    initializedValue = '初期化された時間：$currentDateTime';
  }
}

void main() {
  Initialization instance = Initialization();
  print(instance.initializedValue);
}
```
```
インスタンスが生成されました
初期化された時間：2023-05-10 18:44:51.807500
```

&nbsp;
# おわり
Factoryを用いたデザインパターンは、Dartなどの言語に限らずオブジェクト指向言語でよく使われきたパターンです。
歴史があって調べ出すと奥の深い話なので、今後も実践しながら学んでいきたいです📚