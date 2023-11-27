---
title: "【Flutter】Androidのdebugモード時に、FirebaseAuthのGoogleサインインするとsign_in_failed"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, firebaseAuth, android, googleSignIn]
published: true
---
## エラー内容
以下の条件時に、`sign_in_failed`が発生。
- android
- debugモード
- Firebase Auth
- Googleサインイン
 
`Unhandled Exception: PlatformException(sign_in_failed, com.google.android.gms.common.api.ApiException: 10: , null, null)`

## 原因
エラー内容で検索すると、Firebaseの設定にフィンガープリント（SHA-1）なるものを設定する必要があるとのこと。

### 参照
https://riscait.medium.com/apiexception-10-error-in-sign-in-with-google-using-firebase-auth-in-flutter-1be6a44a2086

https://qiita.com/yass97/items/ee502eba6c8aa5a3b809


## 対応
### フィンガープリント（SHA-1）の取得
参照：https://stackoverflow.com/questions/55496090/how-to-get-sha1-of-android-app-in-vs-code

1. `cd android`
2. `./gradlew signingReport`を実行

```txt
// 以下のような形式で出力される

> Task :app:signingReport
Variant: <実行名>
Config: <debug or releaseとか>
Store: <keystoreファイルのパス>
Alias: <alias名>
MD5: <hash値>
SHA1: <hash値>
SHA-256: <hash値>
Valid until: <有効期限>
```
### Firebaseの設定
アプリの設定画面から、`SHA 証明書フィンガープリント`項目に上記で出力した値を追加。
![](https://storage.googleapis.com/zenn-user-upload/9aeb37ff948e-20231127.png =600x)

## 備考
### フィンガープリント（SHA-1）とは？
SHA-1（Secure Hash Algorithm 1）
任意の長さのデータを元に、160bit（20byte）の固定長のハッシュ値を生成するハッシュ関数。
データの一意的な識別子として機能し、暗号化、認証、デジタル署名などに利用される。

