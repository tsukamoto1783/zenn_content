---
title: "【RevenueCat】OfferingなどのRevenueCat独特の構成を理解する"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [RevenueCat, flutter,ios,android, inAppPurchase,]
published: true
publication_name: ncdc
---
Entitlement、Offering、Package, Productなど、RevenueCat独特の構成？が存在する。
[公式ドキュメント](https://www.revenuecat.com/docs/entitlements)にも詳しく図付きで説明してくれているので分かりやすいが、理解するまで少し時間がかかったので自分なりにざっくりまとめてみた。

※より**詳細に正しく**知りたい方は[公式ドキュメント](https://www.revenuecat.com/docs/entitlements)を参照してください。
※この記事の添付画像はAppleのサブスク製品のみが記載されていますが、GooglePlayのサブスク製品も同様の対応です。

<br>

![](https://storage.googleapis.com/zenn-user-upload/9db517e30b32-20231116.png =500x)
*RevenueCatのコンソールでいう赤枠部分が対象*

<br>

## 全体の構成例
![](https://storage.googleapis.com/zenn-user-upload/5bdee00f8de0-20231117.png =700x)

<br>

## Product
![](https://storage.googleapis.com/zenn-user-upload/fe7043cbe360-20231117.png =350x)

**名前の通り「製品 ≒ サブスク製品」のこと。**

AppStoreConnectやGooglePlayで作成したサブスク製品を全てRevenueCatの`Products`に登録する。

登録する際は、**AppStoreConnectやGooglePlayで作成した製品IDを使用**することで、RevenueCatと紐づく。


![](https://storage.googleapis.com/zenn-user-upload/32c9c492b542-20231117.png)
*RevenueCatコンソール*
 
![](https://storage.googleapis.com/zenn-user-upload/e3e4ef949ef2-20231116.png)
*AppStoreConnect*



<br>

## Entitlement
![](https://storage.googleapis.com/zenn-user-upload/5d56c7795265-20231117.png =400x)

**ユーザーが受け取れる、アクセス、機能、コンテンツのレベルのこと。**


仮に、以下のような課金体系のアプリがある場合、Entitlements定義は2種類必要。
- free版
- pro版（Entitlements：pro）
- premium版（Entitlements：premium）

Productsに登録したサブスク製品を、該当のEntitlementに全てAttachしていく。

![](https://storage.googleapis.com/zenn-user-upload/22c3ac04a584-20231117.png)
*RevenueCatコンソール: Entitlements*

![](https://storage.googleapis.com/zenn-user-upload/32aa19489e4c-20231117.png)
*RevenueCatコンソール: Entitlements > [Entitlement名]*


<br>

## Package
![](https://storage.googleapis.com/zenn-user-upload/ab60ca4e7e60-20231117.png =400x)

**複数プラットフォームの同じレベルの製品をまとめたもの。**

**アプリが複数のプラットフォームで共有する場合、各プラットフォームの同等の製品を1つのPackageにまとめる必要がある。**

<br>

※アプリがWeb版も存在して、アカウントを共有する必要があるなら、Stripなども同様の対応をする。

<br>

RevenueCatのコンソールにはPackages単体の項目はなく、Offeringの設定の中でそれぞれPackageを作成することになる。
以下の[Offerings](#offering)章を参照。


<br>

## Offering
![](https://storage.googleapis.com/zenn-user-upload/c9943adaa666-20231117.png =400x)

**アプリの支払い画面などでユーザーに見せる製品一覧のこと。**
**上記で設定した[Package](#package)を組み合わせることで、特定の条件のユーザーに異なる製品一覧を動的に表示することが可能。**

詳細はこちらも参照：[docs/製品の表示](https://www.revenuecat.com/docs/displaying-products)
> Offeringsを設定した場合、アプリのアップデートを必要とせずにユーザーに表示する商品をコントロールすることができる。
> 様々な商品構成に対応できる動的な支払い画面を構築することで、リモートアップデートの柔軟性を最大限に高めることができる。

<br>

### Best Practices（docs引用）
| Do                                                                       | Don't                                                              |
| ------------------------------------------------------------------------ | ------------------------------------------------------------------ |
| ✅ ハードコードされた文字列を最小化または削除して、支払い画面を動的にする | ❌ 特定のプロダクトIDでハードコードされた静的な支払い画面にする     |
| ✅デフォルトのPackage typesを使用                                         | ❌デフォルト・オプションの代わりにカスタム・パッケージ 識別子を使用 |
| ✅ 任意の数の商品を選択できるようにする                                   | ❌一定の商品数のみをサポートする                                    |
| ✅ 異なる無料トライアル期間、または無料トライアルなしをサポート           | ❌ 無料トライアルテキストをハードコードする                         |

<br>

![](https://storage.googleapis.com/zenn-user-upload/c84dcb41e2c4-20231117.png)
*RevenueCatコンソール: Offering*

![](https://storage.googleapis.com/zenn-user-upload/f351c7814279-20231117.png)
*RevenueCatコンソール: Offering > [Offering名]*

![](https://storage.googleapis.com/zenn-user-upload/1ee701d4d715-20231117.png)
*RevenueCatコンソール: Offering > [Offering名] > [Package名]*


## 備考
どの項目はIdentifierは変更できないので、命名規則はプロジェクトで考えてから作成すること注意。