---
title: "【Flutter】【Riverpod 1.0系】StateProviderとStateNotifierProviderの違い"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter,Dart,Riverpod]
published: true
---

※詳細は公式ドキュメント参照
(Riverpod)[https://riverpod.dev/ja/docs/getting_started]


# StateProvider の特徴
- 単一のシンプルなstateを管理する際に有効。
  - int, String, boolなど。
- 逆に以下のような場合は使用すべきではない。
  - class, List, Mapなどの複雑なオブジェクト。
  - ステートを変更するためのロジックが単純な count++ よりは高度な場合。
- stateは可変（ミュータブル）であり、外部から値を変更できる。

# StateNotifierProvider の特徴
- 複雑なオブジェクト。（class, Lit, Map 等）
- ステートを変更するための関数等を一元管理するので、保守性を高められる。
- stateは不変（イミュータブル）であるので、外部から値を変更できない。
  - StateNotifierのclass内の、stateを変更する関数で値を変更する。
  （正しくは、stateの「変更」よりは「上書き」がニュアンス的には適切）

# サンプルアプリで見る違い
動作確認しながら違いを見てみる。
↓以下チェックボックスを使用したサンプル
![](/images/checkbox_handling/checkbox2.gif =200x)


## StateProvider
```dart:チェックボックスのサンプルアプリ
class CheckboxListSample extends ConsumerWidget {
  CheckboxListSample({Key? key}) : super(key: key);

  final List<String> _valueList = ['A', 'B', 'C', 'D'];
  // ========== [NG箇所] ==========
  final AutoDisposeStateProvider<List<bool>> _checkedListProvider =
      StateProvider.autoDispose<List<bool>>((ref) {
    return List.generate(4, (index) => false);
  });
  // ==============================
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final List<bool> checkedList = ref.watch(_checkedListProvider);

    return Scaffold(
      appBar: AppBar(title: const Text("StateProvider Sample")),
      body: Center(
        child: ListView.separated(
          itemBuilder: (context, index) => CheckboxListTile(
            title: Text(_valueList[index]),
            subtitle: Text(checkedList[index] ? "ON" : "OFF"),
            value: checkedList[index],
            onChanged: (bool? checkedValue) {
              // ========== [NG箇所] ==========
              // 以下の処理だと、画面は更新されない。
              ref.read(_checkedListProvider.notifier).state[index] =
                  checkedValue!;
              // ==============================
            },
          ),
          itemCount: _valueList.length,
        ),
      ),
    );
  }
}
```
### NG箇所の解説
StateProviderでListを定義するのが、まず良くない。
上記でもエラーは出ないので、仮にStateProviderでListを定義するとしても、例のようにList要素を変更しても変更は検知されない。

StateProviderがListの要素レベルの状態変更を検知できないのは、StateProviderが変更を検知（画面の再生成・リビルド）するのは、stateオブジェクト全体が変更された時のみのため。

List内の要素が変更された場合でも、state（リスト全体）が新しいオブジェクトにならない限り、StateProviderはその変更を検知しない。

<!-- これはStateProviderに限った話ではなく、Riverpodの仕組み上、リストの要素レベルの状態変更を検知することはできない。 -->
<!-- →上記のような記事をちらほら目にするが、公式ドキュメントとしてどこに記載があるのかが見つけられず。どこに記載がある。。？ -->

---------

## StateNotifireProvider
```dart:チェックボックスのサンプルアプリ
final checkedListProvider =
    StateNotifierProvider.autoDispose<CheckedListState, List<bool>>(
  (ref) => CheckedListState(),
);

class CheckedListState extends StateNotifier<List<bool>> {
  // 親クラスであるStateNotifierのコンストラクタをsuperで呼び出し、StateNotifierのstateに初期値を設定する。
  // 以下の場合だと、state=[false, false, false, false]となる。
  // 親クラスのstateを継承するので、CheckedListStateクラスのstateも同じ初期値が設定される。
  CheckedListState() : super(List.generate(4, (index) => false));

  // stateのList[index]の値を、checkedValueに変更し、stateを更新（上書き）する。
  // stateが更新されると、UIが再ビルドされる。
  void updateListElement(int index, bool updateValue) {
    state = [
      for (int i = 0; i < state.length; i++)
        i == index ? updateValue : state[i],
    ];
  }
}

class CheckboxListSample extends ConsumerWidget {
  CheckboxListSample({Key? key}) : super(key: key);
  final List<String> _valueList = ['A', 'B', 'C', 'D'];

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final checkedList = ref.watch(checkedListProvider);

    return Scaffold(
      appBar: AppBar(
          title: const Text("StateNotifierProvider Sample")),
      body: Center(
        child: ListView.separated(
          itemBuilder: (context, index) => CheckboxListTile(
            title: Text(_valueList[index]),
            subtitle: Text(checkedList[index] ? "ON" : "OFF"),
            value: checkedList[index],
            onChanged: (bool? checkedValue) {
              // ========== [NG例] ==========
              // StateNotifirerのstateはimmutableなので、外部から直接変更はできない。
              ref.read(checkedListProvider.notifier).state[index] =
                  checkedValue!;
              // ===========================

              // ========== [OK例] ==========
              // StateNotifierのstateを変更するには、内部で変更が必要。
              // なので、StateNotifierの中で定義したstateを変更するメソッドを呼び出す必要あり。
              ref
                  .read(checkedListProvider.notifier)
                  .updateListElement(index, checkedValue!);
              // ===========================
            },
          ),
          itemCount: _valueList.length,
        ),
      ),
    );
  }
}
```

### NG箇所の解説
コード内のコメントに記載のある通りで、stateは直接外部から変更はできないので、StateNotifierのclass内でstateを更新する処理を実行するようにする。

---------


# おわり
Riverpod2.0系が出て、StateProviderやStateNotifierProviderはNotifierProviderで統一されたので、今後使用する機会は減っていくかも知れない。
けれど、まだまだ既存のコードで目にする機会は多いと思うので、しっかりと違いを把握したい。