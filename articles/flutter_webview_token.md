---
title: "【Flutter】Webviewに認証情報を渡す"
emoji: "🔑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, dart, webview, token, cookie]
published: true
publication_name: ncdc
---

[webview_flutter](https://pub.dev/packages/webview_flutter) で認証情報を付与して Web ページを表示したく、色々と調査してみました。

::: message
本記事は webview_flutter: **4.10.0** 時点での記載です。
:::

<br>

## 前提

[webview_flutter](https://pub.dev/packages/webview_flutter) の基本的な使い方については記載しません。
他の有益な記事が沢山ございますので、基本的な使用方法についてはそちらを参照ください。

【検証環境】

```txt
flutter doctor
[✓] Flutter (Channel stable, 3.22.2, on macOS 14.0 23A344 darwin-arm64, locale ja-JP)
[✓] Android toolchain - develop for Android devices (Android SDK version 33.0.2)
[✓] Xcode - develop for iOS and macOS (Xcode 15.2)
[✓] Chrome - develop for the web
[✓] Android Studio (version 2024.1)
[✓] VS Code (version 1.95.1)
[✓] Connected device (6 available)
[✓] Network resources
• No issues found!
```

<br>

## headers

```dart
controller = WebViewController()
  ...loadRequest(Uri.parse('https://flutter.dev'), headers: {});
```

`WebViewController.loadRequest` のオプショナルの引数で`headers`プロパティが存在するが、android には対応していないと Readme に記載あり。
なので、クロスプラットフォームである以上`headers`は実質使えない。
（回避策もありそうだが、できれば platform 毎に分けたくない。）

[Readme: Setting custom headers on POST requests](https://pub.dev/packages/webview_flutter#setting-custom-headers-on-post-requests)

> Currently, setting custom headers when making a post request with the WebViewController's loadRequest method is not supported on Android. If you require this functionality, a workaround is to make the request manually, and then load the response data using loadHtmlString instead.

<br>

## window.localStorage

[WebViewController](https://pub.dev/documentation/webview_flutter/latest/webview_flutter/WebViewController-class.html)の`runJavascript`や`runJavascriptReturningResult`を使用すれば webview 上で javascript を実行することができる。

ブラウザの localStorage [^1]に認証情報を保存し、webview で表示する際に `localStorage` から認証情報を取得する方法が考えられる。

```dart
final accessToken = "your_access_token";
// ...省略

final controller = WebViewController();
await controller.setJavaScriptMode(JavaScriptMode.unrestricted);
await controller.loadRequest("url");

// 1.アクセストークンをlocalStorageに保存
// 2.localStorageからアクセストークンを取得し、確認用でダイアログで表示
controller.runJavaScript("""
  window.localStorage.setItem('access_token', '$accessToken');
  const token = window.localStorage.getItem('access_token');
  window.confirm('取得したアクセストークン: ' + token);
""");
```

上記の設定で、**android** で webview を表示すると、ダイアログが表示されて認証情報が localStorage に保存されていることが確認できる。

![](https://storage.googleapis.com/zenn-user-upload/3ecf6832e89e-20241107.png =300x)

ただ、**ios で実行すると、javascript が実行されずに終了**してしまう。（上記のようなモーダルが出ない。）

<br>

### 確認１

Issue: [[WebView] Provide webview_flutter a way to enable localStorage](https://github.com/flutter/flutter/issues/146274) より、`onPageFinished`の中で実行するようにしてみたりしたが、特に変化なく ios だと javascript が実行されない。

:::: details memo
以下、android だと意図通り javascript が実行されるが、ios だと実行されない。

```dart
final isSetToken = useState(false);
final accessToken = "your_access_token";
// ...省略

final controller = WebViewController();
await controller.setJavaScriptMode(JavaScriptMode.unrestricted);
await controller.loadRequest("url");
await controller.setNavigationDelegate(
    NavigationDelegate(
    onPageFinished: (url) {
        if (isSetToken.value) return;

        // 1.アクセストークンをlocalStorageに保存
        // 2.localStorageからアクセストークンを取得し、確認用でダイアログで表示
        controller.runJavaScript("""
          window.localStorage.setItem('access_token', '$accessToken');
          const token = window.localStorage.getItem('access_token');
          window.confirm('取得したアクセストークン: ' + token);
        """);

        // 試しに runJavaScriptReturningResult で実行してみても結果は変わらず。
        Future.delayed(
        const Duration(milliseconds: 10),() async {
        final result = await controller.runJavaScriptReturningResult("""
          window.localStorage.setItem('access_token', '$accessToken');
          const token = window.localStorage.getItem('access_token');
          window.confirm('取得したアクセストークン: ' + token);
          """);
        print('webview javascript result: $result');


        // localStorageにトークンをセットした後、ページを再読み込みする。
        isSetToken.value = true;
        // await controller.reload(); // ← reload関数もあるが、適切に制御しないと無限ループになる可能性があるため注意。
    },
));
```

::::

### 確認２

Issue の[[webview_flutter] Can not invoke Javascript function on iOS](https://github.com/flutter/flutter/issues/128489)より、なにやら iOS だと javascript の実行ができないやらなんやらと記載がある。

が、解消せず。

### 確認３

ドキュメント Readme の[Platform-Specific Features](https://pub.dev/packages/webview_flutter#platform-specific-features)セクションの内容や、上記の Issue[[WebView] Provide webview_flutter a way to enable localStorage](https://github.com/flutter/flutter/issues/146274) にも記載ある Platform 固有の設定を適切に設定したら ios でも javascript が実行されるかもしれない。

が、未検証。（できるだけ固有の設定をしたくなかっため。）

### 確認４

ios 側の権限設定が足りないのか？と検索して出てきたそれっぽい権限（NSAllowsArbitraryLoads や NSAllowsArbitraryLoadsInWebContent）を info.plist に追加してみたが、改善なし。
[参考 Issue:](https://github.com/flutter/flutter/issues/122890)
<br>

## cookie

Webview で表示する際に cookie に証情情報を付与してみる。

[webview_flutter](https://pub.dev/packages/webview_flutter) で定義されている`WebviewCookie`と`WebViewCookieManager`を使用。

```dart
final accessToken = "your_access_token";
// ...省略

final controller = WebViewController();
await controller.setJavaScriptMode(JavaScriptMode.unrestricted);
await controller.loadRequest("url");

// cookie設定
final cookie = WebViewCookie(
  name: 'access_token',
  value: accessToken ?? '',
  domain: "<domain URL>", // 設定するURLに注意
);
await WebViewCookieManager().setCookie(cookie);
```

これだけで cookie の設定が完了。
ただ、`WebViewCookieManager`では、`setCookie()`と`clearCookies()` ぐらいしか関数が提供されていないため、本当に設定されたかどうかが確認できない。

`.runJavaScript`で確認しようにも ios だと実行されないので。
（android なら javascript の[document.cookie](https://developer.mozilla.org/ja/docs/Web/API/Document/cookie)で取得確認可能。）

今回は確認用に[webview_cookie_manager](https://pub.dev/packages/webview_cookie_manager) を使用して、設定した cookie を取得確認してみる。
webview_cookie_manager には cookie を扱うための関数がいくつか提供されている。

```dart
final controller = WebViewController();
await controller.setJavaScriptMode(JavaScriptMode.unrestricted);
await controller.loadRequest("url");

// cookie設定
final cookie = WebViewCookie(
  name: 'access_token',
  value: accessToken ?? '',
  domain: "<domain URL>",
);
await WebViewCookieManager().setCookie(cookie);

await controller.setNavigationDelegate(
  NavigationDelegate(
    onPageFinished: (url) async {
      // 以下確認用
      final cookies = await WebviewCookieManager().getCookies(url);
      print('get cookies: $cookies');
    },
  ),
);
```

上記だと`onPageFinished`時に cookie が取得できて出力されることを確認できる。

```txt
get cookies: [access_token=your_access_token; Domain=flutter.dev; Path=/, ...省略]
```

この方法だと Platform 毎のややこしい設定しなくても認証情報を webview に渡すことができる。

::: message
`webview_cookie_manager`で定義されているのは `WebviewCookieManager()` 。
`webview_flutter`で定義されているのは `WebViewCookieManager()` 。
`V`の大文字小文字が違うだけとややこしいので注意。
:::

::: message
webview_cookie_manager は 3 年以上更新が無いライブラリであること注意。今回使いたい関数については正常に動いている & 確認用のため使用。
:::

<br>

[^1]:
    参照：[「Local Storage とは？」](https://webliker.info/web-skill/how-to-use-localstrage/)、[「Window: localStorage プロパティ」
    ](https://developer.mozilla.org/ja/docs/Web/API/Window/localStorage)
