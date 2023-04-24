---
title: "【Flutter】auto_routeパッケージの動作確認"
emoji: "🏁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, Dart, AutoRoute]
published: true
publication_name: ncdc
---

auto_routeパッケージについて、あまり日本語記事がなかった＆ドキュメント通りだと若干サンプルコードが動かないので、メモがてら記事にまとめてみた。


以下、ドキュメントに沿ってサンプルアプリを作成しながら確認していく。
※見出し名は、ドキュメントの見出し名と対応してます。
https://pub.dev/packages/auto_route

:::message
auto_route 6.3.0時点での記事です。
:::

# STEP1
動作確認用に、ボタンを2画面表示するだけの簡単なアプリを作成する。

```yaml
project(root)
└─ lib
   ├─ main.dart
   ├─ app_router.dart
   └─ tabs
      ├─ home_page.dart
      └─ settings_page.dart
```


::::details flutter doctor
```
[✓] Flutter (Channel stable, 3.7.10, on macOS 13.2.1 22D68 darwin-arm64, locale ja-JP)
[✓] Android toolchain - develop for Android devices (Android SDK version 33.0.2)
[✓] Xcode - develop for iOS and macOS (Xcode 14.2)
[✓] Chrome - develop for the web
[✓] Android Studio (version 2022.1)
[✓] VS Code (version 1.77.2)
[✓] Connected device (3 available)
[✓] HTTP Host Availability
```
::::

::::details サンプルアプリコード
```dart main.dart
void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: const HomePage(),
    );
  }
}
```

```dart home_page.dart
class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Home Page'),
      ),
      body: Center(
        child: ElevatedButton(
          child: const Text('Setting Page'),
          onPressed: () => Navigator.push(
            context,
            MaterialPageRoute(builder: (context) => const SettingPage()),
          ),
        ),
      ),
    );
  }
}

```

```dart setting_page.dart
class SettingsPage extends StatelessWidget {
  const SettingsPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Settings Page'),
      ),
      body: Center(
        child: ElevatedButton(
          child: const Text('Home Pageへ戻る'),
          onPressed: () => Navigator.pop(context),
        ),
      ),
    );
  }
}
```

::::

## Installation
pubspec.yamlに必要なパッケージを追加。
```yaml :pubspec.yaml
dependencies:                    
  auto_route: ^6.3.0                    
                    
dev_dependencies:                    
  auto_route_generator: ^6.2.0                    
  build_runner: ^2.3.3
```
&nbsp;

## Using part builder
ルータークラスを作成。
ドキュメントに倣って、AppRouterクラスとする。
freezedなどを自動生成設定する時と基本的には同じ構造。

※バージョン5系と6系とでドキュメントの記載が違うので注意。

:::message
@AutoRouterConfigに、replaceInRouteNameを指定することで、ジェネレート時に文字列を変換してくれる。
ex.) HomePage -> HomeRoute

::::details 詳細は以下参照

```dart
class AutoRouterConfig {
/// Auto generated route names can be a bit long with
/// the [Route] suffix
/// e.g ProductDetailsPage would be ProductDetailsPageRoute
///
/// You can replace some relative parts in your route names
/// by providing a replacement in the follow pattern
/// [whatToReplace,replacement]
/// what to replace and the replacement should be
/// separated with a comma [,]
/// e.g 'Page,Route'
/// so ProductDetailsPage would be ProductDetailsRoute
///
/// defaults to 'Page|Screen,Route', ignored if a route name is provided.
final String? replaceInRouteName;

/// Use for web for lazy loading other routes
/// more info https://dart.dev/guides/language/language-tour#deferred-loading
/// defaults to false
final bool deferredLoading;

/// Only generated files exist in provided directories will be processed
/// defaults = const ['lib']
final List<String> generateForDir;
```
::::


```dart :app_router.dart
part 'app_router.gr.dart';            

@AutoRouterConfig(replaceInRouteName: 'Page,Route')      
class AppRouter extends _$AppRouter {      
    
  @override      
  List<AutoRoute> get routes => [      
   /// 後ほどbuild_runnerで生成したルートを記載する。
   ]    
 }
```
&nbsp;

## Generating Routable pages
各Pageの頭にアノテーションをつける。

```dart :home_page.dart
@RoutePage()
class HomePage extends StatelessWidget {
```
&nbsp;

## Now simply run the generator
build_runnerを実行する。
```
flutter packages pub run build_runner build
```
&nbsp;

## Add the generated route to your routes list
build_runnerを実行したら、お馴染みの.gr.dartファイルが生成されるので、生成されたルートをルータークラスに記載する。


:::message
ドキュメントコピペだとエラーになるので注意。
:::


```dart :app_router.dart
@AutoRouterConfig(replaceInRouteName: 'Page,Route')      
class AppRouter extends $AppRouter {      
    
  @override      
  List<AutoRoute> get routes => [      
    // ドキュメントのままだと、以下の内容でエラーになる。
    // アプリ起動時の初期ルート設定しないと、"Can not resolve initial route"と怒られる。
    // page: requiredの名前付きコンストラクタなので、page:の記載が必要。

    // 以下のように修正
    AutoRoute(page: HomeRoute.page, initial: true),
    // initial:true ではなく、path:'/' でも可。
    // AutoRoute(path: '/', page: HomeRoute.page),


    AutoRoute(page: SettingsRoute.page),
   ]    
 }
```
※ pathを指定しない場合は、自動的に割り当てられる。
>if you don’t specify a path it’s going to be generated from the page name e.g. BookListPage will have ‘book-list-page’ as a path, if initial arg is set to true the path will be / unless it's relative then it will be an empty string ''.

他にもプロパティがあるので、詳細は下記参照。


::::details AutRouteのプロパティ一覧
```dart :auto_route_config.dart
class AutoRoute {
  /// The name of page this route should map to
  final String name;
  final String? _path;

  /// Weather to match this route's path as fullMatch
  final bool fullMatch;
  final RouteCollection? _children;

  /// The list of [AutoRouteGuard]'s the matched route
  /// will go through before being presented
  final List<AutoRouteGuard> guards;

  /// If set to true the [AutoRoutePage] will use the matched-path
  /// as it's key otherwise [name] will be used
  final bool usesPathAsKey;

  /// a Map of dynamic data that can be accessed by
  /// [RouteData.mete] when the route is created
  final Map<String, dynamic> meta;

  /// Indicates what kind of [PageRoute] this route will use
  /// e.g [MaterialRouteType] will create [_PageBasedMaterialPageRoute]
  final RouteType? type;

  /// Whether to treat the target route as a fullscreenDialog.
  /// Passed To [PageRoute.fullscreenDialog]
  final bool fullscreenDialog;

  /// Whether the target route should maintain it's state.
  /// Passed To [PageRoute.maintainState]
  final bool maintainState;

  /// Builds page title that's passed to [_PageBasedCupertinoPageRoute.title]
  /// where it can be used by [CupertinoNavigationBar]
  ///
  /// it can also be used manually by calling [RouteData.title] inside widgets
  final TitleBuilder? title;

  /// Builds a String value that that's passed to
  /// [AutoRoutePage.restorationId]
  final RestorationIdBuilder? restorationId;

  /// Whether the target route should be kept in stack
  /// after another route is pushed above it
  final bool keepHistory;

  /// Marks route as initial destination of a router
  ///
  /// initial will auto-generate initial paths
  /// for routes with defined-paths
  ///
  /// if used with a non-initial defined path it auto-generates
  /// a RedirectRoute() to that path
  final bool initial;
```
::::
&nbsp;

## Finalize the setup
build_runnerで生成されたルータークラスを、MaterialAppに接続する。
```dart
class MyApp extends StatelessWidget {
  MyApp({super.key});

  final _appRouter = AppRouter(); //追加

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router( // routerを追加
      routerConfig: _appRouter.config(), // 追加
      title: 'Flutter Demo',
      theme: ThemeData(primarySwatch: Colors.blue),
      // home: const HomePage(), // homeを削除
    );
  }
}
```
&nbsp;

## Navigating Between Screens
画面遷移の記述を、auto_routeの記述に変更する。

:::message
auto_routeは、ジェネレートされたルートとpathの両方を使って、ページのスタック操作をするための一般的なpush、popなどのメソッドを提供します。
:::

```dart :home_page.dart
child: ElevatedButton(
    child: const Text('Setting Page'),

    // <修正前>
    // onPressed: () => Navigator.push(context,
    //   MaterialPageRoute(builder: (context) => const SettingPage()),),
    
    // <修正後>
    onPressed: () => context.router.push(const SettingsRoute()),
    
    // 引数がある場合はシンプルに引数を渡す
    // onPressed: () => context.router.push(const SettingsRoute(name: "Hoge")),
),        
```


```dart :メソッド一覧
// 以下どちらの記述でも可能
AutoRouter.of(context)
context.router

// ページスタックに新しいエントリを追加
context.router.push(const BooksListRoute())                  
// pathを使用する場合は以下
context.router.pushNamed('/books')

// 以下省略。詳細はドキュメント参照。
```
&nbsp;


## Returning Results
画面遷移の結果を返すには2パターンある。
"pop completer"を使うか、オブジェクトを渡すのと同じようにコールバック関数を引数として渡すか。
&nbsp;

<!-- 必要になったら追記する -->




# STEP2
ここまでで、基本的な画面遷移はauto_routeで実装できた。
ここからは、auto_routeでボトムナビゲージョンバーの実装を実施してみる。

※ドキュメントでは、TabPageの名前はUsers,Posts,Settingsというページを作っているが、ここでは以下で実装してます。やってることは一緒です。

画面構成を以下とする。
```yaml
project(root)
└─ lib
   ├─ main.dart
   ├─ app_router.dart
   ├─ dashboard_page.dart  // 追加
   └─ tabs
      ├─ home_page.dart
      ├─ posts_page.dart   // 追加
      ├─ details_page.dart // 追加
      └─ settings_page.dart
```
&nbsp;


## Tab Navigation / Nested navigation
auto_routeの"AutoTabsRouter"を使用して、ボトムナビゲーションバーを実装する。
"AutoTabsRouter"を使用すれば簡単にボトムナビゲージョンバーが実装できる。

また、各タブ画面にネストされたナビゲーションを実装するには、各タブ画面を親ルートとして、子ルートを設定してあげる必要がある。

:::message
ネストしたルートを構築するには、親ルートのページ内で、ネストしたルーター・ビューとして機能するAutoRouterウィジェットが必要。

> To render/build nested routes we need an AutoRouter widget that works as an outlet or a nested router-view inside of our dashboard page.
:::
&nbsp;

上記、AutoRouterを使用しろと記載があるが、具体例がドキュメントには無い。。
→AutoRouterを使用せずに、直感的にルート設定すると、思った通りの遷移にならない。
→いくつかIssueを見る感じ、AutoRouterクラスをラップしているIssueがちらほら見つかる。
→Issueの通りに実装すると上手くいった。

https://github.com/Milad-Akarie/auto_route_library/issues/1453



```dart :app_router.dart
@AutoRouterConfig(replaceInRouteName: 'Page,Route')
class AppRouter extends _$AppRouter {
  @override
  List<AutoRoute> get routes => [
        AutoRoute(
          path: '/dashboard',
          initial: true,
          page: DashboardRoute.page,
          children: [
            AutoRoute(path: 'home', page: HomeRoute.page, initial: true),
            AutoRoute(path: 'posts', page: PostsRoute.page),
            AutoRoute(
              path: 'setting',
              page: SettingsAutoRouterRoute.page,
              children: [
                AutoRoute(initial: true, page: SettingsRoute.page),
                AutoRoute(path: 'details', page: DetailsRoute.page),
              ],
            ),
          ],
        　// 以下だと上手くいかない
          // children: [
          //  AutoRoute(path: 'home', page: HomeRoute.page, initial: true),
          //  AutoRoute(path: 'posts', page: PostsRoute.page),
          //  AutoRoute(
          //    path: 'setting',
          //    page: SettingsRoute.page,
          //    children: [
          //      AutoRoute(path: 'details', page: DetailsRoute.page),
          //    ],
          //  ),
          // ],
        ),
      ];
}
```

```dart
@RoutePage()
class SettingAutoRouterPage extends AutoRouter {
  const SettingAutoRouterPage({super.key});
}

@RoutePage()
class SettingPage extends StatelessWidget {
  const SettingPage({super.key, required this.name});
  final String name;
  @override
  Widget build(BuildContext context) {
    return // 以下省略
```
&nbsp;


### AutoTabsRouter


"AutoTabsRouter"ではなくて、よりスッキリ書ける"AutoTabsScaffold"でも代用可能。
(むしろAutoTabsScaffoldを推奨?)

>/// A scaffold wrapper widget that makes creating an [AutoTabsRouter]
> /// much easier and cleaner

> if you think the above setup is a bit messy you could use the shipped-in AutoTabsScaffold that makes things much cleaner.


![](https://storage.googleapis.com/zenn-user-upload/88180ade89a4-20230424.gif =250x)
&nbsp;


### AutoTabsRouter.pageView
タブの切り替えを、スワイプで切り替えることができるようになる。

### AutoTabsRouter.tabBar
ヘッダー部分にタブバーを表示させる。
![](https://storage.googleapis.com/zenn-user-upload/107f666f60c4-20230424.png =250x)
&nbsp;


# その他
## popUntil()
```
context.router.popUntil((route) => route.settings.name == <指定のルート名>.name)
```
指定のスタック層（指定のディレクトリ）まで戻れる。
![](https://storage.googleapis.com/zenn-user-upload/8f8b8c6e614b-20230424.gif =250x)
&nbsp;

```
context.router.popUntil((route) => route.isFirst)
```
全てのスタックを破棄できる。（ルートディレクトリまで戻れる）
![](https://storage.googleapis.com/zenn-user-upload/dde986dfce6b-20230424.gif =250x)
&nbsp;


# 参考
https://github.com/Milad-Akarie/auto_route_library/issues/1453
https://appunite.com/blog/navigation-with-autoroute
