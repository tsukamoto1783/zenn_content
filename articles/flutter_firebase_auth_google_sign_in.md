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

<br>

## 原因
エラー内容で検索すると、Firebaseの設定にフィンガープリント（SHA-1）なるものを設定する必要があるとのこと。

### 参照
https://riscait.medium.com/apiexception-10-error-in-sign-in-with-google-using-firebase-auth-in-flutter-1be6a44a2086

https://qiita.com/yass97/items/ee502eba6c8aa5a3b809

<br>

## 対応
### フィンガープリント（SHA-1）の取得
参照：https://stackoverflow.com/questions/55496090/how-to-get-sha1-of-android-app-in-vs-code

1. `cd android`
2. `./gradlew signingReport`を実行

```txt: Google Play Servicesより記載引用
// 署名レポートには、アプリの各バリアントの署名情報が含まれます。

> Task :app:signingReport
Variant: debug
Config: debug
Store: ~/.android/debug.keystore
Alias: AndroidDebugKey
MD5: A5:88:41:04:8D:06:71:6D:FE:33:76:87:AC:AD:19:23
SHA1: A7:89:E5:05:C8:17:A1:22:EA:90:6E:A6:EA:A3:D4:8B:3A:30:AB:18
SHA-256: 05:A2:2C:35:EE:F2:51:23:72:4D:72:67:A5:6C:8C:58:22:2A:00:D6:DB:F6:45:D5:C1:82:D2:80:A4:69:A8:FE
Valid until: Wednesday, August 10, 2044
```
https://developers.google.com/android/guides/client-auth?hl=ja

<br>

### 内部テストやクローズドテストなどをdebugモードで配信している場合
Google Play Consoleの[リリース]>[設定]>[アプリの署名]からSHA-1を取得できる。
![](https://storage.googleapis.com/zenn-user-upload/3a7feb4b20d3-20231128.png =600x)

<br>

### Firebaseの設定
アプリの設定画面から、`SHA 証明書フィンガープリント`項目に上記で取得した値を追加。
![](https://storage.googleapis.com/zenn-user-upload/9aeb37ff948e-20231127.png =600x)

<br>

## 備考
### フィンガープリント（SHA-1）とは？
SHA-1（Secure Hash Algorithm 1）
任意の長さのデータを元に、160bit（20byte）の固定長のハッシュ値を生成するハッシュ関数。
データの一意的な識別子として機能し、暗号化、認証、デジタル署名などに利用される。

