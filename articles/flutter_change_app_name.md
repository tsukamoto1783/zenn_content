---
title: "【Flutter】プロジェクトのアプリ名を変更する"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, Dart, Android, iOS]
published: true
---
アプリ名とアプリIDを変更する際の手順を記載する。

確認用にアプリを作成する。
```
flutter create sample_app
```

&nbsp;
# android対応
1. [change_app_package_name](https://pub.dev/packages/change_app_package_name)を使用してアプリ名とアプリIDを一括で変更する。
以下のコマンドと、変更後のアプリ名等を指定して実行すると、androidに関するアプリ名とアプリIDが一括で変更される。

```
flutter pub run change_app_package_name:main com.tmp.new_sample_app

========= console出力結果 =========
Old Package Name: com.example.sample_app
Updating build.gradle File
Updating Main Manifest file
Updating Debug Manifest file
Updating Profile Manifest file
Project is using kotlin
Updating MainActivity.kt
Creating New Directory Structure
Deleting old directories
```

&nbsp;
「/android/app/build.gradle」の変更例。
```diff app/build.gradle
android {
-    namespace "com.example.sample_app"
+    namespace "com.tmp.new_sample_app"
```

&nbsp;
2. iconの表示名を変更する。
「/android/app/src/main/AndroidManifest.xml」のlabel名を変更。
```diff 
    <application
-        android:label="sample_app"
+        android:label="new_sample_app"
```

&nbsp;
# iOS対応
1. 「/ios/Runner.xcodeproj/project.pbxproj」の「PRODUCT_BUNDLE_IDENTIFIER」箇所を全て変更する。
```diff project.pbxproj
- PRODUCT_BUNDLE_IDENTIFIER = com.example.sampleApp;
+ PRODUCT_BUNDLE_IDENTIFIER = com.tmp.newSampleApp;
```

&nbsp;
2. iconの表示名を変更する。
「/ios/Runner/Info.plist」の「CFBundleDisplayName」箇所を変更。
```diff Info.plist
	<key>CFBundleDisplayName</key>
-	<string>Sample App</string>
+	<string>New Sample App</string>
```


&nbsp;
# 注意
- アプリID（bundle identifier）について、
androidは、アンダースコア（ _ ）はOKだが、ハイフン（ - ）は使用不可。
iOSは、アンダースコア（ _ ）は使用不可で、ハイフン（ - ）はOK。

- flavorを使用している場合は、flavor毎のアプリ名等も変更すること忘れず。
以下、「Dart-define-from-file」を使用してflavor分けしている例。
```diff 
- "appName": "sample app",
+ "appName": "new sample app",
```