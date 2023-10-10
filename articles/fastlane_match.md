---
title: "【Flutter/iOS】fastlane matchを使用して各プロジェクトの証明書を一元管理する"
emoji: "🔐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [fastlane,ios,flutter ,profile, certificate]
published: true
publication_name: ncdc
---
[fastlane](https://fastlane.tools)

この記事では、**fastlane match**を使用して、iOSのCertificates（証明書）とProvisioning Profilesを一元管理/自動化する方法を紹介します。

この記事では、fastlane を使用してのデプロイプロセスの自動化までは実施しません。
↑ここは別途記事にできたらと思います。

複数のプロジェクトが既に存在する組織に対して、デプロイ等の自動化まで一度に全て構築するのは大変だと感じます。
まずは煩雑になりがちなCertificate（証明書）周りの一元管理/自動化が構築されるだけで十分fastlaneの恩恵を感じれるかと思っています。

Certificate（証明書）周りの一元管理だけなら最低限のコマンドだけで構築できるので、fastlaneを導入する最初のステップとしては良いのではと思っているポイントです。

今回の**fastlane match**を通じてfastlaneの全体感を掴めてきたなら、デプロイ周りの自動化も検討してみてください。

【備考】
証明書周りの知識についての整理は、以下参照ください。
[ややこしい証明書周りの関係をざっくり整理して理解してみた](https://zenn.dev/tsukatsuka1783/articles/apple_delevoler)

<br>

## fastlane とは？
[fastlane](https://fastlane.tools)とは、iOSとAndroidのビルド、テスト、デプロイを自動化するためのオープンソースツールです。
複数のツールとプロセスを一元化し、コマンド一つで多くの作業を自動化できます。
例えば、証明書の管理、テストの実行、ビルドの生成、App StoreやGoogle Playへのアップロードなどが挙げられます。

## fastlane match とは？
[fastlane match](https://docs.fastlane.tools/actions/match/) とは、fastlaneが持つ機能の一つで、iOSのCertificates（証明書）とProvisioning Profilesの管理を自動化するためのツールです。
この機能を使用することで、Certificates（証明書）とProvisioning Profilesを一元的に管理し、チームメンバーやCI/CDシステムと簡単に共有することができるようになります。

具体的には、fastlane matchは以下のような作業を自動化します：

1. 証明書とプロビジョニングプロファイルの自動生成
2. それらのファイルを暗号化してリモートのGitHubリポジトリに保存
3. 必要に応じて、これらのファイルを自動的に更新または再生成

fastlane match を使用することで、Certificates（証明書）とProvisioning Profilesの手動管理から解放され、それによるエラーや手間を削減することができます。
これは特に大きなチームや複数のプロジェクトを管理している場合に有用です。

fastlane matchの特徴についてより詳しく知るには[fastlane docs](https://docs.fastlane.tools/actions/match/) を参照してください。

<br>

【fastlane matchの全体イメージ】
![](https://storage.googleapis.com/zenn-user-upload/15aee2bed59e-20231003.png)
<br>

## fastlane match の導入
※AWSやAzureなどで管理することも可能ですが、今回はGitHubを使用して管理する前提で説明します。
※fastlaneに関するフォルダ構成については、チームの方針によって適宜変更してください。ここでは一例としてFlutterプロジェクトのフォルダ構成を想定しています。

<br>

1. fastlaneをインストールする。
   - インストールするツールはなんでもいいですが、ここではgemを使用しています。
    `sudo gem install fastlane`

    
      - [参考：fastlane docs/Getting started with fastlane for iOS](https://docs.fastlane.tools/getting-started/ios/setup/)
      - [参考：Flutter docs/fastlane](https://docs.flutter.dev/deployment/cd#fastlane)

2. 一元管理用の空のリポジトリを用意する。
   - シンプルにGitHubからリポジトリを作成するだけでOK。

3. 管理したいプロジェクトのディレクトリで初期化処理。
  `[project]/ios` ディレクトリで、`fastlane init` を実行。
    ↓目的に合わせて2か3を選択。
    ![](https://storage.googleapis.com/zenn-user-upload/f29235c00993-20231003.png)

    ↓開発するチームを選択。
    ![](https://storage.googleapis.com/zenn-user-upload/fa2954a4936d-20231003.png)

    ↓Apple IDを入力。
    ![](https://storage.googleapis.com/zenn-user-upload/09a482c36d0e-20231003.png)

    全て完了すると、以下のフォルダが生成される。
    ```yaml
    project
      ├─ios
        ├─ Gemfile
        ├─ Gemfile.lock
        └─ fastlane
            ├─ Appfile   ≒ fastlane設定ファイル
            └─ Fastfile  ≒ fastlane実行ファイル
    ```

    Appfilesを開くと以下の情報が先ほどの`fastlane init`時の入力通り記載されている。
    この値が一致していることを再度確認する。
    - app_identifier
    - apple_id
    - itc_team_id
    - team_id

1. 管理したいプロジェクトのIdentifier（App ID）を生成する。
   以下の方法などで生成する。
   - `fastlane produce`を使用してIdentifierを生成する。
  produceについての詳細は以下の[「fastlane-produceの使い方」](#fastlane-produceの使い方)参照
   - 従来通り、Apple DeveloperコンソールからIdentifierを生成する。

1. 生成されたfastlaneディレクトリへ移動して、`fastlane match init`を実行する。
    - 実行すると管理用リポジトリのURLを聞かれるので、入力する。
  その後、`[project]/ios/fastlane/Matchfile`が生成される。

    - Matchfileに記載のあるURLに間違いないか念のため確認する。
    `git_url("リポジトリURL")`

    ![](https://storage.googleapis.com/zenn-user-upload/946d3e0e2d36-20231003.png)

1. fastlaneディレクトリで、`fastlane match ××××`を実行する。
    **※以下、管理元のリポジトリでCertificate（証明書）の保持はまだしていない（空）と想定して記載する。**
    - `××××`は、`development` or `appstore` or `adhoc` or `enterprise` のいずれかを指定する。
    - これで証明書とプロビジョニングプロファイルが自動生成され、生成されたファイルは暗号化されて管理用リポジトリに保存される。
    - 初回matchに自動生成されたCertificate（証明書）は、一度登録されたら有効期限が切れるとか意外は基本的には再生成されない。
    この管理されたCertificate（証明書）を元に各プロジェクトのProvisioning Profilesが生成される。

<br>

## 導入後の使用方法
新規プロジェクト参画者（開発メンバー）を想定して以下記載する。
1. プロジェクトのリポジトリをCloneする。
2. [project_path]/fastlaneディレクトリ配下で、`fastlane match ×××× --readonly`を実行する。
3. 実行後、ローカルにp12とprofileがDLされる。

以上のみで、すぐに開発が開始できる。

【備考】
`Appfile`で指定のあるApple IDのパスワードを知らない場合や、チーム所属でないApple IDが指定された場合などは、開発者は、`--readonly`しか実行することができす、証明書等の更新はできない。
≒プロジェクトやチームの管理者のみが証明書の更新は可能とすることができる。

<br>

## その他ポイント
### fastlane produceの使い方
produceコマンドを使用すると、「Apple DeveloperのIdentifier」と、「App Store Connectにアプリ（提出準備中）」が作成できる。
`fastlane produce`を実行すると、app_nameを聞かれるので回答するだけで完了。これで上記二つが自動生成される。
![](https://storage.googleapis.com/zenn-user-upload/8229e6e6e077-20231003.png)
※App Name: 「Apple DeveloperのIdentifierのDescription部分」と、「App Store Connectのタイトル（アプリ名）」のこと。


Bundle IDなどの情報は、自動生成されたAppfileファイルに記載のある値がデフォルトで使用される。
変更したい場合はオプションで指定可能。

Apple DeveloperのIdentifierだけ生成したい場合は、`--skip_itc`オプションを指定して実行すれば、App Store Connectへの生成はされない。

※注意
[fastlane docs/produce](https://docs.fastlane.tools/actions/produce/)を見る限り、produceコマンドからだとIdentifierのCapabilityの指定が出来なさそう。
なので、push通知とか設定をしたい場合は結局Apple Developerから手動で変更しないといけない。また、デフォルトだとGameCenterにチェックが入ってしまっていることも注意。
（この辺もコマンドで指定できる情報がないか調べてみる。。）

<br>

### 既存のCertificate（証明書）を使用できる？
`fastlane match import`を実行すると、Certificate（証明書）とプロビジョニングプロファイルを管理用リポジトリにインポートすることができる。

ただし、基本的には管理リポジトリで管理されるCertificate（証明書）は一つだけであることに注意。

上記を踏まえると、もしimportを使用するなら、タイミングは組織全体でfastlane導入時かCertificate（証明書）の期限が切れた時だけかと。

管理リポジトリの中身が空の状態で、証明書をインポートすることで管理リポジトリに証明書が登録される。
そして、このインポートした証明書が組織の基準証明書となる。

一応importすると証明書を重複して管理することは可能だが、結局`fastlane match ××××`を実行してもどちらか基準の証明書だけが参照されることになる。また、match実行時に証明書を切り替えれる記述はドキュメントには記載がない。

もし、複数のチームのCertificate（証明書）を同じリポジトリで管理したい場合は、fastlaneのブランチ機能を使って切り替えれるとの記載はあるので試してみてください。[fastlane docs/match/Multiple teams](https://docs.fastlane.tools/actions/match/)
（組織ごとに管理リポジトリを新たに立てる方法も考えられそう。）

<br>

### 証明書の期限が切れた場合はどうする？
`nuke`コマンドで削除してください。
  
`nuke`コマンドで Certificate（証明書）を削除する場合は、その証明書に付随するProfileも全て削除されるので便利。
ただ、割と破壊的なコマンドなので、**実行する際は注意が必要。**
![](https://storage.googleapis.com/zenn-user-upload/f2a10833f3fd-20231003.png)

管理者だけが実行できる。などのルールにした方が安全そう。

<br>

### Identifier（App ID）を作成せずにfastlane match init実行するとどうなる？
以下のようなログが出てProfileが生成されません。
```
 ==========================================
 Could not find App ID with bundle identifier '<Bundle ID>'
 You can easily generate a new App ID on the Developer Portal using 'produce':
 
 fastlane produce -u <user address> -a <Bundle ID> --skip_itc
 
 You will be asked for any missing information, like the full name of your app
 If the app should also be created on App Store Connect, remove the --skip_itc from the command above
==========================================
An app with that bundle ID needs to exist in order to create a provisioning profile for it
==========================================
```

<br>

### 手動でApple DeveloperコンソールからProfileを削除したらどうなる？
【試した事】
1. 既に`fastlane match development`でprofileが正常に生成されているものとする。
2. Apple Developerコンソールから、profileを手動で削除。
3. 再度、`fastlane match development`を実行すると、再生成してくれる。
```
Installing provisioning profile...
Provisioning profile '<profile_id>' is not available on the Developer Portal for the user <user_address>, fixing this now for you 
...省略
No existing profiles found, that match the certificates you have installed locally! Creating a new provisioning profile for you
Creating new provisioning profile for '<Bundle ID>' with name 'match Development <Bundle ID>' for 'ios' platform
...省略
```

<br>

### Deviceを追加/削除したが更新される？
通常の`fastlane match ×××××`だけだと更新されない。オプションで`--force_for_new_devices`を指定するとデバイス情報の変更を検知して更新（再生成）してくれる。

デフォルトで常にforce_for_new_devicesオプションは指定する習慣をつけていた方が確実そう。

※`force`オプションと`force_for_new_devices`オプションでは動作が異なること注意。

| option                | detail                                                                                                                                           |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| force                 | the provisioning profiles every time you run match                                                                                               |
| force_for_new_devices | Renew the provisioning profiles if the device count on the developer portal has changed. Ignored for profile types 'appstore' and 'developer_id' |

## おわり
ここでは基本的な使用方法を主に記載しました。
実際にはチーム毎にこのオプションを指定して実行する。などが出てくると思います。

この記事では主にコマンドベースでの`fastlane match`の使用でしたが、fastlaneの実行ファイルにmatchを組み込めばデプロイの自動化もすぐに構築できるかと思いますので、ぜひ検討してみてください。

<br>

::::details 参考
https://zenn.dev/tsukatsuka1783/articles/apple_delevoler
https://qiita.com/kotarella1110/items/840af2cf80aaea1fb035
::::
