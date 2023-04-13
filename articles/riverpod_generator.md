---
title: "riverpod_generatorについて"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# riverpod_generator



# NotifierProviderは、build()が必要。
buildつけろって怒られる。
```text
[SEVERE] riverpod_generator on lib/src/providers/google_map_provider.dart (cached):

No "build" method found. Classes annotated with @riverpod must define a method named "build".
[SEVERE] Failed after 10.0s
pub finished with exit code 1
```

  // NOTE: NotifierProviderの場合、build()が必要
  // 今回のように、ChangeNotifierProviderで実装する場合は何を返す？
  // 特にstateは使用していなかったので、nullでいいか？
  @override
  dynamic build() => null;

# ChangeNotierProviderはgeneratorに対応していない
→どうする？
→ChangeNotifierProviderを使わない実装に変更する必要あるかも。
→riverpod2.0系の導入が必要かも。
ß