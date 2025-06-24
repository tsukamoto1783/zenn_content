---
title: "【Swift】URLComponentsのqueryにパーセントエンコード済みの値を渡したい"
emoji: "📤"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Swift, URLComponents, URL, percentEncode, encode]
published: false
# publication_name: ncdc
---

# 問題

[URLComponents](https://developer.apple.com/documentation/foundation/urlcomponents) を使って [URL](https://developer.apple.com/documentation/foundation/url) を生成する際、クエリパラメータにパーセントエンコード済みの文字列を渡すと、意図せず 二重エンコードされてしまいます。

コンソールで確認してみると、URL を生成する際に **URLComponents が自動的にパーセントエンコードを行っており、意図しない文字列（2 重エンコード）となってしまっていました。**

ex.
`/` → `%2F` → `%252F`

```swift
func createURL() -> URL {
    let baseURL = "https://example.com"
    let parameters = [
        "id": "22",
        "date": "2025%2F01%2F01", // encode済み文字列：2025/01/01
        "text": "aa%2Baa%3Daa%25", // encode済み文字列：aa+aa=aa%
    ]

    var components = URLComponents(string: baseURL)!
    components.queryItems = parameters.map { URLQueryItem(name: $0.key, value: $0.value) }
    print("🐛components.queryItems: \(components.queryItems ?? [])")

    let componentsUrl = components.url!
    print("🐛componentsUrl: \(componentsUrl)")
    return componentsUrl
}
```

```txt
🐛components.queryItems: [id=22, text=aa%2Baa%3Daa%25, date=2025%2F01%2F01]
🐛componentsUrl:         https://example.com?id=22&text=aa%252Baa%253Daa%2525&date=2025%252F01%252F01 // ← 2重エンコードとなっている
```

<br>

# 対応策

[percentEncodedQuery](https://developer.apple.com/documentation/foundation/urlcomponents/percentencodedquery) を使って、エンコード済みの クエリ文字列をセットすると、**指定したクエリ文字列をそのまま最終的なクエリパラメータとして扱うことができます**。

```swift
func createURL() -> URL {
    let baseURL = "https://example.com"
    let parameters = [
        "id": "22",
        "date": "2025%2F01%2F01", // encode済み文字列：2025/01/01
        "text": "aa%2Baa%3Daa%25", // encode済み文字列：aa+aa=aa%
    ]

    var components = URLComponents(string: baseURL)!
    components.queryItems = parameters.map { URLQueryItem(name: $0.key, value: $0.value) }
    print("🐛components.queryItems: \(components.queryItems ?? [])")

+    // クエリ文字列を percentEncodedQuery にセットする
+    components.percentEncodedQuery = components.query!

    let componentsUrl = components.url!
    print("🐛componentsUrl: \(componentsUrl)")
    return componentsUrl
}
```

```txt
🐛components.queryItems:  [date=2025%2F01%2F01, text=aa%2Baa%3Daa%25, id=22]
🐛componentsUrl:          https://example.com?date=2025%2F01%2F01&text=aa%2Baa%3Daa%25&id=22 // ←2重エンコードが解消
```

<br>

# 備考・補足

上記に至るまでに色々と調べたので、その過程や確認した内容を以下に記載していきます。

<br>

## URLComponents とは

URL 情報 を構成する各要素（host や query 等）を解析・取得したり、各要素から URL オブジェクトを構築したりするための構造体です。
`RFC 3986` という規格に基づいて生成されます。

[URLComponents](https://developer.apple.com/documentation/foundation/urlcomponents) 構造体 とは別に、[URL](https://developer.apple.com/documentation/foundation/url) 構造体 もありますが、これは少し古い RFC 規格が対応されているみたいです。
URL 構造体よりもより柔軟に URL 情報を取り扱えるようになったのが、 URLComponents 構造体 だと考えていただけると大旨イメージは合っているかと思います。

※ 以降、"...構造体" の記載は省きます。

<br>

## URLComponents の基本的な挙動

以下のような簡単な URL を URLComponents を使って生成してみます。
URLComponents から URL オブジェクトを生成する際には、自動的にパーセントエンコード（URL エンコード）が行われます。

```swift
func createURL() -> URL {
    let baseURL = "https://example.com"
    let parameters = [
        "id": "22",
        "date": "2025/01/01",
        "text": "あい/う/ /え+お-%",
    ]

    var components = URLComponents(string: baseURL)!
    components.queryItems = parameters.map { URLQueryItem(name: $0.key, value: $0.value) }
    print("🐞components.queryItems: \(components.queryItems ?? [])")

    let componentsUrl = components.url!
    print("🐞componentsUrl: \(componentsUrl)")
    return componentsUrl
}
```

```txt
🐞components.queryItems: [id=22, date=2025/01/01, text=あい/う/ /え+お-%]
🐞componentsUrl:         https://example.com?id=22&date=2025/01/01&text=%E3%81%82%E3%81%84/%E3%81%86/%20/%E3%81%88+%E3%81%8A-%25
```

<br>

パーセントエンコードは、`RFC 3986`に基づいてエンコードされます。
ここで注意したいのは、**特殊文字等のエンコードの挙動について**です。

上記のサンプルでは、`あい/う/ /え+お-%`という文字列は、`%E3%81%82%E3%81%84/%E3%81%86/%20/%E3%81%88+%E3%81%8A-%25`にエンコードされました。

`/ +`の記号はそのまま表示され、`<空白> %`はエンコードされている違いがあります。
こちらの違いについては、`RFC 3986`の規定に基づいています。

<br>

## RFC 3986

`RFC 3986`とは、**URI（URL）の**一般的な構文と目的についての**標準仕様です**。

https://tex2e.github.io/rfc-translater/html/rfc3986.html

ドキュメントの内容は膨大なので、今回は必要な部分だけ抜粋 & 要約とします。
ここで抑えておきたい部分は、パーセントエンコード周り（[2. Characters](https://tex2e.github.io/rfc-translater/html/rfc3986.html#2--Characters)）についてです。

**【パーセントエンコード対象文字についてのまとめ】**

- **非予約文字: エンコードしない**

  - 非予約文字は、URI の構文上で特別な意味を持たず、いつでもそのまま URI に含めることができる文字です。これらはパーセントエンコードされる必要はありません。
  - 英大文字: `A-Z`
  - 英小文字: `a-z`
  - 数字: `0-9`
  - ハイフン: `-`
  - ピリオド: `.`
  - アンダースコア: `_`
  - チルダ: `~`

- **予約文字: エンコードしない**
  - 予約文字は、URI の構文上で区切り文字などの特別な意味を持つ文字のため、エンコードしません。
  - **エンコード対象**（特別な意味を持たない文字列の一部）**としたい場合は、US-ASCII[^1] を元に、プログラム上で明示的にエンコードする必要があります。**
    - 予約文字の例: `: / ? # [ ] @ ! $ & ' ( ) * + , ; =`
- **その他の文字・特殊文字（スペース、%、日本語などのマルチバイト文字）: 常にエンコードする**

> 2.2. Reserved Characters
> URI の生産アプリケーションは、これらの文字が URI スキームによってそのコンポーネントのデータを表すために特別に許可されない限り、予約セットの文字に対応するパーセントエンコードデータオクテットが必要です。
> **予約済みの文字が URI コンポーネントに見られる場合、その文字では区切りの役割が知られていない場合、US-ASCII でのそのキャラクターのエンコードに対応するデータを表すと解釈する必要があります。**

<br>

以下は [「URL エンコードツール」](https://develop.tools/percent-encoding/)で文字列：`あい/う/ /え+お-%`をエンコードした結果です。
`%E3%81%82%E3%81%84%2F%E3%81%86%2F%20%2F%E3%81%88%2B%E3%81%8A-%25`

理想としては、URLComponents 内でも、よしなに上記のようにパーセントエンコードしてほしいですが、そうはいきません。

<br>

## 予約文字を含むクエリ文字列の取り扱いについて

上記で見た通り、URLComponents では、文字列内に含まれる予約文字についても意味のある文字だと認識して、エンコードしてくれません。

なので、先にクエリの文字列をエンコードしてから、URLComponents にセットする方法も１つです。
文字列に対するエンコード処理は、[addingPercentEncoding(withAllowedCharacters:)](<https://developer.apple.com/documentation/foundation/nsstring/addingpercentencoding(withallowedcharacters:)>)で行うことができます。

```swift
extension String {
    func strictQueryEncoded() -> String? {
        // urlQueryAllowedから '+' や '/' などを除外します。
        var customQueryAllowed = CharacterSet.urlQueryAllowed
        customQueryAllowed.remove(charactersIn: "+/")

        // カスタムの CharacterSet を使用して文字列をエンコードします。
        return self.addingPercentEncoding(withAllowedCharacters: customQueryAllowed)
    }
}

func createURL() -> URL {
    let baseURL = "https://example.com"
    let parameters = [
        "id": "22",
        "date": "2025/01/01",
        "text": "あい/う/ /え+お-%",
    ]

    var components = URLComponents(string: baseURL)!
    components.queryItems = parameters.map { URLQueryItem(name: $0.key, value: $0.value) }
    // この時点では予約文字のエンコードがされてないことを確認
    print("🐛components.percentEncodedQueryItems: \(components.percentEncodedQueryItems ?? [])")

    // カスタムのエンコード設定に基づき、エンコードしたクエリ文字列をセット
    components.percentEncodedQuery = components.query?.strictQueryEncoded()

    let componentsUrl = components.url!
    // 指定の予約文字もエンコードされていることを確認
    print("🐛componentsUrl: \(componentsUrl)")
    return componentsUrl
}
```

```txt
🐛components.percentEncodedQueryItems: [date=2025/01/01, id=22, text=%E3%81%82%E3%81%84/%E3%81%86/%20/%E3%81%88+%E3%81%8A-%25]
🐛componentsUrl: https://example.com?date=2025%2F01%2F01&id=22&text=%E3%81%82%E3%81%84%2F%E3%81%86%2F%20%2F%E3%81%88%2B%E3%81%8A-%25

```

<br>

## その他

1.`percentEncodedQueryItems` の挙動について。
`percentEncodedQuery`でエンコード済みのクエリ文字列をセットすると、`percentEncodedQueryItems` もセットしたクエリ文字列を返します。
セットしていない状態で`percentEncodedQueryItems`を出力確認すると、 URL として返却される予定のクエリ文字列が返却されます。

```swift
func createURL() -> URL {
    let baseURL = "https://example.com"
    let parameters = [
        "id": "22",
        "date": "2025%2F01%2F01", // encode済み文字列：2025/01/01
        "text": "aa%2Baa%3Daa%25", // encode済み文字列：aa+aa=aa%
    ]

    var components = URLComponents(string: baseURL)!
    components.queryItems = parameters.map { URLQueryItem(name: $0.key, value: $0.value) }
    print("🐛components.queryItems: \(components.queryItems ?? [])")

    // components.urlで結果的にエンコードされる値が出力される → 2重エンコードされていることを確認
    print("🐛components.percentEncodedQueryItems 1: \(components.percentEncodedQueryItems ?? [])")

    // クエリ文字列を、percentEncodedQuery にセットする
    components.percentEncodedQuery = components.query!

    // セットした値が出力される
    print("🐛components.percentEncodedQueryItems 2: \(components.percentEncodedQueryItems)")

    let componentsUrl = components.url!
    print("🐛componentsUrl: \(componentsUrl)")
    return componentsUrl
}
```

```txt
🐛components.queryItems:                 [id=22, date=2025%2F01%2F01, text=aa%2Baa%3Daa%25]
🐛components.percentEncodedQueryItems 1: [id=22, date=2025%252F01%252F01, text=aa%252Baa%253Daa%2525]
🐛components.percentEncodedQueryItems 2: Optional([id=22, date=2025%2F01%2F01, text=aa%2Baa%3Daa%25])
🐛componentsUrl:                         https://example.com/api?id=22&date=2025%2F01%2F01&text=aa%2Baa%3Daa%25
```

<br>

2.URL オブジェクトを使用した場合、挙動に差異が発生。

以下のような基本な URL オブジェクトを生成すると、二重エンコードは発生しませんでした。

```swift
func onPressed() {
    let createdURL = createURL()
    print("🐞created URL: \(createdURL)")
}

func createURL() -> URL {
    let baseURL = "https://example.com"
    let parameters = [
        "id": "22",
        "date": "2025%2F01%2F01", // encode済み文字列：2025/01/01
        "text": "aa%2Baa%3Daa%25", // encode済み文字列：aa+aa=aa%
    ]

    let urlString = baseURL + "?" + parameters.map { "\($0.key)=\($0.value)" }.joined(separator: "&")
    print("🐞urlString: \(urlString)")
    return URL(string: urlString)!
}
```

```txt
🐞urlString:   https://example.com/api?id=22&text=aa%2Baa%3Daa%25&date=2025%2F01%2F01
🐞created URL: https://example.com/api?id=22&text=aa%2Baa%3Daa%25&date=2025%2F01%2F01
```

<br>

しかし、parameters に Base64 の画像文字列のような長い文字列をセットすると、URLComponents の挙動と同じように、二重エンコードが発生してしまいます。
（原因不明 & 詳細は未調査）

```swift
func onPressed() {
    let createdURL = createURL()
    print("🐞created URL: \(createdURL)")
}

func createURL() -> URL {
    let baseURL = "https://example.com"
    let parameters = [
        "id": "22",
        "date": "2025%2F01%2F01", // encode済み文字列：2025/01/01
        "text": "aa%2Baa%3Daa%25", // encode済み文字列：aa+aa=aa%
        "image": loadImage(), // エンコード済みのBase64文字列（以下別途添付）
    ]

    let urlString = baseURL + "?" + parameters.map { "\($0.key)=\($0.value)" }.joined(separator: "&")
    print("🐞urlString: \(urlString)")
    return URL(string: urlString)!
}

// エンコード済みの適当な画像ファイルを読み込む
func loadImage() -> String {
    if let path = Bundle.main.path(forResource: "test", ofType: "txt"),
       let content = try? String(contentsOfFile: path, encoding: .utf8) {
        return content
    }
    return ""
}
```

```txt
【Base64のエンコード済み画像文字列】
iVBORw0KGgoAAAANSUhEUgAAAGAAAAA4CAYAAAACRf2iAAABXGlDQ1BJQ0MgUHJvZmlsZQAAKJF1kLtLQgEUxn%2BWFYhQg0NBg0M09TAzalULERzElB7QcL2aBj4u1xvR1lB7UEtTz6G%2FoJaG1mgwCBokorU5cqjkdq5WWtGBw%2Ffj45zDx4E2p6JpOTuQLxh6LBRwzy8surue6MSFg1FQ1JLmj0YjMsKX%2FqzqHTZLb4etW5XJ%2FfLhVd%2F14NL%2BycSra%2Bzv%2FI9ypNIlVfRd2qdqugE2j3B0zdAs3hB26RJKeNfiTINPLU42%2BKI%2BE48FhW%2BEe9SskhJ%2BFB5KtviZFs7nVtXPDFZ6Z7qQmBXtle4nQgg3CdE4MfzMEWaamX92fPWdIEU01tFZIUMWQy74xdHIkRYOU0BlhCFhLx7pCevXv3%2FY9IpHMPUC7dtNL7kH51sSs9L0Bg6gexPOypqiK9%2BftVXtpeVxb4OdAeh4MM3nQejagdq2ab4dmWbtWO7fw2XhAzcfZTDtpGhfAAAAVmVYSWZNTQAqAAAACAABh2kABAAAAAEAAAAaAAAAAAADkoYABwAAABIAAABEoAIABAAAAAEAAABgoAMABAAAAAEAAAA4AAAAAEFTQ0lJAAAAU2NyZWVuc2hvdAHDxO8AAAHUaVRYdFhNTDpjb20uYWRvYmUueG1wAAAAAAA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJYTVAgQ29yZSA2LjAuMCI%2BCiAgIDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI%2BCiAgICAgIDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiCiAgICAgICAgICAgIHhtbG5zOmV4aWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20vZXhpZi8xLjAvIj4KICAgICAgICAgPGV4aWY6UGl4ZWxZRGltZW5zaW9uPjU2PC9leGlmOlBpeGVsWURpbWVuc2lvbj4KICAgICAgICAgPGV4aWY6UGl4ZWxYRGltZW5zaW9uPjk2PC9leGlmOlBpeGVsWERpbWVuc2lvbj4KICAgICAgICAgPGV4aWY6VXNlckNvbW1lbnQ%2BU2NyZWVuc2hvdDwvZXhpZjpVc2VyQ29tbWVudD4KICAgICAgPC9yZGY6RGVzY3JpcHRpb24%2BCiAgIDwvcmRmOlJERj4KPC94OnhtcG1ldGE%2BCgFOdpwAAAWPSURBVHgB7VtLSFVdFF7qRQJfKKIN0lDxhSCYReHIcGBIDnyGoqQIilaigyYVPiA0kkINhTRTMcwe2iDBAl%2FpyHQSCA6URFOQEB%2F4IDX171t0D1e7nbstz79%2F%2BPeCc%2B45e6291l7ft9fax4F23t7e%2B6REGgL20iKrwIyAIkDyRlAEKAIkIyA5vKoARYBkBCSHVxWgCJCMgOTwqgIUAZIRkBxeVYAiQDICksOrClAESEZAcnhVAYoAyQhIDq8qQBEgGQHJ4VUFKAIkIyA5vKoARYBkBCSHVxWgCJCMgOTwqgL%2BiwT4%2Bvr%2Bdll6ut9OkqA4znVmZmbSgwcPOIuEhASqra09toxM165do7q6OnYYFBREd%2B%2FeJQcHB7Kzs6O%2Bvj4hnchq0tLSKCUlhT5%2B%2FEj37t3TnXLlyhXCdVgs1wPdzZs36cKFC%2FTo0SMaHBxkc70c2OAPbiDz9OnTPPPUqVOE67jEPiYmhoKDg9nf7du36dOnT5SUlMSMi%2BpsLQYLhs%2BlpSUm1pb9ixcvKDExUbsKCwt5ysjIiDYVY%2BfPn6fS0lINfCh%2Fl4O%2Fvz%2FV19dTdHS05sP8oKeDzdraGn3%2F%2Fp3NV1ZWaG9vzzz1r3%2F5DLh8%2BTKdPHmSXFxctB3v6OjIzm3pYOTl5cVl2dnZSa9fv6aamhry8%2FPTFldeXk4fPnyg%2Bfl5bQwPZWVlBLAx59mzZ3Tx4sUDevNLdnY2IfGxsTEewpoAZH9%2FP21ubprNdHP4%2FPkzDQ8PU05ODnV0dFBeXh7ni8l6OuhXV1dpe3sbj7wOMxk88OOGCnn58uUftSbTwsICeXp6ko%2BPD%2B3v73OA4uJiOnfuHInosIiqqireFQ8fPqSvX7%2FS1atXKSQkhKanpyk3N5fs7e25TYAIS3n16hUntrW1RVlZWTxvYGDA0oQ8PDwoIiKC55sV6enp9O3bNwoNDSVUKcgBsHo5YO7z58%2F5Onv2LMFHS0sLvXv3jhobG3V1ra2thAvy9u1bvvjl5w2b1WQykbOzs%2BWw0LPJ3BYAPgRlimcsEOcBzgI9XUBAAO%2BkiooKbYfeuXOHfUEXGxtLJSUl%2FI5FosrQkubm5mh2dpbi4uJ4B5krkA0tbmg1GxsbZEnMiRMnCOPwAUHloSLW19f53VoOrPh529nZod3dXX7DmixFT2dpZ%2Fk8NTXF7dJyTPTZhPYxPj7OyQDsL1%2B%2BUGVlJc8X0ZlbFXbhYYmKiuJE0achSBZkolJQGU%2BePOG4o6OjvIsvXbp0wMWP%2F12g8PBwam5u1sbd3d352c3NTSMABJ05c4ba29t5w1jLAZOSk5MpPj6enJyc%2BGMAlYuKtaVjA4Nu9mg%2F3d3d3G6QiKurK4eKjIzk1mRLNzExwe0nPz9fm4skAVxbWxulpqZqF3Y8%2BjjG8FWB1lRUVEQ9PT0UFhb2ywF9%2Ffp1bjUoe7MsLy8zaBkZGdo6Ufpv3rzRzSEwMJB3aW9vLxNx%2F%2F59DXw9nTmu3i8quqmpiW7cuKFnZlVnev%2F%2BPR9C0GJHoH10dXWxsaiuurqaWwJ6KnY4enpDQ8MvAS2%2FHvC1hfaHWDjUhoaG%2BBDFYQaCsPtBCs6Jw%2FL48WMCOWg9aCVoTzhvIHo5oK1ak8nJSW651nQiYzgvUZmLi4si5gds7Kz9ixJ258zMzAFD84stHT7ZAOy%2FITg38KFgTfTWac3%2Bb8aePn1KOJfwZYX8jyJWCTiKg%2F%2B7Lf4QLCgooFu3bmln0lEwUQQcBS0DbPkPMQP8KpeCCCgCBIEyykwRYBSygn4VAYJAGWWmCDAKWUG%2FigBBoIwyUwQYhaygX0WAIFBGmSkCjEJW0K8iQBAoo8wUAUYhK%2BhXESAIlFFmigCjkBX0qwgQBMooM0WAUcgK%2BlUECAJllJkiwChkBf0qAgSBMsrsH11zn4f6ff2tAAAAAElFTkSuQmCC%0A
```

```txt
🐞urlString:   https://example.com/api?text=aa%2Baa%3Daa%25&date=2025%2F01%2F01&id=22&image=iVBO...Uxn%2BWFYh...Tz6G%2FoJ...
🐞created URL: https://example.com/api?text=aa%252Baa%253Daa%2525&date=2025%252F01%252F01&id=22&image=iVBO...Uxn%252BWFYh...Tz6G%252FoJ...
```

# 参考記事

:::details 【参考記事】
https://medium.com/ios-ic-weekly/the-problem-of-percent-encoding-on-ios-ebadd39e8b6f
https://developer.apple.com/forums/thread/738631
https://swapnil-adnak.medium.com/double-encoding-issue-in-urlcomponents-8d6aa3498cff
https://qiita.com/417_72ki/items/47ef002586b15ed0d5ed
https://qiita.com/ta9yamakawa/items/99eebefac79bd344cbac
https://qiita.com/hyuga_amazia/items/dd754ba1185c28e12a5f
https://telecom-engineer.blog/blog/2023/05/13/url/
https://zenn.dev/ymasutani/articles/b995a605dff3ff
:::

---

[^1]: `US-ASCII`: 文字コード規格の `ASCII` と**ほぼ**同義。ASCII の元祖的な位置付け。
