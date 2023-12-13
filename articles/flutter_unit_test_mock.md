---
title: "【Flutter】mockitoを使用したユニットテストについて学ぶ"
emoji: "😼"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter,dart, unitTest, mock, test]
published: true
publication_name: ncdc
---
本記事では、mockitoパッケージを使用したユニットテストについて記載する。

モックを使用しない基本的なユニットテストについては、下記記事に記載しております。
基本的なユニットテストについて確認したい方は、こちらぜひご参照ください。
https://zenn.dev/ncdc/articles/flutter_unit_test

&nbsp;
&nbsp;

# 公式のチュートリアル
https://docs.flutter.dev/cookbook/testing/unit/mocking

mockitoを使ったユニットテストについて、上記の公式チュートリアルを基に確認していく。

上記のチュートリアルでは、以下のサンプルコードで、mockitoを使用したユニットテストの挙動が確認できる。

::::details チュートリアルのサンプルコード（コメント詳細に追加 Ver）
```dart :main.dart
import 'dart:async';
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

// アルバムデータを取得する関数
Future<Album> fetchAlbum(http.Client client) async {
  // http.Clientを使って、外部APIにアクセスする。
  // ここではid=1のアルバムデータを取得する。
  final response = await client
      .get(Uri.parse('https://jsonplaceholder.typicode.com/albums/1'));

  if (response.statusCode == 200) {
    // サーバーから"200 OK"が返ってきた場合、Albumクラスのインスタンスを返す。
    return Album.fromJson(jsonDecode(response.body));
  } else {
    // サーバーから"200 OK"が返ってこなかった場合、例外を投げる。
    throw Exception('Failed to load album');
  }
}

class Album {
  final int userId;
  final int id;
  final String title;

  // 通常のコンストラクタ
  const Album({required this.userId, required this.id, required this.title});

　// ファクトリコンストラクタ
　factory Album.fromJson(Map<String, dynamic> json) {
    return Album(
      userId: json['userId'],
      id: json['id'],
      title: json['title'],
    );
  }
}

// APIから取得したアルバムデータを表示するだけの画面
void main() => runApp(const MyApp());

class MyApp extends StatefulWidget {
  const MyApp({super.key});

  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  late final Future<Album> futureAlbum;

  @override
  void initState() {
    super.initState();
    futureAlbum = fetchAlbum(http.Client());
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Fetch Data Example',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Fetch Data Example'),
        ),
        body: Center(
          child: FutureBuilder<Album>(
            future: futureAlbum,
            builder: (context, snapshot) {
              if (snapshot.hasData) {
                return Text(snapshot.data!.title);
              } else if (snapshot.hasError) {
                return Text('${snapshot.error}');
              }
              return const CircularProgressIndicator();
            },
          ),
        ),
      ),
    );
  }
}
```
```dart :fetch_album_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:http/http.dart' as http;
import 'package:flutter_test_sample/main.dart';
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';

import 'fetch_album_test.mocks.dart';

// Mockitoパッケージを使用して、MockClientを自動生成する。
@GenerateMocks([http.Client])
void main() {
  group('fetchAlbum', () {
    test('httpの呼出が正常に成功した場合、取得データを返却する', () async {
      // MockClientインスタンスの作成(モックオブジェクト)
      final client = MockClient();

      // client.getが呼び出されたときに、成功したレスポンスを返すように設定。
      // clientは、http.Clientをモックした、MockClientインスタンスであるため、.getをしても実際のAPIアクセスは行わず、デフォルトだとnullを返す。
      // そこで、when().thenAnswer()で、指定の戻り値を返すように設定する。（スタブ化）
      when(client
              .get(Uri.parse('https://jsonplaceholder.typicode.com/albums/1')))
          .thenAnswer((_) async =>
              http.Response('{"userId": 1, "id": 2, "title": "mock"}', 200));

      // fetchAlbum関数内の、client.get部分は、上記で設定した振る舞い通りとなる。
      // fetchAlbum関数を実行し、戻り値がAlbum型のインスタンスであることを確認する
      expect(await fetchAlbum(client), isA<Album>());
    });

    test('httpの呼び出しがエラーで完了した場合、例外を投げる', () {
      final client = MockClient();

      // MockClientインスタンスを使用し、.getをすると失敗したレスポンスを返すように設定。
      when(client
              .get(Uri.parse('https://jsonplaceholder.typicode.com/albums/1')))
          .thenAnswer((_) async => http.Response('Not Found', 404));

      expect(fetchAlbum(client), throwsException);
    });
  });
}
```
::::

&nbsp;
&nbsp;
&nbsp;

# mockito
チュートリアルのサンプルコードを動かして「はい、おしまい。」では、理解が浅く、味気ないので、次はmockitoのドキュメントを基に、サンプルコードを作って実際に動かしながら確認していく。

挙動確認用のサンプルコードでは、ドキュメントに記載のあるCatモデルを使用する。

※Catモデルだと、モック化する旨みは無いが、挙動確認用のサンプルコードなのであしからず。

:::message
記事作成時のmockitoバージョンは`5.4.0`です。
:::

https://pub.dev/packages/mockito

```dart
@GenerateNiceMocks([MockSpec<Cat>()])
import 'cat.mocks.dart';

class Cat {
  String sound() => "Meow";
  bool eatFood(String food, {bool? hungry}) => true;
  Future<void> chew() async => print("Chewing...");
  int walk(List<String> places) => 7;
  void sleep() {}
  void hunt(String place, String prey) {}
  int lives = 9;
}
```

:::message
Topics
- when()
- verify()
- 引数matchers
- アノテーション
:::

&nbsp;
&nbsp;
&nbsp;

# when()
when()を使用する事で、モックインスタンスのメソッドの戻り値の設定ができる（スタブ化）。
基本的にはwhen()単体ではなく、他のメソッドと組み合わせて使用する。

:::message
モックインスタンスのメソッドに対して、when()の使用がない、またはwhen()単体の指定だと、デフォルトの戻り値が返ってくることなるので注意。
:::

```dart
test('when test', () {
    // スタブ化せずに呼び出すと、デフォルト値が返される
    expect(cat.sound(), equals("")); // test OK
});
```

### thenReturn:
特定のメソッド呼び出しに対して、固定値を返す。
```dart
test('when.thenReturn test', () {
    when(cat.sound()).thenReturn("Hoge");

    expect(cat.sound(), equals("Hoge")); // test OK
});
```

### thenThrow:
特定のメソッド呼び出しに対して、例外をスローする。
```dart
test('when.thenThrow test', () {
    when(cat.lives).thenThrow(RangeError('Boo'));
    
    expect(() => cat.lives, throwsRangeError); // test OK
});
```

### thenAnswer:
特定のメソッド呼び出しに対して、非同期処理や、引数による動的な戻り値など、より柔軟な設定ができる。
単に固定値を返すならthenReturn()、動的に変化する値を返すならthenAnswer()、といった具合の使い分け。

```dart
test('when.thenAnswer test', () {
    var responses = ["Purr", "Meow"];
    when(cat.sound()).thenAnswer((_) => responses.removeAt(0));

    expect(cat.sound(), "Purr");
    expect(cat.sound(), "Meow");
});
```
&nbsp;
&nbsp;
&nbsp;

# verify
モックインスタンスで呼び出されたメソッドが、期待通りに呼び出されているかを確認するメソッド。
DBの書き込みが正しい回数で行われているか、API呼び出しが適切な順序で行われているか、などを確認できる。

:::message
[verify top-level property](https://pub.dev/documentation/mockito/latest/mockito/verify.html) より引用
> Note: When mockito verifies a method call, said call is then excluded from further verifications.
> A single method call cannot be verified from multiple calls to verify, or verifyInOrder.

一度verifyで検証されたメソッドは、以降のverifyやverifyInOrderでは検証対象から除外されるので注意。

```dart
test('Example test', () {
  var mockObject = MockObject();

  // テスト対象の処理を実行
  mockObject.method();

  // 1回目の検証
  verify(mockObject.method()).called(1); // test OK

  // 2回目の検証
  verify(mockObject.method()).called(1); // test NG
});
```
:::

### verify(): 
期待される処理が実行されたかどうかを確認する。
```dart
group('verify', () {
    var cat = MockCat();

    test('verify test', () {
        cat.sound();

        // 指定の処理が呼び出されかどうかを確認
        verify(cat.sound()); // test OK
    });

    test('verify.called test', () {
        cat.sound();
        cat.sound();
        cat.sound();

        // 指定の処理が、指定した回数だけ呼び出されかどうかを確認
        verify(cat.sound()).called(3); // test OK
        // verify(cat.sound()).called(greaterThan(3)); // matcherも使用可能
    });
});
```

### verifyInOrder():
 複数の処理が特定の順序で実行されたことを確認する。
 ```dart
test('verifyInOrder test', () async {
　  var cat = MockCat();

    cat.sound();
    cat.sleep();
    await cat.chew();

    // 複数の処理が、特定の順序で実行されたことを確認
    verifyInOrder([cat.sound(), cat.sleep(), cat.chew()]); // test OK
    // verifyInOrder([cat.sound(), cat.chew(), cat.sleep()]); // test NG
});
```

### verifyNever():
 期待される処理が一度も実行されていないことを確認する。
 ```dart
test('verifyNever test', () {
    var cat = MockCat();

    cat.sound();
    cat.sleep();

    // 指定の処理が、一度も呼ばれてないかを確認
    verifyNever(cat.chew()); // test OK
});
```


### verifyZeroInteractions():
 モックオブジェクト内の処理が、一切呼び出されていないことを確認する。
 ```dart
test('verifyZeroInteractions test', () {
　  var cat = MockCat();

    // 指定の処理が、一度も呼ばれていないかを確認
    verifyZeroInteractions(cat); // test OK
});
```
※ groupで共通のモックオブジェクトを使用する場合は注意。
 ```dart
group('verify', () {
    var cat = MockCat();

    test('verify test', () {
        cat.sound();

        verify(cat.sound()); // test OK
    });

    test('verifyZeroInteractions test', () {
        // 他のtest内でcatは使用されているため、テストNGとなる。
        verifyZeroInteractions(cat);
    });
　});
```

### verifyNoMoreInteractions():
モックオブジェクト内の処理が、期待される処理以外に呼び出されていないことを確認する。
```dart
group('verifyNoMoreInteractions verify', () {
    var cat = MockCat();

    test('test1', () {
        cat.sound();
        verify(cat.sound());

        verifyNoMoreInteractions(cat); // test OK
    });

    test('test2', () {
        cat.sound();
        verify(cat.sound());
        cat.sound();

        // verify後にcatが使用されているため、テストNG
        verifyNoMoreInteractions(cat);
    });

    test('test3', () {
        cat.sound();
        cat.sleep();
        verify(cat.sound());

        // cat.sleep()に対するverifyが実行されておらず、catが使用された状態のため、（cat.sleep()が検証対象から除外されていない）テストNG
        verifyNoMoreInteractions(cat);
    });
});
```

&nbsp;
&nbsp;
&nbsp;

# Argument matchers
mockitoには、引数matcherと呼ばれる概念（特殊なメソッド）がある。
引数matcherを使用することで、特定の引数パターンで、スタブ化や検証を行うことができる。
代表的な引数matcherを以下に記載。

### any
引数を任意の値とすることができる。
名前付き引数の場合は、anyNamed()を使用する。
```dart
test('any matchers test', () {
    // 引数が何であっても、eatFoodメソッドがfalseを返すように設定
    when(cat.eatFood(any, hungry: anyNamed('hungry'))).thenReturn(false);

    // さまざまな引数でメソッドを呼び出し、常にfalseが返されることを確認
    expect(cat.eatFood('fish', hungry: true), isFalse);
    expect(cat.eatFood('meat', hungry: false), isFalse);
    expect(cat.eatFood('vegetable'), isFalse);
});
```

### argThat
引数が特定の条件を満たす場合のみ、スタブ化や検証を行うようにする。
```dart
test('argThat matcher', () {
    // 引数foodが'fish'である場合にのみ、trueを返すように設定
    // equalsやcontains等のmatcherを併用することも可能
    when(cat.eatFood(argThat(equals('fish')))).thenReturn(true);

    // 'fish'の引数でメソッドを呼び出し、trueが返されることを確認
    expect(cat.eatFood('fish'), isTrue);

    // 'meat'の引数でメソッドを呼び出しても、スタブ化されていないため初期値であるfalseが返されることを確認
    expect(cat.eatFood('meat'), isFalse);
});
```

### captureAny
任意の引数を記録し、後で検証等を行うことができる。
※詳細のドキュメントが少ない、、
```dart
test('captureAny matchers', () {
    cat.eatFood('fish');
    cat.eatFood('meat');

    expect(verify(cat.eatFood(captureAny)).captured, ["fish", "meat"]);
});
```

&nbsp;
&nbsp;
&nbsp;

# mockitoのアノテーション
ライプラリが提供するアノテーションには、以下の2つがある。
主な違いは、when()で設定されていない（スタブ化されていない）場合の、メソッド呼び出し時の挙動の違い。

### @GenerateNiceMocks:
**※ライブラリにはこちらを推奨と記載あり。**
モックインスタンスのメソッドが、when()で設定されていない（スタブ化されていない）呼び出しに対しても、デフォルトの値（空文字、0、空のリスト、など）を返してくれるようになる。
そのため、モックインスタンスのメソッドの戻り値の設定がされていない場合でも、テストは失敗せずにデフォルト値が返される。

### @GenerateMocks:
モックインスタンスのメソッドが、when()で設定されていない（スタブ化されていない）呼び出しに対して、例外を投げるようになる。
そのため、モックインスタンスのメソッドの戻り値の設定がされていない場合は、テストは失敗となる。

&nbsp;
&nbsp;
&nbsp;

# おわり
mockitoパッケージには他にもメソッドがあるので、より込んだテストを書きたい方は調べてみてください！
