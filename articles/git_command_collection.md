---
title: "よく使うgitコマンド"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [git]
published: false
---

個人的によく使うgitコマンド集。

一つの例なので、より詳細なコマンドや詳細はググって調べて見てください。

# git add して、「ステージング」されたファイルを取り消す
```
git reset <ファイル名>
```
git add した状態を「ステージング」「ステージ」と言う。

# ローカルで直前にコミットした内容を取り消す
```
git reset --soft HEAD^
```

# 直前のプッシュを取り消したい
1. git reset：履歴が残らずに修正できる
2. git revert：履歴を残して修正する

【git reset】
```
git reset --soft HEAD^
```
```
git push origin HEAD
```
【git revert】
※こっちはあまり使ったことないから挙動の把握はしてない。
```
git revert HEAD
```
```
git push origin HEAD
```

# 特定の操作履歴に戻る
```
git reflog
```
```
git reset --hard < コミットID > or < HEAD@{数字} >
```

# ブランチを作成して、そのブランチに移動する
```
git checkout -b <ブランチ名>
```

# 変更差分を確認しながら、ステージングする
```
git add -p <ファイル名>
```
使用する主なオプションは以下。
- y：ステージングする
- n：ステージングしない
- e：エディタで差分を確認する



# 参考
https://www.r-staffing.co.jp/engineer/archive/category/%E3%83%9E%E3%83%B3%E3%82%AC%E3%83%BBGit