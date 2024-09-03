---
title: "【Flutter】【android】background_fetchライブラリでバックグラウンドタスクを扱う"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, android, backgroundFetch, background, dart]
published: false
publication_name: ncdc
---

[background_fetch](https://pub.dev/packages/background_fetch)

ios についてはこの記事ではあまり言及しません。
iOS については制約が多いため、別記事にまとめました。

基本的な動作確認は sample を動かすと全体イメージはつく。
android での設定を中心に、実装方法を記載します。

## setup

[README](https://github.com/transistorsoft/flutter_background_fetch/blob/master/help/INSTALL-ANDROID.md) 参照に、記載通りに設定を行う。

## init 処理

まずは init 処理。
BackgroundFetch.configure を呼び出して、バックグラウンドでの処理を設定する。

```dart
void init() {
  BackgroundFetch.configure(
    BackgroundFetchConfig(
      minimumFetchInterval: 15,
      stopOnTerminate: false,
      enableHeadless: true,
      startOnBoot: true,
      requiresBatteryNotLow: false,
      requiresCharging: false,
      requiresStorageNotLow: false,
      requiresDeviceIdle: false,
      requiredNetworkType: NetworkType.NONE,
    ),
    ), (String taskId) async {  // <-- Event handler
      // This is the fetch-event callback.
      print("[BackgroundFetch] Event received $taskId");
      setState(() {
        _events.insert(0, new DateTime.now());
      });
      // IMPORTANT:  You must signal completion of your task or the OS can punish your app
      // for taking too long in the background.
      BackgroundFetch.finish(taskId);
    }, (String taskId) async {  // <-- Task timeout handler.
      // This task has exceeded its allowed running-time.  You must stop what you're doing and immediately .finish(taskId)
      print("[BackgroundFetch] TASK TIMEOUT taskId: $taskId");
      BackgroundFetch.finish(taskId);
    });
  );
      // If the widget was removed from the tree while the asynchronous platform
    // message was in flight, we want to discard the reply rather than calling
    // setState to update our non-existent appearance.
    if (!mounted) return;

}
```

BackgroundFetchConfig の各パラメータについての詳細は API ドキュメントを参照してください。

※ 必須のプロパティは `minimumFetchInterval` のみです。
※ `minimumFetchInterval`以外のプロパティは、**全て android のみ有効なプロパティです**。

| プロパティ            | 説明（翻訳引用）                                                                                                                                                                                                                                                                                                                                                                                                                         |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| minimumFetchInterval  | バックグラウンド・フェッチ・イベントを実行する最小間隔を分単位で指定する。<br>デフォルトは 15 分である。<br>注意：バックグラウンドフェッチイベントは、15 分以上の頻度で発生することはありません。Apple はフェッチイベントの頻度を調整するために秘密のアルゴリズムを使っています、 おそらくアプリの使用パターンに基づいていると思われます。フェッチイベントは、設定した `minimumFetchInterval` よりも少ない頻度で発生することがあります。 |
| stopOnTerminate       | ユーザーがアプリを終了した後もバックグラウンド・フェッチ・イベントを継続するには false を設定します。 デフォルトは true です。                                                                                                                                                                                                                                                                                                           |
| startOnBoot           | デバイスの再起動時にバックグラウンド・フェッチ・イベントを開始するには true を設定する。 デフォルトは false。                                                                                                                                                                                                                                                                                                                            |
| enableHeadless        | アプリ終了後のフェッチイベントを処理するヘッドレスメカニズムを有効にするには true を設定します。                                                                                                                                                                                                                                                                                                                                         |
| forceAlarmManager     | タスクが JobScheduler ではなく Android AlarmManager メカニズムを使用するようにするには true を設定します。 デフォルトは false です。 より正確なタスクのスケジューリングが可能になりますが、その代償としてバッテリーの使用量が増えます。                                                                                                                                                                                                  |
| requiredNetworkType   | あなたの仕事に必要なネットワークの種類を詳しく説明する。                                                                                                                                                                                                                                                                                                                                                                                 |
| requiresBatteryNotLow | このジョブを実行するには、デバイスのバッテリー残量が少なくなっていないことを指定する。                                                                                                                                                                                                                                                                                                                                                   |
| requiresStorageNotLow | このジョブを実行するには、デバイスの利用可能なストレージが少なくなってはならないことを指定する。                                                                                                                                                                                                                                                                                                                                         |
| requiresCharging      | このジョブを実行するには、デバイスが充電中（または Android TV デバイスのような常時電源に接続された非バッテリー駆動デバイス）でなければならないことを指定します。 デフォルトは false です。                                                                                                                                                                                                                                               |
| requiresDeviceIdle    | true を設定すると、デバイスがアクティブに使用されている場合、このジョブが実行されないようにします。                                                                                                                                                                                                                                                                                                                                      |

基本的な機能としては、この設定だけで OK。
第二引数に指定した callback 関数が、バックグラウンド状態で定期的に実行される。

上記の基本的な使用方法以外に、3 つの大きな機能が提供されている。

## scheduleTask

> 任意の "ワンショット "または定期的なタスクをスケジューリングするための scheduleTask メソッドを提供するようになりました。

> Executing Custom Tasks
> BackgroundFetch.configure によって定義されたデフォルトのバックグラウンド・フェッチ・タスクに加えて、任意の "oneshot "タスクや定期的なタスクを実行することもできます（iOS は追加のセットアップ手順が必要です）。 しかし、**すべてのイベントは BackgroundFetch#configure に提供されたコールバックに発射されます**：

ボタンの Tap イベント等で、バックグラウンド処理をフォアグランドで実行することもできる。

```dart
  void _onClickButton() async {
      // Schedule a "one-shot" custom-task in 10000ms.
      // These are fairly reliable on Android (particularly with forceAlarmManager) but not iOS,
      // where device must be powered (and delay will be throttled by the OS).
      BackgroundFetch.scheduleTask(TaskConfig(
          taskId: "com.transistorsoft.customtask",
          delay: 10000,
          periodic: true,
          forceAlarmManager: true,
          stopOnTerminate: false,
          enableHeadless: true));
  }
```

TaskConfig の各プロパティについての詳細は API ドキュメントを参照してください。

| プロパティ                  | 説明（翻訳引用）                                                                                                                 |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| taskId                      | ユニークなタスク ID。 この `taskId` は、[BackgroundFetch.finish] で使用するために BackgroundFetch コールバック関数に提供される。 |
| delay                       | このタスクが起動するミリ秒数。                                                                                                   |
| periodic                    | このタスクを繰り返し実行するかどうかを制御する。 デフォルトは `false` である。                                                   |
| type                        | DEFAULT or HEALTH_RESEARCH                                                                                                       |
| requiresNetworkConnectivity | [iOS のみ] このタスクがネットワーク接続を必要とする場合、`true`を設定する。か。                                                  |

## Headless Task

> Android プラグインはヘッドレス実装を提供し、アプリ終了後もイベントの処理を継続することができます。

完全にアプリを終了した状態でバックグラウンド処理を行うことも可能。
ヘッドレスタスクを有効にするには、`enableHeadless: true` を設定する。

```dart
// [Android-only] This "Headless Task" is run when the Android app is terminated with `enableHeadless: true`
// Be sure to annotate your callback function to avoid issues in release mode on Flutter >= 3.3.0
@pragma('vm:entry-point')
void backgroundFetchHeadlessTask(HeadlessTask task) async {
 String taskId = task.taskId;
 bool isTimeout = task.timeout;
 if (isTimeout) {
   // This task has exceeded its allowed running-time.
   // You must stop what you're doing and immediately .finish(taskId)
   print("[BackgroundFetch] Headless task timed-out: $taskId");
   BackgroundFetch.finish(taskId);
   return;
 }
 print('[BackgroundFetch] Headless event received.');
 // Do your work here...

 // デフォルトのバックグラウンド処理以外に処理を走らせたい場合は、スケジュールタスクを設定することも可能。
  if (taskId == 'flutter_background_fetch') {
    BackgroundFetch.scheduleTask(TaskConfig(
        taskId: "com.transistorsoft.customtask",
        delay: 5000,
        periodic: false,
        forceAlarmManager: false,
        stopOnTerminate: false,
        enableHeadless: true));
  }

 BackgroundFetch.finish(taskId);
}

void main() {
  // Enable integration testing with the Flutter Driver extension.
  // See https://flutter.io/testing/ for more info.
  runApp(new MyApp());

  // Register to receive BackgroundFetch events after app is terminated.
  // Requires {stopOnTerminate: false, enableHeadless: true}
  BackgroundFetch.registerHeadlessTask(backgroundFetchHeadlessTask);
}

```

<br>

## Precise event-scheduling with forceAlarmManager: true

より正確なイベントスケジューリングを使用する場合は、`forceAlarmManager: true` を設定することができる。

> forceAlarmManager: true でイベントの正確なスケジューリングを使用したい場合のみ、Android 14（SDK 34）では、「AlarmManager exact alarms」の使用が制限されています。 Android 14 でイベントの正確なタイミングを使用し続けるには、AndroidManifest にこの権限を手動で追加してください。 そうしないと、プラグインは "正確な AlarmManager スケジューリング "にフォールバックします：

> ⚠️ It has been announced that Google Play Store has plans to impose greater scrutiny over usage of this permission (which is why the plugin does not automatically add it).

> デフォルトでは、プラグインは可能な限り Android の JobScheduler を使用します。 JobScheduler API はデバイスの使用量とバッテリー残量に基づいてタスクの実行を調整し、バッテリー残量を優先します。
> forceAlarmManager:を true に設定すると、Android の古い AlarmManager API を使うために JobScheduler をバイパスします。

[JobScheduler](https://developer.android.com/reference/android/app/job/JobScheduler) がデフォルトでは適用されているので、条件次第ではバックグラウンドタスクが実行されないこともある。
それを強制的に実行するためには、`forceAlarmManager: true` を設定する。

ただし、あまり推奨ではなさそう。
審査が厳しくなるとの記載もあるし、そもそも古い AlarmManager API を使用することになるので。
