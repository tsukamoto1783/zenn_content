---
title: "【Swift】Flutterお馴染みのカウンターアプリをSwiftで作ってみた"
emoji: "⏱️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter,swift, ios, swiftui, uikit]
published: true
---
**「SwiftでのiOSアプリ開発の流れや基礎をとりあえず知っておきたい。Viewの実装方法違いをざっくり知りたい。」**

ということでSwiftでFlutterお馴染みのカウンターアプリと同等のアプリを作成し、それぞれの実装方法の違いを確認してみる。

:::message
Swift初触りの記事なので、Swiftコードのお作法とかは考慮されてません。
とりあえず動かしてみることに焦点を当てて記載されていることご了承ください。
:::

<br>

### はじめに
StoryboardやUIKit、SwiftUIなど、「どれがどれ？違いは？」と理解できていなかったので調べていると、以下の記事がわかりやすくまとめてくださっていました。
こちらの内容に軽く目を通すと、かなり理解が進みました。

https://qiita.com/os1ma/items/a8b946dba891f01ccc4e
https://qiita.com/shiz/items/d5d0f0330460a53c16ae

<br>

### 環境
Xcode 15.0
※ XcodeのバージョンによってXcodeのUIが微妙に変わるので注意。

<br>

## Flutter
Flutterお馴染みのカウンターアプリ。
`flutter create <app_name>`を実行するだけで、カウンターアプリが作成される。
シミュレーターを選択して`flutter run` を実行。
![](https://storage.googleapis.com/zenn-user-upload/360700daaf4a-20240127.png =300x)

createされたコードは以下。(お馴染みすぎるが比較のため記載)

:::details main.dart
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
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, required this.title});
  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text(
              'You have pushed the button this many times:',
            ),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: const Icon(Icons.add),
      ),
    );
  }
}
```
:::

<br>

## swift: Storyboard

UIをStoryboard（GUI操作）で作成してみる。
ざっくりと作成手順を記載。


1. Xcodeを開いて、新規プロジェクトを作成。

|                           a. Xcode/File/New/Project                            |                                 b. iOS/App/Next                                 |                               c. choose options                                |
| :----------------------------------------------------------------------------: | :-----------------------------------------------------------------------------: | :----------------------------------------------------------------------------: |
| ![](https://storage.googleapis.com/zenn-user-upload/dc84645fb653-20240110.png) | ![](https://storage.googleapis.com/zenn-user-upload/e744764c1239-20240110.png ) | ![](https://storage.googleapis.com/zenn-user-upload/5927e0bfa0ea-20240110.png) |

2. 新規プロジェクト作成後、左側のナビゲーターから`Main（Main.storyboard）`を選択。選択すると画面プレビューが表示される。
![](https://storage.googleapis.com/zenn-user-upload/b464fb1508c4-20240111.png =600x)

3. オブジェクトライブラリから使用したいオブジェクト（≒ UI部品、コンポーネント）を選択して、ドラッグ&ドロップで配置していく。
オブジェクトライブラリは右上の`＋マーク`押下（cnt + command + L）で表示される。
![](https://storage.googleapis.com/zenn-user-upload/f7f21f0590c5-20240111.png =600x)

4. オブジェクトを任意の位置に配置したら、オブジェクトの各種プロパティ（Style）を設定する。
    ![](https://storage.googleapis.com/zenn-user-upload/bcb4ebeed7ec-20240111.png =800x)

    ※ Buttonの角丸や影などのStyle設定は、プレビューには表示されないものもある。。。
    反映を確認するには毎度ビルドしてシミュレーターで確認する。
    → 反映されない。という記事はいくつか見かけるが、なぜなのか。まで記載されている記事はざっとみただけだと見つけられなかった。。。
    |                       プレビュー - ButtonにStyle反映無し                       |                        シミュレーター - ButtonにStyle反映有り                        |
    | :----------------------------------------------------------------------------: | :----------------------------------------------------------------------------------: |
    | ![](https://storage.googleapis.com/zenn-user-upload/93aa2cc017ed-20240111.png) | ![](https://storage.googleapis.com/zenn-user-upload/712d32961d14-20240127.png =500x) |
    |

5. `ViewController.swift`に配置したオブジェクトの関数処理や変数設定などを記述する。
![](https://storage.googleapis.com/zenn-user-upload/491f339b9133-20240111.png)

6. swiftファイルで定義した関数や変数を、Maii.storyboardのオブジェクトにドラッグ&ドロップで紐付ける。
![](https://storage.googleapis.com/zenn-user-upload/71fb722dc466-20240111.png)

7. カウンターアプリが完成。（添付画像だとオブジェクトのStyleが多少違うが、調整すればほぼ同じに。）
![](https://storage.googleapis.com/zenn-user-upload/712d32961d14-20240127.png =300x)

<br>

GUI操作で直感的にUIの作成ができるが、細かい設定やStyleの設定がややこしい印象。
どのプロパティがどの項目か、どこにあるのか、などがわかりにくく設定しづらい。（慣れの問題説はあるが）
<br>

## swift: UIKit
UIをUIKit（ソースコード）で作成してみる。
1. storyboardの時と同様に、新規プロジェクトを作成。
2. Main.storyboardは使用せずに、赤線内の`.swift`ファイルだけでUIを作成していく。
   ![](https://storage.googleapis.com/zenn-user-upload/a864e6658c6a-20240118.png =600x)
3. `ViewController.swift`にオブジェクトの配置設定を書く。1画面だけのアプリだと基本はこのファイルだけで完結。
    :::details ViewController.swift
    ```swift
    import UIKit

    class ViewController: UIViewController {
        var count = 0
        let countLabel = UILabel()
        let discription = UILabel()
        let addButton = UIButton()

        override func viewDidLoad() {
            super.viewDidLoad()
            setupCounterLabel()
            setupDiscriptionLabel()
            setupIncrementButton()
            setupNavigationBar()
        }

        // UILabelのセットアップ
        func setupCounterLabel() {
            countLabel.text = "\(count)"
            countLabel.textAlignment = .center
            view.addSubview(countLabel)

            // Auto Layoutを有効化
            countLabel.translatesAutoresizingMaskIntoConstraints = false

            // Layout Anchorsを使用した配置設定
            NSLayoutConstraint.activate([
                countLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
                countLabel.centerYAnchor.constraint(equalTo: view.centerYAnchor, constant: 30),
                countLabel.widthAnchor.constraint(equalToConstant: 200),
                countLabel.heightAnchor.constraint(equalToConstant: 20)
            ])
        }

        // UILabelのセットアップ
        func setupDiscriptionLabel() {
            discription.text = "You have pushed the button this many times:"
            discription.textAlignment = .center
            view.addSubview(discription)

            // Auto Layoutを有効化
            discription.translatesAutoresizingMaskIntoConstraints = false

            // Layout Anchorsを使用した配置設定
            NSLayoutConstraint.activate([
                discription.centerXAnchor.constraint(equalTo: view.centerXAnchor),
                discription.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            ])
        }


        // UIButtonのセットアップ
        func setupIncrementButton() {
            addButton.setTitle("+", for: .normal)
            
            // Button Style設定
            addButton.setTitleColor(UIColor.black, for: .normal)
            addButton.backgroundColor = UIColor.systemPurple
            addButton.layer.cornerRadius = 10
            addButton.layer.shadowColor = UIColor.black.cgColor
            addButton.layer.shadowOffset = CGSize(width: 0, height: 4)
            addButton.layer.shadowOpacity = 0.5
            addButton.layer.shadowRadius = 4

            view.addSubview(addButton)
            addButton.addTarget(self, action: #selector(addButtonTapped), for: .touchUpInside)

            // Auto Layoutを有効化
            addButton.translatesAutoresizingMaskIntoConstraints = false

            // Layout Anchorsを使用した配置設定
            NSLayoutConstraint.activate([
                addButton.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -50),
                addButton.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -50),
                addButton.widthAnchor.constraint(equalToConstant: 60),
                addButton.heightAnchor.constraint(equalToConstant: 60)
            ])
        }

        // ナビゲーションBarのセットアップ
        private func setupNavigationBar() {
            navigationItem.title = "Swift UIKit Demo Page"
        }


        @objc func addButtonTapped() {
            count += 1
            countLabel.text = "\(count)"
        }
    }
    ```
    :::

4. `AppDelegate.swift`と`SceneDelegate.swift`にNavigationBarの設定の記述する。
   NavigationBarの設定は、ViewController.swiftの記述だけでは反映されなかっため、それぞれのファイル少し設定を追加。
    :::details AppDelegate.swift
    ```swift 
    import UIKit

    @main
    class AppDelegate: UIResponder, UIApplicationDelegate {

        var window: UIWindow?
        
        func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
            //iOS 15でNavigationBarがデフォルトで透過されるためのios14までと同じように表示する設定
            if #available(iOS 15.0, *) {
                //ナビゲーションバーの外観設定を宣言
                let navigationBarAppearance = UINavigationBarAppearance()
                //デフォルトの背景色を設定
                navigationBarAppearance.configureWithDefaultBackground()
                //各モードに代入
                UINavigationBar.appearance().standardAppearance = navigationBarAppearance
                UINavigationBar.appearance().compactAppearance = navigationBarAppearance
                UINavigationBar.appearance().scrollEdgeAppearance = navigationBarAppearance
                
                //ナビゲーションバーのタイトル文字の色変更
                navigationBarAppearance.titleTextAttributes = [.foregroundColor: UIColor.black]
                //ナビゲーションバーの背景色変更
                navigationBarAppearance.backgroundColor = UIColor.systemPurple
            }
            return true
        }
        // 以下割愛
    }
    ```
    :::

    :::details SceneDelegate.swift
    ```swift 
    import UIKit

    class SceneDelegate: UIResponder, UIWindowSceneDelegate {

        var window: UIWindow?


        func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
            guard let windowScene = (scene as? UIWindowScene) else { return }

            // 以下の設定を加えないとNavigationBarが表示されない。

            // UIWindowを生成し、ウィンドウシーンを設定します。
            let window = UIWindow(windowScene: windowScene)
            // 新しいウィンドウをSceneDelegateのプロパティとして保持します。
            self.window = window
            
            // ルートビューコントローラとしてUINavigationControllerを設定します。
            window.rootViewController = UINavigationController(rootViewController: ViewController())
            // ウィンドウをキーウィンドウとして表示し、ユーザーとのインタラクションを開始します。
            window.makeKeyAndVisible()
            
            // ウィンドウの背景色を白に設定します。
            window.backgroundColor = .white
            
        }
        // 以下割愛
    }
    ```
    :::
5. buildすると、カウンターアプリが完成。（色味はStyleやや違うが、機能は同じカウンターアプリ）
    ![](https://storage.googleapis.com/zenn-user-upload/a6781f6d40a3-20240127.png =300x) 

    ※ Main.storyboardは使用していないので、ファイルを開いても真っ白。
    ![](https://storage.googleapis.com/zenn-user-upload/3d761f287fc1-20240118.png =600x)

<br>

コードベースで全て設定できるが、swift固有の記述が多い？見慣れない書き方が多くて手間取った。特にNavigationBarの設定が無駄にややこしかった。
→ 表示するだけでこんなに記述量がいるので、画面遷移とか設定しだすともっとややこしくなる？

<br>

## SwiftUI
SwiftUIでカウンターアプリを作成してみる。
イメージとしては、storyboardのようなGUI操作もできる、UIKitの強化版みたいなイメージ。

1. storyboardの時と同様に、新規プロジェクトを作成。`Interface`項目にて`SwiftUI`を選択。
2. 作成すると以下のような構成となる。
   ![](https://storage.googleapis.com/zenn-user-upload/87b2d5b36a45-20240127.png)
3. `#Preview`で囲まれた部分がプレビュー画面に表示される。
   ![](https://storage.googleapis.com/zenn-user-upload/e73d9f8401ee-20240127.png)
4. storyboardのように`Inspector`でオブジェクトのプロパティを設定することも可能。
   また、UIライブラリからオブジェクトをドラッグ操作でコード部分に配置することも可能。UIライブラリを開くには、右上の+アイコンか、`Shift + command + L`。
    |xcode|xcode|
    |:-:|:-:|
    |   ![](https://storage.googleapis.com/zenn-user-upload/6804019b5091-20240127.png)|   ![](https://storage.googleapis.com/zenn-user-upload/109e1715fd47-20240127.png)|
5. 以下のように`ContentView.swift`のコードを変更。
   :::details ContentView.swift
   ```swift
   import SwiftUI

    struct ContentView: View {
        @State private var counter = 0
        
        var body: some View {
            NavigationStack{
                VStack() {
                    Spacer()
                    Text("You have pushed the button this many times:")
                    Text("\(counter)")
                        .padding(.top, 16)
                        .font(.title)
                    Spacer()
                    
                    HStack(){
                        Spacer()
                        Button(action: {
                            counter += 1
                        }) {
                            Image(systemName: "plus")
                                .foregroundColor(.black)
                                .padding()
                                .background(Color.purple)
                                .cornerRadius(10)
                                .shadow(radius: 4)
                        }
                        .padding(.trailing, 40.0)
                    }
                }
                .toolbar {
                        ToolbarItem(placement: .principal) {
                            Text("SwiftUI Demo Page")
                                .font(.title)                               .foregroundColor(.white)
                        }
                    }
                .navigationBarTitle("", displayMode: .inline)
                .toolbarBackground(Color.purple,for: .navigationBar)
                .toolbarBackground(.visible, for: .navigationBar)
                .toolbarColorScheme(.dark)
            }
        }
    }

    #Preview {
        ContentView()
    }
   ```

6. 完成。
   ![](https://storage.googleapis.com/zenn-user-upload/d04d6944d57c-20240127.png =300x)

<br>
UIKitよりかなりコードはスッキリして、navigationBarがUIKitに比べて簡単に設定できる印象。
（個人的には公式ドキュメントがやや見にくい。。apple特有のあの小洒落た感じ。。
→ 慣れるとすんなり読めるようになるはずだけど。）

<br>

## おわり
今回は1画面だけのシンプルなカウンターアプリの作成だったため、細かいところまでは比較できていないが、それでもSwiftUIの開発体験はStoryboardやUIKitに比べると良さそうだということは感じることができた。

ただ、1画面だけのアプリだとSwiftUIの良さを十分に感じることはできないので、公式チュートリアルを実施しながら色々と触っていきたい。

https://developer.apple.com/tutorials/swiftui