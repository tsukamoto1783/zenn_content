---
title: "【Flutter】loggerなどでデバッグコンソールに出力される文字に色がつかない"
emoji: "🖍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, logger, dart, debugconsole, vscode]
published: true
publication_name: ncdc
---

Flutter のログライブラリでメジャーな[logger](https://pub.dev/packages/logger) などを使用すると、デバッグコンソールでのログ出力が種類ごとに色がつくようになる。

![](https://storage.googleapis.com/zenn-user-upload/04f0ae337d3a-20240310.png)
_logger の pub.dev から引用_

<br>

しかし、基本の設定をして実際に動かしてみると Android だとデフォルトで問題なくログ毎に色分けされているが、Ios だと色がつかない。
Ios でも色がつくようにしたい。

|                                    Android                                     |                                      Ios                                       |
| :----------------------------------------------------------------------------: | :----------------------------------------------------------------------------: |
| ![](https://storage.googleapis.com/zenn-user-upload/9eb2e1970057-20240310.gif) | ![](https://storage.googleapis.com/zenn-user-upload/7e0fe0809f55-20240310.gif) |

<br>

## 検証環境

エディタは VSCode を使用。

サンプルのアプリは以下のようなボタンが並んでるだけの画面を想定。

![](https://storage.googleapis.com/zenn-user-upload/14c248f9f519-20240313.png =300x)

冒頭にある gif も以下のサンプルコードで作成したもの。

::: details サンプルコード

```dart
import 'package:flutter/material.dart';
import 'package:logger/logger.dart';

void main() {
  runApp(const MyApp());
}

Logger logger = Logger();

class LogButton {
  final Color color;
  final String label;
  final VoidCallback onPressed;

  LogButton(this.color, this.label, this.onPressed);
}

final buttons = [
  LogButton(
    Colors.grey,
    "Level.trace",
    () => logger.t("Log a message at level [Level.trace]."),
  ),
  LogButton(
    Colors.yellow,
    "Level.debug",
    () => logger.d("Log a message at level [Level.debug]."),
  ),
  LogButton(
    Colors.blue,
    "Level.info",
    () => logger.i("Log a message at level [Level.info]."),
  ),
  LogButton(
    Colors.orange,
    "Level.warning",
    () => logger.w("Log a message at level [Level.warning]."),
  ),
  LogButton(
    Colors.red,
    "Level.error",
    () => logger.w("Log a message at level [Level.error]."),
  ),
  LogButton(
    Colors.purpleAccent,
    "Level.fatal",
    () => logger.f("Log a message at level [Level.fatal]."),
  ),
];

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
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: const Text('logger debug console sample'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: buttons
              .map((button) => ElevatedButton(
                    style: ElevatedButton.styleFrom(
                      backgroundColor: button.color,
                      minimumSize: const Size(160, 40),
                    ),
                    onPressed: button.onPressed,
                    child: Text(
                      button.label,
                      style: const TextStyle(
                        color: Colors.black,
                        fontSize: 16,
                      ),
                    ),
                  ))
              .toList(),
        ),
      ),
    );
  }
}
```

:::

<br>

## 解決策

[logger の Issue](https://github.com/simc/logger/issues/1)に記載があるように、下記のように記載したら、Ios でも色がつくようになる。

```dart
import 'dart:developer' as developer;
import 'dart:io';

import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:logger/logger.dart';

void main() {
  runApp(const MyApp());
}

Logger logger = Logger(output: ConsoleOutput());

class ConsoleOutput extends LogOutput {
  @override
  void output(OutputEvent event) {
    if (kReleaseMode || !Platform.isIOS) {
      event.lines.forEach(debugPrint);
    } else {
      event.lines.forEach(developer.log);
    }
  }
}

class LogButton {
    // 以下、サンプルコードと同じのため省略。
```

![](https://storage.googleapis.com/zenn-user-upload/1b9ba2c4f059-20240310.gif)

<br>

## 参考

https://github.com/SourceHorizon/logger/issues/18

https://github.com/flutter/flutter/issues/64491
