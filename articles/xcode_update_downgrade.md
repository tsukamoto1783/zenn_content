---
title: "Xcodeのバージョンを下げる方法"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Xcode, build, flutter]
published: true
---


Xcodeのアップデートをすると、毎回必ずと言っていい程ビルドエラーとなり、中々エラー解消できずに時間を溶かすことが多々。

エラーで作業できなくても時間は待ってくれないので、元々正常に動いていたXcodeバージョンへ下げる手順をメモ。

# 0. 発生したビルドエラー
Xcode 14.3にアップデートしてビルドすると、以下のエラーが発生。
```txt
xcodebuild[62639:16214099]
DVTCoreDeviceEnabledState: DVTCoreDeviceEnabledState_Disabled set via user default (DVTEnableCoreDevice=disabled)
```

色々と試したが解消せず、以下のstack overflowのように、バージョンを戻す対応をすることに。（Xcodeのアップデートの嬉しさがあまり感じてなかった＆とりあえず作業を進めたかったため）
https://stackoverflow.com/questions/75902251/flutter-ios-build-failed-dvtcoredeviceenabledstate-disabled

# 1. 使用したいバージョンのXcodeをDL
以下のリンクから、使用したいバージョンのXcodeを検索してDL。
※DL、解凍には結構時間かかるので注意。

https://developer.apple.com/download/all/


![](https://storage.googleapis.com/zenn-user-upload/176bc9948856-20230402.png)

# 2. Xcodeでバージョンの設定
Xcode > Settings > Locations > Command Line Tools から、DLしたXcodeのバージョンを選択
![](https://storage.googleapis.com/zenn-user-upload/12bdea297bd1-20230402.png)



# おわり
「Xcodeのアップデートアラートがしつこいからアップデートしたけど、案の定ビルドエラーに、、、」
「急ぎで作業しないとだめなのに、アップデートによるビルドエラーが解消しない、、、」

って時はエラー解消を諦めてバージョンを下げるのも一つかと。ご参考までに。
