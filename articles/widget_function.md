---
title: "【Flutter】Flutterライブラリ：〇〇builder系のプロパティの指定方法について"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

Flutterライブラリ：〇〇builder系のプロパティの指定方法について、
カスタマイズして使用したかったが、いまいち指定方法が分からず理解が浅かったため、改めて調べてメモする。

今回はサンプルとして"flutter_chat_ui"ライブラリを用いて調べてみた。

# 問題点
〇〇builder系のプロパティの「指定方法、型、実際の挙動」等が理解できてなかったため、思ったような実装ができない。

# 確認
flutter_chat_uiで挙動確認。
```dart
Chat{
  「bubbleBuilderプロパティ」を例として確認する。
}
```


bubbleBuilderプロパティの定義は以下。Widgetを返す関数。
```dart
// ====== flutter_chat_ui/chat.dart ======
  final Widget Function(
    Widget child, {
    required types.Message message,
    required bool nextMessageInGroup,
  })? bubbleBuilder;
```


実際のライブラリ内部で使用されている箇所は以下。
```dart
// ====== flutter_chat_ui/message.dart ======
// ====== 見やすくするために一部削除したり加工してます ======
children: [
  GestureDetector(
    onDoubleTap: () => onMessageDoubleTap?.call(context, message),
    onLongPress: () => onMessageLongPress?.call(context, message),
    onTap: () => onMessageTap?.call(context, message),
    child: VisibilityDetector(
            key: Key(message.id),
            onVisibilityChanged: (visibilityInfo) =>
                onMessageVisibilityChanged!(
              message,
              visibilityInfo.visibleFraction > 0.1,
            ),
            child: _bubbleBuilder(
              context,
              borderRadius.resolve(Directionality.of(context)),
              currentUserIsAuthor,
              enlargeEmojis,
            ),
          )
   ),
],

  Widget _bubbleBuilder(
    BuildContext context,
    BorderRadius borderRadius,
    bool currentUserIsAuthor,
    bool enlargeEmojis,
  ) =>
      bubbleBuilder != null
          ? bubbleBuilder!(
              _messageBuilder(),
              message: message,
              nextMessageInGroup: roundBorder,
            )
```

bubbleBuilder関数の定義をプロパティとして渡してあげれば、その定義を用いてWidgetが返却される流れ。


bubbleBuilderプロパティに渡す用の自作の関数定義を以下に記載。（公式のサンプル引用）
```dart
import 'package:bubble/bubble.dart';

  Widget _bubbleBuilder(
    Widget child, {
    required types.Message message,
    required bool nextMessageInGroup,
  }) =>
    Bubble(
      child: child,
      color: _user.id != message.author.id ||
              message.type == types.MessageType.image
          ? const Color(0xfff5f5f7)
          : const Color(0xff6f61e8),
      margin: nextMessageInGroup
          ? const BubbleEdges.symmetric(horizontal: 6)
          : null,
      nip: nextMessageInGroup
          ? BubbleNip.no
          : _user.id != message.author.id
              ? BubbleNip.leftBottom
              : BubbleNip.rightBottom,
    );
```

実際に呼び出す際は、以下のように記載。
処理の流れも記載。
```dart
Chat{
  bubbleBuilder: _bubbleBuilder
}
```

以下のライブラリ内部の呼び出し箇所と、自作の関数定義箇所とでは、以下のように相関する。
```dart
// ====== flutter_chat_ui/message.dart ======
  Widget _bubbleBuilder(
    BuildContext context,
    BorderRadius borderRadius,
    bool currentUserIsAuthor,
    bool enlargeEmojis,
  ) =>
      bubbleBuilder != null
          ? bubbleBuilder!(                    // 自作関数定義「_bubbleBuilder」での以下に該当
              _messageBuilder(),               // →Widget child
              message: message,                // →types.Message message
              nextMessageInGroup: roundBorder, // →bool nextMessageInGroup
            )
```



# 詰まった点
- bubbleBuilderプロパティに渡すのは、「Widget Function ≒ 関数定義」であって、関数を呼び出して渡すわけでは無いこと注意。
```dart
// NG例
// 括弧をつけると、関数呼び出しとなるので、戻り地(Widget)がプロパティとして渡されてしまう。
Chat{
  bubbleBuilder: _bubbleBuilder() // ←括弧いらない。
}
```

# おわり
ちゃんと調べる前に、カスタマイズできない。と諦めずに、ちゃんとコード読み解くこと。
ちゃんと公式読んでやる前からめげないこと。
安易にパッケージそのものをローカル化して、パッケージ内部を修正しようとしないこと。（戒め。懺悔。）