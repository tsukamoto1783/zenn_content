---
title: "【Flutter】uncaught exception 'NSInvalidArgumentException'"
emoji: "🗺️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, dart, ios, googleMap]
published: true
---

# エラー内容
`google_maps_flutter`ライブラリの`GoogleMap`Widgetを使用している画面へ遷移して、`.pop()`で戻った際に以下のエラーが発生してアプリが落ちてしまう。

エラー発生時のflutter versionは`3.10.1`

```text
*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '*** -[__NSArrayM insertObject:atIndex:]: object cannot be nil'
*** First throw call stack:
(0x1c759de48 0x1c08738d8 0x1c7743088 0x1c773fab0 0x1c75a7f50 0x10904ab4c 0x1093f4f24 0x1093f59d4 0x1092fa864 0x1093f4880 0x1093f6950 0x1093f5c20 0x1093f6608 0x1092f8e9c 0x1092fcc98 0x1c766233c 0x1c761e9b8 0x1c75c2558 0x1c760ffb0 0x1c7614ec0 0x20166b368 0x1c9b0a86c 0x1c9b0a4d0 0x10476c958 0x1e5e36960)
libc++abi: terminating with uncaught exception of type NSException
* thread #1, queue = 'com.apple.main-thread', stop reason = signal SIGABRT
    frame #0: 0x0000000204f0f160 libsystem_kernel.dylib`__pthread_kill + 8
libsystem_kernel.dylib`:
->  0x204f0f160 <+8>:  b.lo   0x204f0f180               ; <+40>
    0x204f0f164 <+12>: pacibsp
    0x204f0f168 <+16>: stp    x29, x30, [sp, #-0x10]!
    0x204f0f16c <+20>: mov    x29, sp
Target 0: (Runner) stopped.
Lost connection to device.
```


# 解決方法
flutter versionを`3.13.0`以降にアップデートすることで解消された。

[Issue: app crashes on iOS Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '*** -[__NSArrayM insertObject:atIndex:]: object cannot be nil'](https://github.com/flutter/flutter/issues/131999)
