---
title: "【Flutter】スマホのシステム言語設定に対応してWidgetやテキストを多言語化する"
emoji: "🌎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, dart, language, 多言語化]
published: true
publication_name: ncdc
---

Flutter アプリの多言語対応について、動作確認しながら調査してみました。

<br>

:::: details 【flutter doctor】

```yml
[✓] Flutter (Channel stable, 3.22.0, on macOS 14.0 23A344 darwin-arm64, locale ja-JP)
[✓] Android toolchain - develop for Android devices (Android SDK version 33.0.2)
[✓] Xcode - develop for iOS and macOS (Xcode 15.2)
[✓] Chrome - develop for the web
[✓] Android Studio (version 2024.1)
[✓] VS Code (version 1.98.0)
[✓] Connected device (6 available)
[✓] Network resources
```

::::

<br>

## 多言語化対応-1

※ 以下、内部的な挙動確認がメインなので、端的に多言語化対応方法を知りたい方は[「多言語化対応-2」](#多言語化対応-2)からご参照ください。

<br>

デフォルトの Widget だけを多言語化対応する場合を想定してみます。

動作確認用にカレンダーが表示されるだけのシンプルな画面で確認していきます。

![](https://storage.googleapis.com/zenn-user-upload/2456fd175af3-20240711.png =300x)

:::: details 検証用コード

```dart
void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(),
    );
  }
}

class MyHomePage extends StatelessWidget {
  const MyHomePage({super.key});

  @override
  Widget build(BuildContext context) {
    final languageSetting = _getCurrentLanguageSetting(context);

    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: const Text("multi language sample"),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text('現在の言語設定: $languageSetting'),
            const SizedBox(height: 40),

            CalendarDatePicker(
              initialDate: DateTime.now(),
              firstDate: DateTime(1900),
              lastDate: DateTime(2100),
              onDateChanged: (value) {},
            ),

          ],
        ),
      ),
    );
  }
}

/// 現在の言語設定を取得する
String _getCurrentLanguageSetting(BuildContext context) {
  Locale locale = Localizations.localeOf(context);
  String languageCode = locale.languageCode;
  return languageCode;
}

```

::::

**【前提条件】**

- 現在の言語設定の取得は、`Localizations.localeOf(context)` で取得するものとします。
- 基本は[公式ドキュメント](https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization) を参照してます。
- 検証はシミュレーターで動作。
- シミュレーターの言語設定は以下の順番
  1. 日本語
  2. 中国語
  3. 英語
     ![](https://storage.googleapis.com/zenn-user-upload/b1d3accd7aee-20240711.png =300x)

### 1. MaterialApp に何も設定なし。

```dart
 return MaterialApp(
   home: const MyHomePage(),
```

取得される言語設定は `en`。
デバイスの言語設定に関わらず、`en` が取得される。
![](https://storage.googleapis.com/zenn-user-upload/2456fd175af3-20240711.png =250x)

<br>

### 2. MaterialApp に locale を指定。

```dart
 return MaterialApp(
   locale: const Locale('ja'),
   home: const MyHomePage(),
```

local に `ja` を指定してみても、取得される言語設定は `en`。
![](https://storage.googleapis.com/zenn-user-upload/2456fd175af3-20240711.png =250x)

<br>

### 3. MaterialApp に localizationsDelegates を指定

```dart
 return MaterialApp(
    localizationsDelegates: const [
         GlobalMaterialLocalizations.delegate,
         GlobalWidgetsLocalizations.delegate,
         GlobalCupertinoLocalizations.delegate,
    ],
    home: const MyHomePage(),
```

取得できる言語設定は `en`。
![](https://storage.googleapis.com/zenn-user-upload/2456fd175af3-20240711.png =250x)

<br>

### 4. aterialApp に localizationsDelegates と local を指定

```dart
 return MaterialApp(
   locale: const Locale('ja'),
   localizationsDelegates: const [
     GlobalMaterialLocalizations.delegate,
     GlobalWidgetsLocalizations.delegate,
     GlobalCupertinoLocalizations.delegate,
   ],
   home: const MyHomePage(),
```

取得できる言語設定は `en`。
![](https://storage.googleapis.com/zenn-user-upload/2456fd175af3-20240711.png =250x)

<br>

### 5. MaterialApp に supportedLocales だけを指定。

```dart
 return MaterialApp(
    supportedLocales: const [
      Locale('ja'),
      Locale('zh'),
      Locale('en'),
    ],
    home: const MyHomePage(),
```

エラーが発生。delegate が見つかりませんと怒られる。

```txt
════════ Exception caught by widgets ═══════════════════════════════════════════
The following message was thrown:
Warning: This application's locale, ja, is not supported by all of its localization delegates.
• A MaterialLocalizations delegate that supports the ja locale was not found.
• A CupertinoLocalizations delegate that supports the ja locale was not found.
The declared supported locales for this app are: ja, zh, en
See https://flutter.dev/tutorials/internationalization/ for more information about configuring an app's locale, supportedLocales, and localizationsDelegates parameters.
```

<br>

### 6. MaterialApp に supportedLocales と localizationsDelegates を指定

```dart
 return MaterialApp(
    supportedLocales: const [
      Locale('zh'),
      Locale('en'),
      Locale('ja'),  // ← 優先順位的には低いがデバイスの言語設定が ja の場合は ja が優先される
    ],
    localizationsDelegates: const [
      GlobalMaterialLocalizations.delegate,
      GlobalWidgetsLocalizations.delegate,
      GlobalCupertinoLocalizations.delegate,
    ],
    home: const MyHomePage(),
```

supportedLocales にデバイスの言語設定の値が存在するなら、それが優先される。supportedLocales の格納順番に関係なく。

| -                                                                                    | ja                                                                                   |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| ![](https://storage.googleapis.com/zenn-user-upload/b1d3accd7aee-20240711.png =250x) | ![](https://storage.googleapis.com/zenn-user-upload/72b000993d5b-20240711.png =250x) |

デバイスの言語設定の値が supportedLocales に存在しない場合、supportedLocales の最初の言語が優先される。

```dart
return MaterialApp(
    supportedLocales: const [
    Locale('zh'), // ←これが適用される。
    Locale('en'),
    // Locale('ja'),
    ],
    localizationsDelegates: const [
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    GlobalCupertinoLocalizations.delegate,
    ],
    home: const MyHomePage(),
```

| -                                                                                    | zh                                                                                   |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| ![](https://storage.googleapis.com/zenn-user-upload/b1d3accd7aee-20240711.png =250x) | ![](https://storage.googleapis.com/zenn-user-upload/72df22dca1b0-20240711.png =250x) |

<br>

### 7. MaterialApp に supportedLocales と localizationsDelegates と locale を指定

```dart
return MaterialApp(
    locale: const Locale('zh'),
    supportedLocales: const [
       Locale('en'),
       Locale('zh'), // ←zhが適用
       Locale('ja'),
    ],
    localizationsDelegates: const [
       GlobalMaterialLocalizations.delegate,
       GlobalWidgetsLocalizations.delegate,
       GlobalCupertinoLocalizations.delegate,
    ],
    home: const MyHomePage(),
```

supportedLocales に設定のある言語が locale に指定された場合、supportedLocales の格納順番に関係なく locale の値が優先される。

| -                                                                                    | zh                                                                                   |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| ![](https://storage.googleapis.com/zenn-user-upload/b1d3accd7aee-20240711.png =250x) | ![](https://storage.googleapis.com/zenn-user-upload/72df22dca1b0-20240711.png =250x) |

supportedLocales に設定の無い言語が locale に指定された場合は、supportedLocales の格納順一番上の言語が適応される。
※ **この場合はデバイスの言語設定の優先順位が下がる。**

```dart
return MaterialApp(
    locale: const Locale('zh'),
    supportedLocales: const [
        Locale('en'), // ←これが適用
        // Locale('zh'),
        Locale('ja'),
    ],
    localizationsDelegates: const [
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        GlobalCupertinoLocalizations.delegate,
    ],
    home: const MyHomePage(),
```

| -                                                                                    | en                                                                                   |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| ![](https://storage.googleapis.com/zenn-user-upload/b1d3accd7aee-20240711.png =250x) | ![](https://storage.googleapis.com/zenn-user-upload/2456fd175af3-20240711.png =250x) |

<br>

### 8. Widget 個別で言語設定する

`Localizations.override`で Widget をラップしてあげると、その Widget 内だけ言語設定が変更される。

```dart
return MaterialApp(
    locale: const Locale('zh'),
    supportedLocales: const [
        Locale('en'),
        Locale('zh'),
        Locale('ja'),
    ],
    localizationsDelegates: const [
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        GlobalCupertinoLocalizations.delegate,
    ],
    home: const MyHomePage(),

// ...省略

Localizations.override(
    context: context,
    locale: const Locale('vi'), // ベトナム後
    child: Builder(
    builder: (context) {
        return CalendarDatePicker(
        initialDate: DateTime.now(),
        firstDate: DateTime(1900),
        lastDate: DateTime(2100),
        onDateChanged: (value) {},
        );
    },
    ),
),
```

![](https://storage.googleapis.com/zenn-user-upload/5de37469f564-20240711.png =300x)
_↑ カレンダー部分だけベトナム語。_

<br>

## 多言語化対応-2

続いて、コンテンツ（Text）を含めて多言語化対応する場合を想定してみます。

基本的には、上記の「多言語化対応-１」のような Widget だけを多言語化のターゲットにすることは珍しいので、こちらの対応になるかと思います。

対応手順の詳細は[公式ドキュメント](https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization)を参照してください。

### 対応手順

1.  必要なライブラリの追加

    > flutter pub add flutter_localizations --sdk=flutter
    > flutter pub add intl

2.  pubspec.yaml に設定を追加

    ```yaml
    flutter:
      generate: true # Add this line
    ```

3.  多言語化設定ファイルの作成
    プロジェクト配下に、多言語化設定ファイル：`l10n.yaml`を作成し、以下のように設定します。

    ```yaml
    arb-dir: lib/l10n # arbファイルの格納先
    template-arb-file: app_en.arb # テンプレートとなるarbファイル
    output-localization-file: app_localizations.dart # 出力されるファイル名
    preferred-supported-locales: en # デフォルト言語
    ```

4.  テンプレートとなる arb ファイルを作成
    `lib/l10n` 配下に、テンプレートとなる `arb ファイル`（≒ 言語リソースファイル）を作成。

    ```yaml: app_en.arb
    {
      "@@locale": "en",
      "text": "multi language sample",
      "@text": {
        "description": "the text sample"
      }
    }
    ```

5.  テンプレート以外の言語の arb ファイルを作成
    `lib/l10n` 配下に、上記と同じ用に arb ファイルを言語毎に作成。

    ```yaml: app_ja.arb
    {
      "@@locale": "ja",
      "text": "多言語化サンプル",
    }
    ```

    ```yaml: app_ko.arb
    {
      "@@locale": "ko",
      "text": "다국어화 샘플",
    }
    ```

6.  自動生成
    `flutter gen-l10n` コマンドを実行して、設定した各言語化ファイルを自動生成。
    `.dart_tool/flutter_gen/gen_l10n` 配下に生成される。
    ![](https://storage.googleapis.com/zenn-user-upload/5b6d86bf7a4a-20250313.png =300x)

7.  `MaterialApp`のプロパティ設定
    `MaterialApp` のプロパティに `localizationsDelegates` と `supportedLocales` を設定。
    手動で設定でもいいが、自動生成された `app_localizations.dart` を利用することも可。

    ```diff:dart
    return MaterialApp(
    -      supportedLocales: const [
    -        Locale('en'),
    -        Locale('ja'),
    -        Locale('ko'),
    -      ],
    -      localizationsDelegates: const [
    -        AppLocalizations.delegate,
    -        GlobalMaterialLocalizations.delegate,
    -        GlobalWidgetsLocalizations.delegate,
    -        GlobalCupertinoLocalizations.delegate,
    -  ],
    + localizationsDelegates: AppLocalizations.localizationsDelegates,
    + supportedLocales: AppLocalizations.supportedLocales,
    ```

    ※ 自動生成されたファイルの中身を見ると、手動で設定した内容と同じ定義があるので実質は一緒。

8.  Text Widget 部分の対応
    `AppLocalizations.of(context).text` でスマホのシステム言語設定に応じたテキストを取得してくれる。

    ```dart
    Text(AppLocalizations.of(context).text)
    ```

9.  動作確認
    スマホのシステム言語設定によって表示テキストが切り替わることを確認。
    ![](https://storage.googleapis.com/zenn-user-upload/9a39394bd3f3-20250313.gif =300x)

<br>

### 備考

- `l10n.yaml` の設定項目は他にもたくさんあるので、[ドキュメント](https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization#configuring-the-l10n-yaml-file)を元に適宜設定してください。

- 中国語のように、同じ言語でも複数種類（繁体字、簡体字）がある場合、それぞれの`scriptCode`や`countryCode`まで指定することも可。詳細は[ドキュメント参照](https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization#advanced-locale-definition)。
  その場合、適切に命名指定しないと自動生成時にエラーになるので注意。

  例：
  以下の例だと、`Hans`を`hans`にすると怒られる。

  ```yaml
  { "@@locale": "zh_hans", "text": "多语言化示例" }
  ```

  ```txt
  The locale specified in @@locale and the arb filename do not match.
  Please make sure that they match, since this prevents any confusion
  with which locale to use. Otherwise, specify the locale in either the
  filename of the @@locale key only.
  Current @@locale value: zh_hans
  Current filename extension: zh_Hans
  ```

- 「言語コード」参考：https://so-zou.jp/web-app/tech/data/code/language.htm

<br>

## 参照

https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization
https://zenn.dev/aryzae/articles/0555959ac3800e
