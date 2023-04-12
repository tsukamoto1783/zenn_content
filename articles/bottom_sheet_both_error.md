---
title: "【FLutter】Error:'ModalBottomSheetRoute' is imported from both ..."
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter,Dart]
published: true
---

Flutter3.3系から、FLutter3.7系にバージョンアップした際に、以下のエラーが発生。

```
Error: 'ModalBottomSheetRoute' is imported from both 'package:flutter/src/widgets/routes.dart' and 'package:flutter/src/material/bottom_sheet.dart'.
```

# 解決方法
modal_bottom_sheetのバージョンを3.0.0-preに変更することで解決。

```
modal_bottom_sheet: ^3.0.0-pre
```

[modal_bottom_sheet/issues/325:Migration to Flutter 3.7](https://github.com/jamesblasco/modal_bottom_sheet/issues/325)