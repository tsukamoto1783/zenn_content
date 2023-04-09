---
title: "【Flutter】checkboxの状態管理をいろんなパターンで試す"
emoji: "✅"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter,Dart,Checkbox,Riverpod,Stateful]
published: true
---
チェックボックスの状態管理をいろんなパターンで試してみた。
正解とか不正解とかではなく、こういった方法でも実装できるよねって感じで。

# 【Stateful】
## ・シンプルなCheckbox
サンプル通りのシンプルなチェックボックスをStatefulで。

::::details コード
```dart
class CheckboxSingle extends StatefulWidget {
  const CheckboxSingle({Key? key}) : super(key: key);

  @override
  State<CheckboxSingle> createState() => _CheckboxSingleState();
}

class _CheckboxSingleState extends State<CheckboxSingle> {
  bool _isChecked = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text("checkbox sample")),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(_isChecked ? "ON" : "OFF"),
            Checkbox(
              value: _isChecked,
              onChanged: (value) {
                setState(() {
                  _isChecked = !_isChecked;
                });
              },
            ),
          ],
        ),
      ),
    );
  }
}
```
::::
![](/images/checkbox_handling/checkbox.gif =200x)

## ・Checkboxを複数表示（List）
ListViewで複数のチェックボックスを表示。
値はListで管理。

::::details コード
```dart
class CheckboxListTiles extends StatefulWidget {
  const CheckboxListTiles({Key? key}) : super(key: key);

  @override
  State<CheckboxListTiles> createState() => _CheckboxListTilesState();
}

class _CheckboxListTilesState extends State<CheckboxListTiles> {
  final List<String> _valueList = ['A', 'B', 'C', 'D'];
  final List<bool> _checkedList = List.generate(4, (index) => false);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text("CheckboxListTile List Ver")),
      body: Center(
        child: ListView.separated(
          itemBuilder: (context, index) => CheckboxListTile(
            title: Text(_valueList[index]),
            subtitle: Text(_checkedList[index] ? "ON" : "OFF"),
            value: _checkedList[index],
            onChanged: (bool? checkedValue) {
              setState(() {
                _checkedList[index] = checkedValue!;
              });
            },
          ),
          separatorBuilder: (context, index) {
            return const Divider(height: 0.5);
          },
          itemCount: _valueList.length,
        ),
      ),
    );
  }
}
```
::::
![](/images/checkbox_handling/checkbox2.gif =200x)

## ・Checkboxを複数表示（Map）
.map()で複数のチェックボックスを表示。
値はList<Map>で管理。（動作はListパターンと一緒）

::::details コード
```dart
class CheckboxListTilesVer2 extends StatefulWidget {
  const CheckboxListTilesVer2({Key? key}) : super(key: key);

  @override
  State<CheckboxListTilesVer2> createState() => _CheckboxListTilesVer2();
}

class _CheckboxListTilesVer2 extends State<CheckboxListTilesVer2> {
  final List<Map<String, dynamic>> _checkedMaps = [
    {'value': 'A', 'checked': false},
    {'value': 'B', 'checked': false},
    {'value': 'C', 'checked': false},
    {'value': 'D', 'checked': false},
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text("CheckboxListTile Map Ver")),
      body: Center(
        child: Column(
          children: _checkedMaps
              .map((e) => CheckboxListTile(
                    title: Text(e['value']),
                    subtitle: Text(e['checked'] ? "ON" : "OFF"),
                    value: e['checked'],
                    onChanged: (bool? checkedValue) {
                      setState(() {
                        e['checked'] = checkedValue;
                      });
                    },
                  ))
              .toList(),
        ),
      ),
    );
  }
}
```
::::
![](/images/checkbox_handling/checkbox2.gif =200x)

---------------

# 【Riverpod】
※挙動は全てStatefulの時と同じのためgifは割愛

## ・シンプルなCheckbox
サンプル通りのシンプルなチェックボックスをRiverpod:StateProviderで。

::::details コード
```dart
class CheckboxRiverpod extends ConsumerWidget {
  CheckboxRiverpod({Key? key}) : super(key: key);

  final AutoDisposeStateProvider<bool> _isCheckedProvider =
      StateProvider.autoDispose((ref) {
    return false;
  });

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final bool isChecked = ref.watch(_isCheckedProvider);

    return Scaffold(
      appBar: AppBar(title: const Text("Checkbox + Riverpod")),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(isChecked ? "ON" : "OFF"),
            Checkbox(
              value: isChecked,
              onChanged: (bool? checkedValue) {
                ref.read(_isCheckedProvider.notifier).state = checkedValue!;
              },
            ),
          ],
        ),
      ),
    );
  }
}
```
::::

## ・Checkboxを複数表示（List）
ListViewで複数のチェックボックスを表示。
値はListで管理。

::::details コード
```dart
class CheckboxListTileRiverpod extends ConsumerWidget {
  CheckboxListTileRiverpod({Key? key}) : super(key: key);
  final List<String> _valueList = ['A', 'B', 'C', 'D'];
  final AutoDisposeStateProvider<List<bool>> _checkedListProvider =
      StateProvider.autoDispose<List<bool>>((ref) {
    return List.generate(4, (index) => false);
  });

  final ProviderListenable<dynamic> _changeNotifierProvider =
      ChangeNotifierProvider((ref) {
    return ChangeNotifier();
  });

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final List<bool> checkedList = ref.watch(_checkedListProvider);
    final changeNotifier = ref.watch(_changeNotifierProvider);

    return Scaffold(
      appBar: AppBar(title: const Text("Checkbox + Riverpod List Ver")),
      body: Center(
        child: ListView.separated(
          itemBuilder: (context, index) => CheckboxListTile(
            title: Text(_valueList[index]),
            subtitle: Text(checkedList[index] ? "ON" : "OFF"),
            value: checkedList[index],
            onChanged: (bool? checkedValue) {
              checkedList[index] = checkedValue!;
              changeNotifier.notifyListeners();
            },
          ),
          separatorBuilder: (context, index) {
            return const Divider(height: 0.5);
          },
          itemCount: _valueList.length,
        ),
      ),
    );
  }
```
::::

## ・Checkboxを複数表示（Map）
.map()で複数のチェックボックスを表示。
値はList<Map>で管理。（動作はListパターンと一緒）

::::details コード
```dart
class CheckboxListTileRiverpodVer2 extends ConsumerWidget {
  CheckboxListTileRiverpodVer2({Key? key}) : super(key: key);
  final AutoDisposeStateProvider<List<Map<String, dynamic>>>
      _checkedMapStateProvider =
      StateProvider.autoDispose<List<Map<String, dynamic>>>((ref) {
    return [
      {'value': 'A', 'checked': false},
      {'value': 'B', 'checked': false},
      {'value': 'C', 'checked': false},
      {'value': 'D', 'checked': false},
    ];
  });

  final ProviderListenable<dynamic> _changeNotifierProvider =
      ChangeNotifierProvider((ref) {
    return ChangeNotifier();
  });

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final List<Map<String, dynamic>> checkedMap =
        ref.watch(_checkedMapStateProvider);
    final changeNotifier = ref.watch(_changeNotifierProvider);

    return Scaffold(
      appBar: AppBar(title: const Text("Checkbox + Riverpod Map Ver")),
      body: Center(
        child: Column(
          children: checkedMap
              .map((e) => CheckboxListTile(
                    title: Text(e['value']),
                    subtitle: Text(e['checked'] ? "ON" : "OFF"),
                    value: e['checked'],
                    onChanged: (bool? checkedValue) {
                      e['checked'] = checkedValue;
                      changeNotifier.notifyListeners();
                    },
                  ))
              .toList(),
        ),
      ),
    );
  }
}
```
::::

-------------------
# コメント
- 状態を格納する変数をListにするかMapにするかは、実際使用するデータ構造によって使い分けか。
  今後他にもパターンがあれば追記していく。
- ~~Riverpodの複数パターンの時、値の更新をnotifyListeners()で行っているのが個人的にはもやもや、、
  StateProviderだけで完結させたかったが、Listの要素の値変更は上手く調整しないと監視されないみたい。
  ここをもうちょっと調べていい感じにしてみる。~~

# 追記
Riverpodでの複数パターンの時、そもそもStateProviderの使用が良くなかった。。
ListやMapを扱う場合は、以下の公式のドキュメントの文言より、StateNotifireProviderを使用する。
```
StateProvider は次のようなステートを公開するために使うべきではありません。
・ステート自体が複雑なオブジェクトである（カスタムのクラスや List/Map など）
```
## ・【修正版】Checkboxを複数表示（List）
ListViewで複数のチェックボックスを表示。
値は、ListをStateNotifireProviderで管理。
Listの中身が更新※されたら、画面が再生成される。

※正しくは、「更新」ではなく、「List自体を上書き」が表現としては正しいか。
→[参考記事]()

::::details コード
```dart
final checkedListProvider = StateNotifierProvider<CheckedListState, List<bool>>(
  (ref) => CheckedListState(),
);

class CheckedListState extends StateNotifier<List<bool>> {
  CheckedListState() : super(List.generate(4, (index) => false));

  void updateCheckedValue(int index, bool checkedValue) {
    state = [
      for (int i = 0; i < state.length; i++)
        i == index ? checkedValue : state[i],
    ];
  }
}

class CheckboxListTileRiverpodNew extends ConsumerWidget {
  CheckboxListTileRiverpodNew({Key? key}) : super(key: key);
  final List<String> _valueList = ['A', 'B', 'C', 'D'];

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final checkedList = ref.watch(checkedListProvider);
    return Scaffold(
      appBar: AppBar(title: const Text("Checkbox + Riverpod List Ver")),
      body: Center(
        child: ListView.separated(
          itemBuilder: (context, index) => CheckboxListTile(
            title: Text(_valueList[index]),
            subtitle: Text(checkedList[index] ? "ON" : "OFF"),
            value: checkedList[index],
            onChanged: (bool? checkedValue) {
              ref.watch(checkedListProvider.notifier).updateCheckedValue(index, checkedValue!);
            },
          ),
          separatorBuilder: (context, index) {
            return const Divider(height: 0.5);
          },
          itemCount: _valueList.length,
        ),
      ),
    );
  }
}

```
::::



# 参考記事
::::details 参考記事
↓ListViewについて
https://www.choge-blog.com/programming/fluttercheckbox-listvoew/

https://zenn.dev/t_fukuyama/articles/0d3e6bfd3881bd

https://qiita.com/code-cutlass/items/3a8b759056db1e8f7639

↓Riverpod周りの参考記事
https://zenn.dev/aomi/articles/19b0b5d016b5fd

https://zenn.dev/shu_illy/articles/d43e842ea9ad4e

https://zenn.dev/taku_zenn/articles/e416eab158ae0f

https://core-tech.jp/blog/tech_log/3069/

<!-- 【List・Mapの取り扱い】 -->
<!-- https://www.choge-blog.com/programming/dartmapupdatebalue/ -->
<!-- https://www.choge-blog.com/programming/dart-list-elementcount/ -->
<!-- https://www.choge-blog.com/programming/dart-map%EF%BC%88%E3%83%9E%E3%83%83%E3%83%97%EF%BC%89%E3%81%AEforeach%E3%83%A1%E3%82%BD%E3%83%83%E3%83%89%E3%81%A8%E3%81%AF%EF%BC%9F/ -->
