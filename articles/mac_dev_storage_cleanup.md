---
title: "【モバイル開発】Mac PC ストレージが逼迫したときの整理チェックリスト"
emoji: "🧹"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [mac, xcode, android, flutter, ios]
published: true
publication_name: ncdc
---

|                                                                       |                                                                       |
| --------------------------------------------------------------------- | --------------------------------------------------------------------- |
| ![](https://static.zenn.studio/user-upload/b26b1ef1c9a0-20261005.jpg) | ![](https://static.zenn.studio/user-upload/4055fa9f7872-20261005.png) |

開発用のMac PCで、気づいたらストレージの空きが無くなって開発体験が非常に悪くなっていたので、整理の手順や何を消すかをまとめてみました。

Xcode、Android Studio、Flutterなどの開発ツールや、色んなアプリを使っていると、気づかないうちにキャッシュや古いバージョンが数十GB単位で溜まっちゃいます。
特にシミュレーターやエミュレーターは、使っていないものが残っているとかなり無駄に容量を食ってしまいます。

消してよいものと、消す前に確認が必要なものを仕分けしながら整理し、結果的に空きを **200GB程** 確保することができました。

整理する頻度が低く、毎回調べ直すのは手間なので、確認手順をチェックリストとしてまとめます。
ストレージがギリギリだけど、どのフォルダを消してよいか分からない、という方の参考になれば幸いです。

<br>

※ 実行するコマンドリストだけ知りたいよ。
という方は、[こちらのチェックリスト](#コマンドまとめ)からご参照ください。

<br>

## 前提・背景

### 空き容量は実データのボリュームで見る

まずは実データが入っているボリュームを見てみます。

```bash
df -h /System/Volumes/Data
```

以下の項目が出力されます。

| 項目          | 意味                                                                                        |
| ------------- | ------------------------------------------------------------------------------------------- |
| Size          | ディスク全体の容量。同じディスク内の他のボリュームも含む                                    |
| Used          | このボリュームが使っている量                                                                |
| Avail         | 空き容量。今回いちばん見る数字                                                              |
| Capacity      | 使用率。Used ÷ (Used + Avail) で計算される                                                  |
| iused / ifree | 使用中と空きのファイル管理枠（inode）。APFSでは枠が動的に増えるため、通常は気にしなくてよい |

私の環境では、作業時は空きが **537MB** しかありませんでした。。。ClaudeのTaskも途中で止まってしまうぐらい。。。

### 容量を使っている場所を探す

ホームディレクトリ直下を、サイズの大きい順に並べます。

```bash
du -sh -- ~/* ~/.[a-zA-Z]* 2>/dev/null | sort -rh | head -30
```

```
 78G	/Users/<user>/Library      // 整理後の値。ここが約200GBあった...
 15G	/Users/<user>/development
8.9G	/Users/<user>/.gradle
8.4G	/Users/<user>/fvm
7.9G	/Users/<user>/.cache
6.3G	/Users/<user>/.vscode
4.6G	/Users/<user>/.npm
...以下省略
```

<br>

さらに容量の多いフォルダ（例: `Library`）があれば、その配下も同じ方法で調べます。

```bash
du -sh -- ~/Library/*/ 2>/dev/null | sort -rh | head -20
```

```
 33G	/Users/<user>/Library/Application Support/
 23G	/Users/<user>/Library/Developer/
 13G	/Users/<user>/Library/Android/
4.1G	/Users/<user>/Library/Caches/
2.7G	/Users/<user>/Library/Containers/
591M	/Users/<user>/Library/Group Containers/
...以下省略
```

### 整理の方針

- 再生成されるキャッシュは消してよい
- バージョン違いが複数ある場合は、実際に使っているものだけ残す
- 削除コマンドは、一括削除だと怖かったので、対象のファイルやディレクトリごとに、影響を確認してから実行する
- そこまで容量を使っていないフォルダは、費用対効果を考えて、未使用でも削除の対象外とする

<br>

## 1. Xcode / iOS Simulator

### CoreSimulator/Devices

シミュレータのデータは `~/Library/Developer/CoreSimulator/Devices` に入っています。

**①壊れたシミュレータ**
シミュレータは、機種の設定やデータと、動かすためのiOS本体を別々に保存する仕組みです。
Xcodeからランタイムだけを削除すると、そのiOS用に作ったシミュレータのデータは残ったままです。このデータは起動できないため、`unavailable` と表示されます。

起動できないのに容量は使っているので、次のコマンドでまとめて削除します。

```bash
xcrun simctl list devices | grep -c unavailable   # 件数を確認
xcrun simctl delete unavailable                     # 削除
```

<br>

**②使っていないシミュレータ**
使ってないシミュレータは容量泥棒なだけなので消しちゃいましょう。

Xcodeは、新しいiOSランタイムをインストールすると、そのiOS用の標準的な機種（iPhoneやiPadなど）を自動で作ります。
複数のiOSバージョンを入れていると、バージョンごとに機種の数だけシミュレータが増える仕組みです。

実際にPJで使っている機種とiOSの組み合わせだけ残して、あとは削除の対象です。

```bash
xcrun simctl list devices        # 一覧とUUIDを確認
xcrun simctl delete <UUID>       # 指定したシミュレータを削除
```

Xcodeの「Window > Devices and Simulators」からも削除できます。消しても、必要になればXcodeから作り直せます。

※ 私は上記の2対応だけでかなりの容量が回復しました。。。

<br>

### Xcode / iOS DeviceSupport

`~/Library/Developer/Xcode/iOS DeviceSupport/` には、実機をつないだときにXcodeが保存するデータが入っています。

iOS本体のデバッグ用情報（シンボルファイル）は、iOSのバージョン毎に作られる仕組みです。
端末のOSを更新したとしても、古いバージョン分は消えないため、同じ端末で複数のバージョンが並んでしまいます。

同じ端末に複数のバージョンがある場合は、最新のものだけ残して、古いものを削除します。
基本的に実機のOSバージョンは下げられないので、古いバージョンの分は使われないままです。

```bash
du -sh ~/Library/Developer/Xcode/"iOS DeviceSupport"/*/ # 保存されているバージョンとサイズを確認
rm -rf ~/Library/Developer/Xcode/"iOS DeviceSupport"/"iPhone12,1 18.6.2 (22G100)" # 古いバージョンを指定して削除（フォルダ名は例）
```

仮に誤って削除しても、再度実機をつなげばXcodeが再取得します。

<br>

### Xcode / Archives

```bash
rm -rf ~/Library/Developer/Xcode/Archives/*
```

`Archives` は、Xcodeの「Product > Archive」で作った配布用のビルド（`.xcarchive`）の保存先です。
TestFlightやApp Storeへの提出、Ad Hocでの配布に使うビルドが、日付ごとのフォルダに保存されます。

配布用ビルドを作るたびに増え、自動では消えません。
1ファイルはそこまで大きくはないものの、一度アップロードしたファイルを保存しておく意味もあまり無いかなと思うので、全て消しても問題ないと考えています。
（履歴管理の観点はあると思うのでチーム方針に従ってください。）
※ アーカイブの `dSYMs` は、リリース済みアプリのクラッシュログの解析に使います。Crashlyticsなどにアップロード済みか確認してから削除してください。

残したいものがある場合は、日付のフォルダを指定して、不要なものだけ削除してください。

<br>

### DerivedData

```bash
rm -rf ~/Library/Developer/Xcode/DerivedData/*
```

DerivedDataは、Xcodeがビルドの途中結果を保存しているキャッシュです。削除しても、次にビルドするときにXcodeが作り直します。ビルドを続ければまた増えるので根本的な対策にはなりませんが、空きをすぐに増やしたいときには役に立ちます。

<br>

## 2. Android

### AVD

`~/.android/avd/` は、Android Studioで作ったエミュレータの実体が入っている場所です。
1台で10GB近くになるものもあり、似た機種を複数作っていると容量の負担が大きくなりがちです。

実際に使っているエミュレータだけ残し、それ以外は削除します。
必要になれば、Android StudioのDevice Managerから作り直せます。

```bash
du -sh ~/.android/avd/*/   # エミュレータ一覧とサイズ確認
```

※ フォルダだけを削除すると、同じ階層の設定ファイル（`.ini`）が残ることがあるため、Device Managerの「Delete」から行うのが確実です。

<br>

### NDK

`~/Library/Android/sdk/ndk/` には、ネイティブビルドに使うNDKが、バージョン毎に入っています。
プロジェクトごとに要求するバージョンが違うため、古いプロジェクトの分が残って、複数のバージョンが並びがちです。

どのバージョンが使われているかは、次の2点から判断します。

1. プロジェクトの `build.gradle` に `ndkVersion` が明示されているか
2. `flutter.ndkVersion` を使っている場合、使用中のFlutter SDKのデフォルト値

どちらからも参照されていないバージョンが、削除の対象です。

```bash
grep -rn "ndkVersion" ~/development --include="*.gradle" --include="*.gradle.kts"   # プロジェクトが明示しているバージョン
grep -rn "ndkVersion" ~/fvm/versions/*/packages/flutter_tools/gradle/ | grep -iE "default|=\s*\"[0-9]"   # fvmで管理しているFlutter SDKごとのデフォルト値
du -sh ~/Library/Android/sdk/ndk/*/   # 保存されているバージョンとサイズを確認
rm -rf ~/Library/Android/sdk/ndk/25.1.8937393   # 未使用のバージョンを指定して削除（バージョンは例）
```

誤って削除しても、必要になったビルド時に自動で再ダウンロードされます。

<br>

### system-images

`~/Library/Android/sdk/system-images/` には、エミュレータを作るときに使うAndroid OSのイメージが入っています。
AVDがなければ使われないため、AVDを全て削除したなら、丸ごと削除して問題ありません。

```bash
rm -rf ~/Library/Android/sdk/system-images/*
```

AVDを新しく作るときに、必要なバージョンだけ再ダウンロードされます。

<br>

### Gradle / wrapper

`~/.gradle/wrapper/dists/` には、過去に使ったGradle本体が溜まっていきます。
プロジェクトごとに要求するバージョンが違うため、使っていないバージョンが残りがちです。

各プロジェクトが要求しているバージョンと突き合わせて、どこからも使われていないものを削除します。

```bash
find ~/development -name "gradle-wrapper.properties" | xargs grep -h "distributionUrl"   # プロジェクトが要求しているバージョン
du -sh ~/.gradle/wrapper/dists/*/   # 保存されているバージョンとサイズを確認
rm -rf ~/.gradle/wrapper/dists/gradle-8.5-all   # 未使用のバージョンを指定して削除（バージョンは例）
```

サイズが数KBしかないフォルダは、ダウンロードが途中で止まっている可能性があります。使われているバージョンでも、削除して再取得させたほうが確実です。
削除しても、必要になったビルド時に自動で再ダウンロードされます。

<br>

### Gradle / caches・daemon

`~/.gradle/caches/` には依存ライブラリやビルドの中間生成物が、`~/.gradle/daemon/` にはGradleデーモンのログやロックファイルが入っています。

どちらも削除して問題ありません。
`caches` は、次のビルド時にMaven CentralやGoogle Mavenから再ダウンロードされるため、削除後の初回ビルドは時間がかかります。
`daemon` は、削除しても影響ありません。

```bash
rm -rf ~/.gradle/caches/* ~/.gradle/daemon/*
```

<br>

## 3. fvm

### fvm / Flutter SDK

`~/fvm/versions/` には、fvmでインストールしたFlutter SDKが、バージョン毎に入っています。
1バージョンで2〜3GBあり、プロジェクトごとに指定するバージョンが違うため、使わなくなったバージョンが残りがちです。

各プロジェクトの `.fvmrc`（古い形式では `.fvm/fvm_config.json`）と、`fvm global` のバージョンを突き合わせて、どちらからも参照されていないものを削除します。

```bash
du -sh ~/fvm/versions/*/   # 保存されているバージョンとサイズを確認
readlink ~/fvm/default   # globalのバージョン
find ~/development \( -name ".fvmrc" -o -name "fvm_config.json" \) -exec cat {} \;   # プロジェクトが指定しているバージョン
fvm remove 3.29.2   # 未使用のバージョンを指定して削除（バージョンは例）
```

誤って削除しても、必要になれば `fvm install` で再インストールできます。

<br>

## 4. エディタ

以下は、そこまで効果が出ないかもしれないので、必要に応じて実施してください。

### キャッシュ

`~/Library/Application Support/<エディタ名>/` には、設定のほかに、キャッシュやログが溜まりがちです。
次のフォルダは、削除しても再生成されます。

`Cache`、`CachedData`、`CachedExtensionVSIXs`、`logs`、`WebStorage`、`Service Worker`、`Crashpad`

`User` には設定やワークスペースの履歴が入っているため、残します。
エディタを終了してから、以下を実行します。

```bash
du -sh ~/Library/Application\ Support/Code/*/ | sort -rh   # サイズの大きいフォルダを確認（VS Codeの例）
rm -rf ~/Library/Application\ Support/Code/{Cache,CachedData,CachedExtensionVSIXs,logs,WebStorage,Service\ Worker,Crashpad}   # キャッシュ類を削除
```

<br>

## 5. ゴミ箱

色々削除した後は、ゴミ箱も空にするのを忘れずに。
※ `rm -rf` やツールの画面から削除したものはゴミ箱を経由しないため、対象はFinderなどでゴミ箱に移したものです。

<br>

## 結果

私の環境で、空き容量が増えた主な項目は以下のとおりです。

| 項目                               | 削減量   |
| ---------------------------------- | -------- |
| iOS Simulator（unavailable 36台）  | 約31GB   |
| iOS Simulator（使っていない機種）  | 約34GB   |
| Android AVD（全削除）              | 約29.5GB |
| Gradle（wrapper、caches、daemon）  | 約20GB   |
| Android system-images              | 約13GB   |
| iOS DeviceSupport（古い3件）       | 約12.7GB |
| Android NDK（未使用の2バージョン） | 約4.5GB  |
| fvm（未使用の1バージョン）         | 約3.2GB  |

<br>

## コマンドまとめ

実行する前に、対象と影響を確認してください。

**現状把握**

- [ ] 空き容量を確認する: `df -h /System/Volumes/Data`
- [ ] 容量の大きい場所を探す: `du -sh -- ~/* ~/.[a-zA-Z]* 2>/dev/null | sort -rh | head -30`

**Xcode / iOS Simulator**

- [ ] 壊れたシミュレータを削除する: `xcrun simctl delete unavailable`
- [ ] 使っていないシミュレータを削除する: `xcrun simctl list devices` で確認して、`xcrun simctl delete <UUID>`
- [ ] iOS DeviceSupportの古いバージョンを削除する: `du -sh ~/Library/Developer/Xcode/"iOS DeviceSupport"/*/` で確認して、古いフォルダを `rm -rf`
- [ ] Archivesを削除する: `rm -rf ~/Library/Developer/Xcode/Archives/*`
- [ ] DerivedDataを削除する: `rm -rf ~/Library/Developer/Xcode/DerivedData/*`

**Android**

- [ ] 使っていないAVDを削除する: `du -sh ~/.android/avd/*/` で確認して、Device Managerの「Delete」
- [ ] 未使用のNDKを削除する: `du -sh ~/Library/Android/sdk/ndk/*/` で確認し、`ndkVersion` の参照元と突き合わせて、古いフォルダを `rm -rf`
- [ ] system-imagesを削除する（AVDを全て削除した場合）: `rm -rf ~/Library/Android/sdk/system-images/*`
- [ ] 未使用のGradle wrapperを削除する: `du -sh ~/.gradle/wrapper/dists/*/` で確認し、プロジェクトの `gradle-wrapper.properties` と突き合わせて、古いフォルダを `rm -rf`
- [ ] Gradleのキャッシュとデーモンを削除する: `rm -rf ~/.gradle/caches/* ~/.gradle/daemon/*`

**fvm**

- [ ] 未使用のFlutter SDKを削除する: `du -sh ~/fvm/versions/*/` で確認し、`.fvmrc` と `fvm global` のバージョンと突き合わせて、`fvm remove <バージョン>`

**エディタ（必要に応じて）**

- [ ] エディタを終了してから、キャッシュを削除する: `rm -rf ~/Library/Application\ Support/Code/{Cache,CachedData,CachedExtensionVSIXs,logs,WebStorage,Service\ Worker,Crashpad}`

**ゴミ箱**

- [ ] ゴミ箱を空にする: Finderの「ゴミ箱を空にする」、またはコマンド `rm -rf ~/.Trash/*`

<br>

## おわり

- 調査はClaudeに依頼し、削除対象の判断と削除コマンドの実行は、影響を確認したうえで実行しています。
- そこまで頻繁に実施する作業でもないので、次に容量がパンパンになったときのために、チェックリストはドキュメントとしても残しました。
- 他にも削れる場所はありそうですが、私はこれだけで一旦満足できたので、ぜひ試してみてください。

使っているものだけ残す方針で、定期的に見直し・実施していきたいですね🧹
