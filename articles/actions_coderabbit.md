---
title: "CodeRabbit（ai-pr-reviewer）を試す際に詰まったポイント"
emoji: "🔍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [CodeRabbit, GitHub, GitHubActions, ChatGPT, OpenAI]
published: true
---
以下の話題になっていたCodeRabbitを実際に触ってみようとして詰まったポイントを記載。
https://zenn.dev/minedia/articles/7928ef7545b393

# 1.「CodeRabbit」と「ai-pr-reviewer」とを混同しない
[CodeRabbit](https://coderabbit.ai/)は、GitHub Appとして動作するサービス。「≒ CodeRabbitのPro版」

[ai-pr-reviewer](https://github.com/marketplace/actions/ai-based-pr-reviewer-summarizer-with-chat-capabilities)は、CodeRabbitが提供するGitHub Actions。「≒ CodeRabbitのOSS版」

基本の機能としてはどちらも同じで、PRのレビューなどをしてくれるもの。

このサービスの違いを混同していたため、導入時にやや詰まった。
（CodeRabbitを使用するためにymlファイルの設定が必要だと勝手に思い込んでいた（二重設定してしまっていた）、、）

<br>

※ai-pr-reviewerについては、こちらの記事もすごく参考になりました。
https://zenn.dev/egstock_inc/articles/86c07c9fe3bddf

<br>

## CodeRabbit（GitHub App）は無料プランでも補助ツールとしては十分使える
せっかくなので、GitHub AppであるCodeRabbitも軽く紹介。
[「CodeRabbit/Docs」](https://coderabbit.ai/docs)

基本的な使用方法は簡単で、AppとGitHubとの連携を承認して、使用したいレポジトリを選択するだけでOK。
Pro版の料金は1シートに対して15ドル。

新規登録後は自動的に7日間の無料期間が始まるとの記載あり。
> 1: Sign Up for CodeRabbit
Start your journey with CodeRabbit by registering via your GitHub or GitLab account. With a simple, few-click process, you'll connect CodeRabbit to your account and be ready to go in no time. Each new sign-up automatically starts with a 7-day trial for up to 50 seats, letting your team dive into CodeRabbit's capabilities.

無料期間終了後は、自動的に無料プランが適用される。
無料プランだと、レビューはされずにPRの要約だけ実行される。

**※ 要約だけでも十分便利なので、無料プランでも十分使えると思う。**
むしろPRがレビューコメントまみれにならなずにスッキリとして思ったよりいい感じ。
PR要約ツールとしては十分使えそう。（自動レビュー目的ではなくなってしまっているが、、）

以下、無料プランの出力例。
![](https://storage.googleapis.com/zenn-user-upload/383a01cb77b9-20231019.png)
![](https://storage.googleapis.com/zenn-user-upload/ff44be8fdc56-20231019.png)


<br>

# 2.OpenAI APIの新規登録した際に、rate limitの上限値が課金してるのに上がらない
「ai-pr-reviewer」を動かすためには、OpenAIのAPI Keyが必要になる。
今までOpenAIのAPI登録（課金）はしていなかったので、記事記載の通りにAPI Keyを取得する。

その後、ymlファイルに設定して、Actionsを実行まではすんなりといった。

**が、記事に記載のあるように綺麗にActionsが完了しない。**

エラーで変更ファイルが読み取れず、レビューされないファイルがちらほら発生してしまっている。

![](https://storage.googleapis.com/zenn-user-upload/babc381f4d52-20231007.png =300x)
*Files not summarized(reviewed) due to errors が発生*

ログを見ると、以下のようなrate limitが上限に達しているとのエラーが何回も出ている。
```
response: undefined, failed to send message to openai: Error: OpenAI error 429: {
    "error": {
        "message": "Rate limit reached for 10KTPM-200RPM in organization <org-OrgName> on tokens per min. 
        Limit: 10000 / min. Please try again in 6ms. 
        Contact us through our help center at help.openai.com if you continue to have issues.",
```

<br>

OpenAIのアカウントから`Rate limits`項目を確認。
"あなたのアカウントが使用できるrate limitの上限は以下"だと記載がある。
![](https://storage.googleapis.com/zenn-user-upload/9f26ceeab488-20231007.png =400x)

「gpt-4のrate limitは「**10000Tokens / 分**」 なのかー。」
と思いつつ、ついでにgpt-3.5-turboのrate limitを確認すると、
「**3リクエスト / 分**」？少なすぎない？

ここで初めて上限がおかしそうだと気づき、色々と調べるとどうやらこれは無料枠の上限値らしい。

なぜか課金して送金されたメールも通知されているのに、rate limitの上限値が上がっていない。

<br>

Issueにも同様の問題が上がっていた。
https://community.openai.com/t/regarding-the-issue-of-the-rate-limit-not-being-raised/410184/1

また、以下のIssueを見ると、有料の上限になるまで2日かかるとの記載がある。
（これは他の記事でも見つけたが、ドキュメントのどこにこの記載があるか見つけられない。。。TODO：ドキュメント箇所を見つけ次第記載。）
https://community.openai.com/t/api-request-limit-increase/409745

<br>

とりあえずは、2日後に上限が上がることを信じて待ちつつ、一応上限値が上がらないです。とメールを送っておいた。

::: message 
TODO: 進展があり次第記事更新

**10/10 追記①**
"rate limitの上限を引き上げたい場合は、Formから申請してください。"的な回答が返ってきた。
→このrate limitが課金後でもデフォルト値だったっぽい？？
→ちょっと試しで使用したかっただけなので、上限引き上げの申請まではしない（正当な必要性がないと申請は認めない的な記載があるため）が、最初からこういった仕様なのか最近変更になったのか気になる。
→もうちょっと深く問い合わせてみる。

**10/13 追記②**
かなり丁寧に問い合わせたつもりだが、テンプレで"rate limitの上限を引き上げたい場合は、Formから申請してください。"と同じ内容で帰ってきたので断念。

**10/19 追記③**
しばらく経ってドキュメントを確認すると、新たにrate limitの制限について明記されていた。
（5ドル振り込んでチャージしただけじゃ上限変わらかったので、実際に5ドルのapi使用料が発生したら上限上がるっぽい）
![](https://storage.googleapis.com/zenn-user-upload/fc6e583276e2-20231019.png)


:::

<!--  -->
<!-- 色々と調べて、〇〇することでrate limitの上限が課金後の上限である以下に増えました。 -->

<!-- ※openaiのapiの仕様やrate limitについては別記事にまとめました。 -->

<!-- rate limitが正常の値になったので、再度actionsを実行。 -->
<!-- すると、無事にレビューが完了しました。 -->

初めてOpenAIのAPIを取得する。って方はrate limitの上限値がしっかりと課金後の上限値になっているかは要確認。
その辺の仕様を知らずに、発生したエラーがActionsの設定の方に問題があると思ってgpt-4のActionsを回しまくると、**簡単に従量課金が進むので要注意。**
<!--  -->

<br>

### おまけ
rate limitが無料枠の時に、`gpt-4`だからActionsが完了しないのかと思い、`gpt-3.5-turbo`に変更して実行してみると、
無料枠では`gpt-3.5-turbo`のrate limitは **「3リクエスト / 分」** しかないので、Actions内でエラーが起きまくり、最終的に空レビューが返されて完了。
　
```
, backtrace: Error: OpenAI error 429: {
"error": {
"message": "Rate limit reached for default-gpt-3.5-turbo in organization <org-OrgName> on requests per min. 
    Limit: 3 / min. Please try again in 20s. 
    Contact us through our help center at help.openai.com if you continue to have issues. 
    Please add a payment method to your account to increase your rate limit. 
    Visit https://platform.openai.com/account/billing to add a payment method.",
"type": "requests",
"param": null,
"code": "rate_limit_exceeded"
}

<...膨大な上記エラーが発生...>

review: nothing obtained from openai
Submitting empty review for PR #××  // 空レビューが返されて完了
```



<!-- rate limitとは、 -->

<!-- rate limitは自動的に組織ごとに割り当てられる。 -->

<!-- お試し期間の無料枠だと、以下らしい。 -->

<!-- 有料会員となった送金すると、以下の上限になるらしい。 -->

<!-- ただ、有料会員になった際に、上限が上がらずに、無料枠のままであることを知らずにactionsが意図した結果にならず、何度もactionsを実行して動作確認していた（使用量だけしっかり取られた。。） -->

<!-- 発生していたエラーは、以下。rate limitが上限に達したので取得できないと言ったエラー。 -->
<!-- エラー後、何度もリトライを繰り返していそうだが、結果としてファイルを読み込んでくれずにactionsが完了してしまっている。 -->
