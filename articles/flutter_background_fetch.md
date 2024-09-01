---
title: "【Flutter】iOS のバックグラウンドタスクについての仕様調査"
emoji: "🔄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, ios, BackgroundTask, background_fetch, background]
published: true
publication_name: ncdc
---

flutter でバックグラウンド時に定期処理を実行したくて、色々と調査した結果を記載します。

※ iOS がバックグラウンドタスクについて制約が多かったので、この記事では iOS メインで記載します。android については言及しません。

:::message
以下の記事内の引用は、README や公式ドキュメントの内容を翻訳して記載しております。
若干翻訳のニュアンスが違うこともあると思うので、詳しくは引用元をご確認ください。
:::

<br>

## ライブラリ選定

[Flutter Gems](https://fluttergems.dev/android-ios/)の`Top Flutter Android/iOS Device Software and Hardware packages`を見てみると、以下のようなライブラリが出てきます。
![](https://storage.googleapis.com/zenn-user-upload/9060a6224ac0-20240901.png)

上位 3 つがどれもバックグラウンド処理に関連するライブラリです。
今回はこの 3 つから選定してみます。

<br>

:::message
以下は 2024/8/31 時点での情報です。
:::

| ライブラリ                                                                        | pub.dev<br>Star | pub.dev<br>最終更新 | Github<br>Star | Github<br>最終 Commit | Github<br>Issue | 感想                                                                                                             |
| --------------------------------------------------------------------------------- | --------------- | ------------------- | -------------- | --------------------- | --------------- | ---------------------------------------------------------------------------------------------------------------- |
| [workmanager](https://pub.dev/packages/workmanager)                               | 2.0K            | 11 months ago       | 836            | 4 months ago          | 94              | Star 数は一番多いので実績はありそう。<br>ただ Issue を見る限り BugTag がやや多い印象。<br>更新頻度も高くはない。 |
| [flutter_background_service](https://pub.dev/packages/flutter_background_service) | 1.2K            | 3 days ago          | 261            | 5 days ago            | 188             | 直近は更新が復活しているので、今後改善が進むかも？<br>Issue が多いのが気になる。                                 |
| [background_fetch](https://pub.dev/packages/background_fetch)                     | 1.1K            | 3 months ago        | 566            | 2 months ago          | 1               | 更新頻度もそこそこで、Issue がほぼ無いのがすごい。                                                               |

<br>

実装中に動かないとかで調査する時間をできるだけ無くしたく、安心感のありそうな `background_fetch`を今回は利用してみます。

<br>

## background_fetch

以下、README より引用。

> Background Fetch は非常にシンプルなプラグインで、**約 15 分ごと**にバックグラウンドでアプリを起動し、バックグラウンドでの短い実行時間を提供します。 このプラグインは、バックグラウンドフェッチイベントが発生するたびに、提供された callbackFn を実行します。
>
> Background Fetch は、任意の「ワンショット」または定期的なタスクをスケジューリングするための scheduleTask メソッドを提供するようになりました

<br>

ライブラリの名前の通り、定期的にバックグラウンドでタスクを実行するためのライブラリとの記載があり。

簡単に意図通りに実装できるだろうと簡単に考えていましたが、README の iOS 項目に以下の記載がありました。

> - **フェッチイベントの発生速度を上げる方法はなく**、このプラグインは可能な限り頻繁に発生するように設定します。**15 分以上早くイベントを受信することはありません**。
>   オペレーティングシステムは、**使用パターンに基づいてバックグラウンドフェッチイベントの発生速度を自動的に調整します**。
>   例: ユーザーが長時間電源を入れていない場合、フェッチイベントの発生頻度は低くなります。
> - **scheduleTask は、デバイスが電源に接続されている時のみ実行されるようです**。`scheduleTask` は優先度の低いタスクのために設計されており、**あなたが望むほど頻繁に実行されることはありません。**
> - デフォルトの `fetch` タスクの方がはるかに頻繁に実行されます。
> - ⚠️ **アプリが終了すると、iOS はイベントを発火しなくなります。** - iOS には `stopOnTerminate: false` というものはありません。
> - **iOS は、アップルの機械学習アルゴリズムが落ち着き、定期的にイベントを開始するまでに数日かかることがある**。イベントが発火するのをじっとログを見つめて待っていてはいけない。シミュレートしたイベントが動作すれば、それだけですべてが正しく設定されていることがわかります。
> - もしユーザーが長時間 iOS アプリを開かなければ、iOS はイベントの発火を停止します。

<br>

ライブラリの目的というか主機能としては、`約 15 分ごとにバックグラウンドでアプリを起動`なのに、これは iOS だとほぼ満たせないのでは、、、

なお、`workmanager` でも似たような内容の記載あり。

> 特に、バックグラウンドタスクがいつ実行されるかは保証されていません。 ドキュメントからの抜粋。
> 処理タスクリクエストをスケジュールして、同期、データベースのメンテナンス、または同様のタスクなど、延期可能で長時間実行する処理を処理するために、バッテリーの寿命に有利な条件が整ったときに、システムにアプリの起動を依頼します。 システムは、ユーザーが過去 1 週間以内にあなたのアプリを使用している限り、今後 2 日以内に可能な限りこの要求を満たそうとします。

`flutter_background_service` の README には、このような記載はないが、Issue のやり取りを見ると iOS のバックグラウンド処理が機能しないとう記載がありました。
理由としては同様の制約っぽいです。
https://github.com/ekasetiawans/flutter_background_service/issues/437

<br>

こういった iOS の制約について全く把握できていなかったので、iOS の Document を読んでこの辺の制約を見てみます。

※ 以下、swift で実際に実装したわけではないのでざっくり理解の記載であることをご了承ください。

<br>

## BackgroundTask

iOS でバックグラウンドでのタスク実行には、[BackgroundTask](https://developer.apple.com/documentation/backgroundtasks) という Framework を利用します。

> Overview
> BackgroundTasks フレームワークを使用して、アプリのコンテンツを最新の状態に保ち、アプリがバックグラウンドにある間に数分で完了するタスクを実行します。**長時間の作業では、オプションで外部電源とネットワーク接続が必要になることもある。**
> アプリの起動時にタスクの起動ハンドラを登録し、**必要に応じてスケジュールを設定します**。システムはバックグラウンドでアプリを起動し、タスクを実行します。

<br>

ざっくりと関係性をまとめると以下のようになります。
BackgroundTask 以下の各クラスについてはこの後順に見ていきます。

```yaml
- BackgroundTask: フレームワーク
  - BGTaskScheduler: バックグラウンドで実行するタスクをスケジュール（管理）するクラス。
    - BGAppRefreshTaskRequest: タスク
    - BGProcessingTaskRequest: タスク
    - etc.: タスク
```

<br>

### BGTaskScheduler

「BGTaskScheduler ≒ バックグラウンドで実行するタスクをスケジュール（管理）するクラス。」
BGTaskScheduler に実施したい Task を登録していくイメージ。

以下、Apple Developer Document / [Using background tasks to update your app](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_background/using_background_tasks_to_update_your_app)より引用。

> Overview
> タスクとは、アプリが実行する独立したアクティビティのことで、多くの場合、定期的に実行される。タスクの例としては、データベースのメンテナンス、機械学習モデルの改良、表示データの更新などがあります。デバイスが使用されていないときに処理時間を活用するために、アプリをバックグラウンドで起動してタスクを実行するように設定できます。
> バックグラウンドで実行するタスクをスケジュールするには、Xcode でバックグラウンドモードを有効にし、必要な特定のタスクを特定し、BGTaskScheduler オブジェクトにタスクを登録します。

> Enable and schedule background tasks
> バックグラウンド・タスクを許可するようにアプリを設定するには、必要なバックグラウンド機能を有効にし、各タスクの一意の識別子のリストを作成します。 バックグラウンド・タスクには 2 つのタイプがあります： BGAppRefreshTask と BGProcessingTask です。 BGAppRefreshTask は、株価のダウンロードなど、迅速な結果が期待できる短時間のタスク用です。 BGProcessingTask は、大きなファイルのダウンロードやデータの同期など、時間のかかるタスク用です。 アプリはこれらの 1 つまたは両方を使用できます。

<br>

### BGAppRefreshTask

> 通常、アプリがバックグラウンドにある間に実行されるコンテンツの更新に使われる、短いタスクを表すオブジェクト。
>
> Overview
> アプリのリフレッシュ・タスクは、最新の株価など、ちょっとした情報をアプリで更新するために使用します。

<br>

### BGProcessingTask

> アプリがバックグラウンドで動作している間に実行される、時間のかかる処理タスク。
>
> Overview
> 長時間のデータ更新、データ処理、アプリのメンテナンスには処理タスクを使用します。 処理タスクは数分間実行できますが、システムが処理を中断することもあります。
> 処理タスクは、デバイスがアイドル状態のときにのみ実行されます。 ユーザーがデバイスの使用を開始すると、システムは実行中のバックグラウンド処理タスクを終了します。 バックグラウンド更新タスクは影響を受けない。

<br>

### Apple Developer Document / Choosing Background Strategies for Your App

> Update Your App’s Content
> 例えば、アプリは定期的にサーバーからコンテンツを取得したり、定期的に内部状態を更新したりします。このような状況では、BGAppRefreshTaskRequest を要求して、BGAppRefreshTask を使用します。
> **システムは、あなたのバックグラウンドタスクを起動する最適な時間を決定し**、あなたのアプリに最大 30 秒間のバックグラウンドランタイムを提供します。この時間内に作業を完了し、setTaskCompleted(success:)を呼び出すか、システムがアプリを終了します。

> Defer Intensive Work
> **バッテリーの寿命とパフォーマンスを維持するために、デバイスが充電される夜間など、アクティビティが低い時間帯にバックグラウンドタスクをスケジュールすることができます**。機械学習モデルのトレーニングやデータベースのメンテナンスなど、アプリが重いワークロードを管理する場合は、この方法を使用してください。
> BGProcessingTask を使用してこれらのタイプのバックグラウンドタスクをスケジュールすると、システムがバックグラウンドタスクを起動する最適な時間を決定します。

<br>

### Apple Developer Document / Refreshing and Maintaining Your App Using Background Tasks

[Advances in App Background Execution](https://developer.apple.com/jp/videos/play/wwdc2019/707/)

40 分もある動画ですがトランスクリプトがついてるので、ざっくりと読むことができます。

<br>

## おわり

Apple Developer Document を読むと、なぜ Flutter ライブラリの仕様がこうなっているのかが概ね理解できました。
Apple の Document には説明が少なくて完全には明記されてない部分も多々ありますが、Flutter ライブラリの注意書きの内容は、iOS の仕様なのでしょうがないということもわかりました。
（この辺の仕様にはるかに詳しいライブラリの作者がそのように明記しているので尚更。）

**結論として**、
iOS でバックグラウンドタスクの実装はできるが、正確に意図通りに制御することは難しいということがわかりました。

iOS のバックグラウンドタスクの仕様の解説については、下記の参考記事の方がより詳細に記載されているので、そちらも併せてご参照ください。

<br>

備考：
（どうしてもバックグラウンドでの定期処理が必要で、時間や頻度を制御したい場合は、background への移行を契機に`Timer` などで無理くり実装できるのか、、、？）

<br>

## 参考記事

::::details 参考記事一覧

https://qiita.com/chocoyama/items/d69322932f400a5d012b
https://sussan-po.com/2023/11/23/background-task/
https://zenn.dev/iceman/articles/1d71f8dd3a7354
https://zenn.dev/k41531/articles/b6a5d96d65ccc5
https://qiita.com/Sashiiii111/items/763ac2b3ce6c95860f81#4-ios%E3%81%A7%E3%81%AEbg%E5%87%A6%E7%90%86%E3%81%AE%E6%AD%A3%E7%A2%BA%E3%81%AA%E3%82%B9%E3%82%B1%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AA%E3%83%B3%E3%82%B0%E3%81%AF%E7%8F%BE%E7%8A%B6%E3%82%B5%E3%83%9D%E3%83%BC%E3%83%88%E3%81%95%E3%82%8C%E3%81%A6%E3%81%84%E3%81%AA%E3%81%84
https://take4-blue.com/program/flutter-%E5%AE%9A%E6%99%82%E5%87%A6%E7%90%86%E3%81%AE%E5%AE%9F%E8%A3%85%E6%96%B9%E6%B3%95%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6/

github:background_fetch
https://github.com/transistorsoft/flutter_background_fetch/issues/285
https://github.com/transistorsoft/flutter_background_fetch/issues/164

github:workmanager
https://github.com/fluttercommunity/flutter_workmanager/pull/511

::::
