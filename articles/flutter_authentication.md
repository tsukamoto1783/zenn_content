---
title: "【Flutter】認証周りについてAuth0で色々と確認してみた"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, dart, OAuth2.0, Auth0, flutter_appauth]
published: true
publication_name: ncdc
---

[Auth0](https://auth0.com/jp) + [Postman](https://www.postman.com) + [flutter_appauth](https://pub.dev/packages/flutter_appauth) で OAuth2.0 を色々と確認しながら試してみました。

::: message
動作環境

> flutter doctor
> [✓] Flutter (Channel stable, 3.22.2, on macOS 14.0 23A344 darwin-arm64, locale ja-JP)
> [✓] Android toolchain - develop for Android devices (Android SDK version 33.0.2)
> [✓] Xcode - develop for iOS and macOS (Xcode 15.2)
> [✓] Chrome - develop for the web
> [✓] Android Studio (version 2024.1)
> [✓] VS Code (version 1.95.3)
> [✓] Connected device (6 available)
> [✓] Network resources

:::

<br>

## Auth0 のセットアップ

[Auth0](https://auth0.com/jp) とは、認証と認可のためのプラットフォーム。
有名どこで言えば Firebase Authentication や AWS Cognito などのサービスと同じようなもの。

まずは Auth0 のセットアップ。
小規模の個人利用程度ならば [Free プラン](https://auth0.com/pricing)で最低限の機能は使用可能。

登録が完了したら`Create Application`からアプリケーションを作成。
今回は Flutter で試したいので Native を選択。

![](https://storage.googleapis.com/zenn-user-upload/9191dd34ad9c-20241108.png =500x)

作成が完了すると、自動で`Domain`や`Client ID`などが払い出される。

Auth0 のコンソールで主に確認する箇所は以下。

- `settigns` -> `Basic Information`
- `settigns` -> `Advanced Settings` -> `Endpoints`
  ![](https://storage.googleapis.com/zenn-user-upload/766c173a2c72-20241108.png =500x)
  ![](https://storage.googleapis.com/zenn-user-upload/3d810aa3c754-20241108.png =500x)

<br>

最後に `settings` -> `Application URIs` でコールバック URL を設定する。
今回は Postman デフォルトのコールバック URL `https://oauth.pstmn.io/v1/callback`と、適当な確認用 URL`https://flutter.dev`を設定しておく。

![](https://storage.googleapis.com/zenn-user-upload/9724d196ac23-20241108.png =500x)

<br>

## Postman のセットアップ

次に [Postman](https://www.postman.com) の設定を行う。

Postman を起動して「新しいトークンの設定」の項目を埋めていく。

![](https://storage.googleapis.com/zenn-user-upload/a13f36f22185-20241108.png =500x)

- トークン名
  - 適当な名前を記入
- Grant タイプ
  - ここでは「認可コード」とする。
- コールバック URL
  - Auth0 のコンソールで設定した URL（今回は`https://flutter.dev`）を設定。
  - ※ ここの値がクライアント側と認証サーバー側で一致していないと認証時にエラーとなること注意。
- 認可 URL
  - Auth0 の`OAuth Authorization URL`の値を設定。
- アクセストークン URL
  - Auth0 の`OAuth Token URL`の値を設定。
- クライアント ID
  - Auth0 の`Client ID`の値を設定。
- クライアントシークレット
  - Auth0 の`Client Secret`の値を設定。

上記の設定で Postman から Auth0 が有効かどうか確認してみる。
画面下部のボタンを押下すると、Auth0 の認証画面が表示される。
Auth0 で登録したユーザー情報でログインを行うと、Postman にトークンが返却されることが確認できたら OK。

|                                       1                                        |                                       2                                        |                                       3                                        |
| :----------------------------------------------------------------------------: | :----------------------------------------------------------------------------: | :----------------------------------------------------------------------------: |
| ![](https://storage.googleapis.com/zenn-user-upload/02e6a5c775a7-20241108.png) | ![](https://storage.googleapis.com/zenn-user-upload/e97af1ee56d3-20241108.png) | ![](https://storage.googleapis.com/zenn-user-upload/e52011039003-20241108.png) |

<br>

## flutter_appauth セットアップ

::: message
検証時の[flutter_appauth](https://pub.dev/packages/flutter_appauth)のバージョンは`8.0.1`。

:::

::: message
auth0 用のライブラリ[auth0_flutter](https://pub.dev/packages/auth0_flutter)も存在するが今回は不使用。
Auth0 に限らず、他の認証プロバイダーも使用する想定なため、汎用的なライブラリで動作確認します。
:::

Flutter アプリで適当にボタンだけ配置。
そして認証に必要な情報を定義する。

Auto0 のコンソールで設定したコールバック URL `flutter.dev`を設定し、ログインが完了して`flutter.dev`が表示されたら OK。

```dart
const String clientId = '<Client ID>';
const String clientSecret ='<Client Secret>';
const String authorizationEndpoint ='<OAuth Authorization URL>';
const String tokenEndpoint ='<OAuth Token URL>';
const String redirectUri = 'https://flutter.dev'; // Auto0のコンソールで設定したコールバック URL

// ...省略
ElevatedButton(
    onPressed: () async {
    final result = await _login();
    },
    child: const Text('auth0 login'),
),

// ...省略
Future _login() async {
  try {
    const FlutterAppAuth appAuth = FlutterAppAuth();

    await appAuth.authorizeAndExchangeCode(
      AuthorizationTokenRequest(
        clientId,
        redirectUri,
        clientSecret: clientSecret,
        serviceConfiguration: const AuthorizationServiceConfiguration(
          authorizationEndpoint: authorizationEndpoint,
          tokenEndpoint: tokenEndpoint,
        ),
        scopes: ['openid', 'profile', 'email'], // 任意
      ),
    );
  } catch (e) {
    print('Login error: $e');
    return;
  }
}
```

|                                       1                                        |                                       2                                        |                                       3                                        |
| :----------------------------------------------------------------------------: | :----------------------------------------------------------------------------: | :----------------------------------------------------------------------------: |
| ![](https://storage.googleapis.com/zenn-user-upload/bd77f8c7db1c-20241110.png) | ![](https://storage.googleapis.com/zenn-user-upload/b2a03bc38203-20241110.png) | ![](https://storage.googleapis.com/zenn-user-upload/133b402d35f2-20241110.png) |

<br>

## アプリへのコールバック

認証後はアプリにコールバックするようにしないと実際のアプリでは使用できない（認証成功しても Token が取得できない）ので、認証後にアプリにコールバックするように設定する。

### scheme の設定

認証成功時に WevView からアプリを起動できるように、アプリの scheme を設定する。
設定方法については[flutter_appauth の Readme](https://pub.dev/packages/flutter_appauth) 通りに設定する。

今回は scheme を`authsample`とする。

::: message
URL スキームやディープリンク等については、以下の記事がわかりやすいので、ご参考までに。
[URL スキームとは？—ディープリンク・Universal Links のお勉強とその活用術](https://8vivid.net/whats-url-scheme/)
:::

**【android】**
`build.gradle`と`AndroidManifest.xml`に scheme の設定を追加。

```gradle
defaultConfig {
    <!-- ...省略 -->
    manifestPlaceholders += [
        'appAuthRedirectScheme': 'authsample'
    ]
```

```xml
<activity
    <!-- ...省略  -->
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data
          android:scheme="authsample"
          android:host="callback"
          android:path="/"
          />
    </intent-filter>
```

**【ios】**
`info.plist`に scheme の設定を追加するのみ。

```xml
  <key>CFBundleTypeRole</key>
  <string>Editor</string>
  <key>CFBundleURLSchemes</key>
  <array>
    <string>authsample</string>
  </array>
```

<br>

※ 備考
以下 Auth0 のドキュメントから引用。
今回はカスタムスキームでのコールバックとしているため、Auth0 側の設定は不要。
Universal Links でコールバックする際は Auth0 のドキュメント（`<project>/Quickstart/Flutter`）に沿って設定が必要。

> It is needed to use Universal Links as callback and logout URLs. Skip this step to use a custom URL scheme instead.

<br>

### scheme を用いたコールバック URL の設定

上記で設定した scheme を用いてコールバック URL を設定します。
今回は適当に`authsample://test/callback`としてみます。

Auth0 コンソールのコールバック URL に上記を追加します。

![](https://storage.googleapis.com/zenn-user-upload/95fcec645932-20241110.png)

Flutter アプリで定義した`redirectUri`を`authsample://test/callback`に変更します。

```diff dart
- const String redirectUri = 'https://flutter.dev';
+ const String redirectUri = 'authsample://test/callback';
```

<br>

### 動作確認

アプリ上で取得する Token が確認できるように適当に Widget を追加。

```dart
const String clientId = '<Client ID>';
const String clientSecret ='<Client Secret>';
const String authorizationEndpoint ='<OAuth Authorization URL>';
const String tokenEndpoint ='<OAuth Token URL>';
const String redirectUri = 'authsample://test/callback'; // 変更

// ...省略
ElevatedButton(
    onPressed: () async {
      final result = await _login();
      if (result != null) {
        accessToken.value = result['accessToken'];
        refreshToken.value = result['refreshToken'];
      }
    },
    child: const Text('auth0 login'),
),

// ...省略
Future _login() async {
  try {
    const FlutterAppAuth appAuth = FlutterAppAuth();

    final AuthorizationTokenResponse? result =
      await appAuth.authorizeAndExchangeCode(
        AuthorizationTokenRequest(
          clientId,
          redirectUri,
          clientSecret: clientSecret,
          serviceConfiguration: const AuthorizationServiceConfiguration(
            authorizationEndpoint: authorizationEndpoint,
            tokenEndpoint: tokenEndpoint,
          ),
          scopes: ['openid', 'profile', 'email'], // 任意
        ),
      );

    if (result != null) {
      debugPrint('Access token: ${result.accessToken}\n');
      debugPrint('Refresh token: ${result.refreshToken}\n');
      return {
        'accessToken': result.accessToken,
        'refreshToken': result.refreshToken
      };
    }
  } catch (e) {
    print('Login error: $e');
    return;
  }
}
```

上記の設定により、アプリ上で認証が成功すると、アプリに戻ってきてトークンを取得できることを確認。

|                                        認証前                                        |                                        認証後                                        |
| :----------------------------------------------------------------------------------: | :----------------------------------------------------------------------------------: |
| ![](https://storage.googleapis.com/zenn-user-upload/388a8375fafe-20241128.png =400x) | ![](https://storage.googleapis.com/zenn-user-upload/4417283d07ed-20241128.png =400x) |

<br>

### 備考

android のデフォルトの package 名（applicationId） だと、`auth.sample.app.auth_sample_app` のような形式だが、これだと認証後にトークンが取得できず、以下のエラーに。（ここが いろんな Issue を調べても原因がわからずで沼った。。。）

> PlatformException(null_intent, Failed to authorize: Null intent received, null, null)

id の形式を`auth.sample.app`に変更すると、認証後にトークンが取得できるようになった。

※ ios だとデフォルトの形式`auth.sample.app.authSampleApp`でも問題なく動作した。

<br>

## 参照

https://dev.classmethod.jp/articles/flutter-android-auth0/
https://8vivid.net/whats-url-scheme/
https://www.itmanage.co.jp/column/host-name/
