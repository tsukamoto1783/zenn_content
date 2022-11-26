---
title: "【Flutter】【AndroidStudio】fvm + dart-define使用時のブレークポイント設定"
emoji: "🐞"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter,Dart,fvm,DartDefine,AndroidStudio]
published: true
---

# 問題
-  **「fvm + dart-define」の環境でも、ブレークポイントを使用して処理止めたい。**
  **AndroidStudioおなじみの虫アイコンを押してブレークポイントで止まるdebugモードで実行したい。**
![](/images/flutter_fvm_dart_define/breakpoint.png)

- 長いコマンド打つのめんどくさい。
- VSCodeの記事が多くて、AndroidStudioの記事が少ない。

# 前提条件
- dart-defineで環境分けをしている。
- バージョン管理はfvmを使用している。
- エディタはAndroidStudioを使用。


# 解決方法
## 【fvm】
※fvmの導入は完了しているものとして以下記載。
　まだの人は"flutter fvm"で検索するとたくさん記事でてくるのでそちらを参照。

①Android Studio/Preferences を選択。
![](/images/flutter_fvm_dart_define/sukusyo5.png =250x)

②Languages & Frameworks / Flutter / SDK / Flutter SDK path: の設定値を変更する。
  デフォルトだと以下のようなパスが指定されているはず。
![](/images/flutter_fvm_dart_define/sukusyo6.png )
  以下のようにプロジェクトのパスに合わして設定値変更。
  ex.) "/Users/<ユーザー名>/StudioProjects/<プロジェクト名>/.fvm/flutter_sdk"
![](/images/flutter_fvm_dart_define/sukusyo7.png )

③上記の設定することで、"/.fvm/fvm_config.json"に記載のflutterバージョン("fvm use"で適用になっているFlutterバージョン)が、デバッグボタンでの実行でも適用される。
![](/images/flutter_fvm_dart_define/sukusyo8.png =600x)
*"/.fvm/fvm_config.json"*

----------------------
## 【dart-define】
①AndroidStudioにて、「Edit Configurations」を選択。
![](/images/flutter_fvm_dart_define/sukusyo1.png)

②左上の追加ボタンを押して"Flutter"を選択。
![](/images/flutter_fvm_dart_define/sukusyo2.png)

③すると、以下のように"Unnamed"でファイルが作成されるので、各種設定。
![](/images/flutter_fvm_dart_define/sukusyo3.png)
  - **Name:** "main.dart"みたいにrunの時に表示される名前を記載。
    ex.) "Development" とか。
  - **Dart entrypont:** 実行したい処理のパスを指定。
    基本的には、"/ <プロジェクトパス> /lib/main.dart" になるかと。
  - **Additional run args:** "flutter run" の後に続くコマンドを記載。 
    ex.) "--dart-define=FLAVOR=<FLAVOR名>"

④設定後、"apply"押して"OK"押して、Nameに設定した値が反映されてたらOK。
![](/images/flutter_fvm_dart_define/sukusyo4.png)

## おわり
上記の設定で虫アイコンを押すだけで簡単にデバック実行ができる(ブレークポイントで止まる)ように。

### 参考記事
::::details 参考記事
https://zenn.dev/mamushi/scraps/13c0232c88227e
https://zenn.dev/altiveinc/articles/separating-environments-in-flutter?redirected=1#android-studio-%E3%81%AE%E8%A8%AD%E5%AE%9A%E4%BE%8B

::::
