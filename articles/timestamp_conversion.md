---
title: "【Flutter】DateTimeの変換処理"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, Dart]
published: false
---

※DateTimeを日本語日付へ変換する際は、基本的にはintlライブラリが必要な認識
import 'package:intl/intl.dart';



DateTimeを「◯月◯日 hh:mm」形式に変換
```dart
String convertFormattedDate(DateTime dateTime){
  String date = DateFormat.MMMd('ja_JP').add_Hm().format(dateTime),
  return data;
}

// もしくは以下。
String convertFormattedDate(DateTime dateTime){
  String date = DateFormat('yyyy年M月 HH:mm').format(dateTime),
  return data;
}

```

エポック秒(UnixTime)の日付をDateTime変換処理
```dart
const String defaultFormat = 'yyyy/MM/dd HH:mm';

String convertFormattedDate(int epoch, [String? format]) {
  DateTime date = DateTime.fromMillisecondsSinceEpoch(epoch);
  DateFormat outputFormat = DateFormat(format ?? defaultFormat);
  return outputFormat.format(date);
}
```

文字列の日付をDateTimeに変換
```dart
const String defaultFormat = 'yyyy/MM/dd HH:mm';

String convertFormattedDateString(String dateString,  [String? format]) {
  DateTime date = DateTime.parse(dateString);
  DateFormat outputFormat = DateFormat(format ?? defaultFormat);
  return outputFormat.format(date);
}
```

FireStoreで保存してる値：Timestampは、.toDate()でDateTimeへ変換。
上記、反転Verは、 Timestamp.fromDate(data);



# 参考記事
https://api.flutter.dev/flutter/intl/DateFormat-class.html

https://note-tmk.hatenablog.com/entry/2022/09/22/092807

https://chaika.hatenablog.com/entry/2021/04/18/083000

https://zenn.dev/taji/articles/d1d94b5efbed35

https://codechacha.com/ja/dart-add-time-to-datetime/