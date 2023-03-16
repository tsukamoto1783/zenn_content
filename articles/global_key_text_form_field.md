---
title: "【Flutter】GlobalKeyとTextFormFieldについてのあれこれ"
emoji: "🔑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter,Dart,Widget, GlobalKey, TextFormField]
published: true
---

Flutterにおける入力フォームで、テキストをどのように管理するかのポイント
- GlobalKeyをFormウィジェットに持たせて一元管理
- TextEditingControllerを用いてのTextFormField毎の管理
- TextFormFieldのonChangedで、変数にフォームの値を代入しての管理

それぞれの違いを分かったようで分かってなかったので、今一度まとめてみる。

# GlobalKeyでの管理と、TextEditingControllerでの管理との違いは？
### 【GlobalKey】
- バリデーションを用意に実装できる。
  validatorプロパティや.currentState.validate()メソッドが用意されているため。
- Formウィジェット配下のフォーム全体を制御することになるので、全てのTextFormFieldに対して一括でバリデーション確認等が実行可能。
- ビルドごとに新しいGlobalKeyを作成すると、パフォーマンスやウィジェットの動作に影響が出るので、GlobalKeyの再生成はしない。
  例） Widget build(BuildContext context) の外側でGlobalKeyの定義をする。
- 入力フォームの外から、入力フォーム内のテキストの変更はできない。
- Stateful、Statelessどちらでも可能だが、一般的にはStateful。
  フォームの状態を管理するので。

### 【TextEditingController】
- TextFormField毎の個別の制御のため、入力値の取得や更新が容易。
- TextFormField毎の個別の制御のため、一括でバリデーション実行とかはできない。
- バリデーションに関しては、自作で関数等を作って対応させる必要がある。
- 入力フォームの外から、入力フォームのテキストの変更も可能。
- 基本的にはStatefulのみ。TextEditingControllerは状態の管理が発生する前提のため、Statelessは基本的にアンチパターン。
  →Statelessだとdispose()の概念がなく、Controllerの状態を破棄できないため。
  → 逆に言うと、Controllerの状態を破棄できたらいいので、TextEditingControllerをStatefulな親ウィジェットや、Riverpod（dispose付いたStateNotifier内）で管理するとかなら対応可能。


## そもそもGlobalKeyとは
[GlobalKeyの公式ドキュメント](https://api.flutter.dev/flutter/widgets/GlobalKey-class.html)では、以下のように記載されている。（DeepL翻訳して引用）

:::: details 長いので省略
アプリ全体でユニークなキーです。

グローバルキーは、要素を一意に識別します。グローバルキーは、BuildContextなど、それらの要素に関連する他のオブジェクトへのアクセスを提供します。StatefulWidgetsの場合、グローバル・キーはStateへのアクセスも提供します。

グローバル・キーを持つウィジェットは、ツリー内のある場所からツリー内の別の場所に移動したときに、そのサブツリーを再ペアレントします。サブツリーを再ペアレントするためには、ウィジェットは、ツリー内の古い場所から削除されたのと同じアニメーションフレームでツリー内の新しい場所に到着する必要があります。

グローバルキーを使用して Element を再ペアレントすることは、比較的高価です。この操作は、関連する State とその子孫すべてに対して State.deactivate の呼び出しを引き起こし、InheritedWidget に依存しているすべてのウィジェットを再構築するよう強制するからです。

上記の機能が不要な場合は、代わりにKey、ValueKey、ObjectKey、UniqueKeyを使用することを検討してください。

同じグローバル キーを持つ 2 つのウィジェットを同時にツリーに含めることはできません。これを実行しようとすると、実行時にアサートされます。

落とし穴
GlobalKeysは、ビルドのたびに再作成されるべきではない。通常は、Stateオブジェクトなどが所有する長寿命のオブジェクトにする必要があります。

ビルドのたびに新しい GlobalKey を作成すると、古いキーに関連付けられたサブツリーの状態が破棄され、新しいキーのために新しいサブツリーが作成されます。パフォーマンスを低下させるだけでなく、サブツリー内のウィジェットに予期せぬ動作を引き起こす可能性があります。たとえば、サブツリー内のGestureDetectorは、ビルドごとに再作成されるため、進行中のジェスチャーを追跡することができなくなります。

代わりに、StateオブジェクトにGlobalKeyを持たせ、State.initStateのようにビルドメソッドの外でインスタンス化するのが良い方法です。

こちらも参照してください。

ウィジェットがキーを使用する方法の詳細については、Widget.keyの議論も参照してください。
::::
ちょっと何言ってるのか分からないので、Formウィジェットに関係ある範囲でざっくり要約。


- アプリ全体で一意のKey。
- 特定のWidgetや状態にアクセスするために使用される。（Formウィジェットとか）
- Formウィジェットの場合、フォーム全体の状態を保持し、一括でバリデーションチェックなどの処理を可能にする。
- GlobalKeyによる状態の保持は、同一画面内で複数フォームにまたがるWidget間の遷移がある状況で値を取得・管理するために使用される。（画面遷移した際には状態は破棄される）
<!-- ただし、フォーカスアウトした際に自動的に状態が保存されるわけではなく、フォームのキーを使用して明示的に状態を保存する必要がある。 -->

------------
文章でつらつら書いてもイメージが湧きづらいので、以下のようなシンプルなログイン画面を用いて、実際に動かしながら確認していく。

※コード量を減らすために、Statefulのおまじない部分やAppBar部分は省略して記載してます。
![](https://storage.googleapis.com/zenn-user-upload/3f342d253773-20230316.png =350x)

-------------
# ①：GlobalKey + TextFormFiled

```dart
class _TextFormFieldSampleState extends State<TextFormFieldSample> {
  // Formウィジェットを一意に識別するためのグローバルキーを作成。フォームのバリデーション等を可能に。
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Form(
        key: _formKey,
        child: Column(
          children: [
            TextFormField(
              decoration: const InputDecoration(
                labelText: 'name',
                hintText: 'Enter your name',
              ),
              validator: (value) {
                // _formKey.currentState!.validate()が実行された時に呼び出される
                if (value == null || value.isEmpty) {
                  return 'Please enter some text';
                }
                return null;
              },
              onSaved: (value) {
                // _formKey.currentState?.save()が実行されたときに呼び出される
                print('The saved name is $value');
              },
            ),
            ElevatedButton(
              child: const Text('Submit'),
              onPressed: () {
                // TextFormFieldのvalidatorで指定したバリデーションを確認し、エラーが無ければtrueを返す
                if (_formKey.currentState!.validate()) {
                  // フォームの状態を保存して、TextFormFieldのonSavedを呼び出し
                  _formKey.currentState?.save();
                }
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

# ②：GlobalKey + TextFormFiled
複数のTextFormFieldを用いることで、GlobalKeyがフォーム全体を管理していることがわかりやすい。

```dart
class _TextFormFieldSampleState extends State<TextFormFieldSample> {
  // Formウィジェットを一意に識別するためのグローバルキーを作成。フォームのバリデーションを可能に。
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Form(
        key: _formKey,
        child: Column(
          children: [
            TextFormField(
              decoration: const InputDecoration(
                labelText: 'name',
                hintText: 'Enter your name',
              ),
              validator: (value) {
                // _formKey.currentState!.validate()が実行された時に呼び出される
                if (value == null || value.isEmpty) {
                  return 'Please enter some text';
                }
                return null;
              },
              onSaved: (value) {
                // _formKey.currentState?.save()が実行されたときに呼び出される
                print('The saved name is $value');
              },
            ),
            TextFormField(
              decoration: const InputDecoration(
                labelText: 'email',
                hintText: 'Enter your email',
              ),
              validator: (value) {
                // _formKey.currentState!.validate()が実行された時に呼び出される
                if (value == null || value.isEmpty) {
                  return 'Please enter some text';
                }
                return null;
              },
              onSaved: (value) {
                // _formKey.currentState?.save()が実行されたときに呼び出される
                print('The saved email is $value');
              },
            ),
            ElevatedButton(
              child: const Text('Submit'),
              onPressed: () {
                // Formウィジェットに一意のGlobalKeyを指定しているので、「_formKey.currentState」を使用すると、Form配下の全てのフォームが対象に。
                if (_formKey.currentState!.validate()) {
                  // Form配下の全てのTextFormFieldのonSavedプロパティが対象（呼び出し）
                  _formKey.currentState?.save();
                }
              },
            ),
          ],
        ),
      ),
    );
  }
}

```

# ③：GlobalKey + TextFormFiled
GlobalKeyをTextFormFieldに指定することはできるが、思ったような動作にはならない。
基本的にはGlobalKeyを用いてのフォーム管理はFromウィジェットで使用する。

```dart: 上手く行かない例
class _TextFormFieldSampleState extends State<TextFormFieldSample> {
  // TextFormFieldウィジェットを一意に識別するためのグローバルキーを作成
  final GlobalKey<FormState> _nameKey = GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          TextFormField(
            key: _nameKey,
            decoration: const InputDecoration(
              labelText: 'name',
              hintText: 'Enter your name',
            ),
            validator: (value) {
              if (value == null || value.isEmpty) {
                return 'Please enter some text';
              }
              return null;
            },
            onSaved: (value) {
              print('The saved name is $value');
            },
          ),
          ElevatedButton(
            child: const Text('Submit'),
            onPressed: () {
              // 以下の部分でエラーが発生する
              if (_nameKey.currentState!.validate()) {
                _nameKey.currentState?.save();
                );
              }
            },
          ),
        ],
      ),
    );
  }
}
```


# ④：TextEditingController + TextFormFiled
GlobalKeyを使用せずにTextEditingControllerだけで実装。

```dart
class _TextFormFieldSampleState extends State<TextFormFieldSample> {
  // Controllerの定義
  final TextEditingController _nameController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();

  // ウィジェットが破棄されるタイミングで、Controllerも破棄する。
  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          TextFormField(
            // controllerプロパティの指定
            controller: _nameController,
            decoration: const InputDecoration(
              labelText: 'name',
              hintText: 'Enter your name',
            ),
          ),
          TextFormField(
            // controllerプロパティの指定
            controller: _emailController,
            decoration: const InputDecoration(
              labelText: 'email',
              hintText: 'Enter your email',
            ),
          ),
          ElevatedButton(
            child: const Text('Submit'),
            onPressed: () {
              // 入力フォーム外からテキストの管理可能
              _nameController.text = "test Name";
              _emailController.text = "test Email";
            },
          ),
        ],
      ),
    );
  }
}
```

外部からFormの値を変更できるのがTextEditingControllerの特徴。
```dart:例） onPressedで入力フォーム内のテキストを外部から変更
            ElevatedButton(
              child: const Text('Submit'),
              onPressed: () {
                // 入力フォーム外からテキストの管理可能
                // print(_nameController.text);
                // print(_emailController.text);
                _nameController.text = "test Name";
                _emailController.text = "test Email";
              },
            ),

```
![](https://storage.googleapis.com/zenn-user-upload/f7be2a9186c6-20230316.gif =350x)

# ⑤：TextEditingController + TextFormFiled
バリデーションを指定する際は、自分でメソッド作って呼び出す必要あり。
以下、超簡単なバリデーションを自作した例。

```dart
class _TextFormFieldSampleState extends State<TextFormFieldSample> {
  // Controllerの定義
  final TextEditingController _nameController = TextEditingController();
  // バリデーションエラー表示用の変数定義
  String? _nameErrorText;

  // ウィジェットが破棄されるタイミングで、Controllerも破棄
  @override
  void dispose() {
    _nameController.dispose();
    super.dispose();
  }

  // バリデーションチェック関数
  void _validate() {
    setState(() {
      _nameErrorText = _validateText(_nameController.text);
    });
  }

  String? _validateText(String email) {
    if (email.isEmpty) {
      return 'フォームが空です';
    }
    return null;
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
          children: [
            TextFormField(
              controller: _nameController,
              decoration: InputDecoration(
                labelText: 'name',
                hintText: 'Enter your name',
                errorText: _nameErrorText,
              ),
              onChanged: (_) => _validate(),
            ),
            ElevatedButton(
              child: const Text('Submit'),
              onPressed: () {
                print(_nameController.text);
              },
            ),
          ],
        ),
      );
  }
}
```

# ⑥：TextFormFiledのonChanged
バリデーションとか考えないなら一番シンプルな記述。

```dart
class TextFormFieldSample extends StatelessWidget {
  const TextFormFieldSample({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    String? _formText;

    return Scaffold(
      body: Column(
        children: [
          TextFormField(
            decoration: const InputDecoration(
              labelText: 'name',
              hintText: 'Enter your name',
            ),
            onChanged: (value) {
              _formText = value;
            },
          ),
          ElevatedButton(
            child: const Text('Submit'),
            onPressed: () {
              print(_formText);
            },
          ),
        ],
      ),
    );
  }
}
```
------------

## まとめ
- FormウィジェットでのGlobalKey（onSaved, validator, etc.）
- TextEditingController
- TextFormField（onChanged）

上記、仕様に合うよう組み合わして実装しましょう。

<!-- # フォーカスアウトの実装パターンを何パターンか。 -->

<!-- # パッケージのFormも試してみる。 -->