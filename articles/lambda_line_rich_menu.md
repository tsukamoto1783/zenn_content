---
title: "【Flutter / AWS】LINEリッチメニュ作成"
emoji: "⛳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [FLutter, Dart, MessagingAPI, Lambda, AWS]
published: true
---
FlutterとAWSで、「ボタンを押下するとLINEリッチメニュが作成される。」
だけのシンプルな構成のアプリ。
![](https://storage.googleapis.com/zenn-user-upload/8a6d17ed482c-20230228.png)

出来上がりはこんな感じ↓
![](https://storage.googleapis.com/zenn-user-upload/6a2f33da162d-20230323.gif =600x)

※記事参考
[curlでシンプルなリッチメニュの作成](https://zenn.dev/tsukatsuka1783/articles/curl_line_rich_menu)

# 対応したこと
1. Lambda：関数作成、環境変数の設定
2. Lambda：リッチメニュ作成するコードの実装
3. S3：リッチメニュに使用する画像データを格納
4. API Gateway：API作成
5. APIをCallするアプリの作成


## 1. Lambda：関数作成、環境変数の設定
- 基本的にはデフォルト設定でLambda関数を作成。(今回はPython3.9を選択)
- S3を読み込むためのアクセス権限の追加。
- Lambda上での環境変数を設定。
LINE Developers のアクセストークンを記載。
![](https://storage.googleapis.com/zenn-user-upload/daae7e8f5f1a-20230225.png)

## 2. Lambda：リッチメニュ作成するコード実装
Pythonで実装。
※あまりPythonに詳しくは無いので、コードのお作法的な部分はご愛嬌。

```yaml:lambdaのフォルダ構成
project(root)
├─ lambda_function.py
├─ const.py
└─ functions.py
```

:::: details lambda_function.py
```python:lambda_function.py
# ディレクトリ内のファイルimport
import const
import functions

def lambda_handler(event, context):
    param = event["queryStringParameters"]
    
    # クエリパラメーターの取得
    param_check_result = functions.check_query_param(param, const.QUERY_PARAM_NAME)
    if param_check_result == None:
        check_query_param_error = const.RESPONSE_DEFALUT_DATE
        check_query_param_error["body"] = "クエリパラメーターが不正です"
        check_query_param_error["statusCode"] = 400
        return check_query_param_error

    is_bool = param_check_result

    # リッチメニュ作成
    create_result = functions.create_rich_menu()
    print(create_result)
    if create_result["statusCode"] != 200:
        return create_result
    
    # リッチメニュID取得
    rich_menu_id = create_result["richMenuId"]

    # リッチメニュに画像をアップロード
    upload_result = functions.upload_rich_menu_image(rich_menu_id, is_bool)
    print(upload_result)
    if upload_result['statusCode'] != 200:
        return upload_result

    # デフォルトのリッチメニュを設定
    set_default_result = functions.set_default_rich_menu(rich_menu_id)
    print(set_default_result)
    if set_default_result['statusCode'] != 200:
        return set_default_result

    return functions.creating_response_date("deploy")
```
::::

:::: details const.py
```python:const.py
import os

QUERY_PARAM_NAME = {クエリパラメータに指定するKey名}
BUCKET_NAME = {S3バケット名}
IMAGE_NAME_1 = {S3に格納の画像名}
IMAGE_NAME_2 = {S3に格納の画像名}

RESPONSE_DEFALUT_DATE = {
    "isBase64Encoded": False,
    "statusCode": 200,
    "body": ""
}

ACCESS_TOKEN = os.getenv('LINE_CHANNEL_ACCESS_TOKEN', None)

RICH_MENU_DATA = """
{
    "size": {
        "width": 2500,
        "height": 1686
    },
    "selected": false,
    "name": "richmenu name",
    "chatBarText": "tap here",
    "areas": [
        {
            "bounds": {
                "x": 0,
                "y": 0,
                "width": 2500,
                "height": 1686
            },
            "action": {
                "type": "postback",
                "data": "action=buy&itemid=123"
            }
        }
    ]
}
"""
```
::::

:::: details functions.py
```python:functions.py
import urllib.request
import json
import boto3
import traceback

# ディレクトリ内のファイルimport
import const

# クエリパラメーターの取得
def check_query_param(param: dict, query_param_name: str):
    query_param_value = None

    # クエリパラメータのbool文字列をbool値に変換
    # 不正のパラメーターだった場合はNoneを返す
    try:
        if param[query_param_name].lower() == "true":
            query_param_value = True
        elif param[query_param_name].lower() == "false":
            query_param_value = False
        else:
            print(query_param_name)
            None
        return query_param_value
    except e:
        print(e)
        return query_param_value
    

# レスポンスjsonデータの成形
def creating_response_date(process_name: str, reslut: bool = True, status_code: int = 200):
    response_date = const.RESPONSE_DEFALUT_DATE
    if reslut:
        response_date["body"] = "Ruch Menu: " + process_name +" Success"
    else:
        response_date["body"] = "Ruch Menu: " + process_name +" error"
        
    if status_code != 200:
        response_date["statusCode"] = status_code
    
    return response_date


# リッチメニュ作成
def create_rich_menu():
    process_name = "create"
    create_reslut = None
    url = "https://api.line.me/v2/bot/richmenu"
    headers = {
        "Authorization": "Bearer " + const.ACCESS_TOKEN,
        "Content-Type": "application/json"
    }
    
    # API Call
    request = urllib.request.Request(url, headers=headers, data=const.RICH_MENU_DATA.encode("utf-8"), method="POST")

    try:
        with urllib.request.urlopen(request) as response:
            create_reslut = creating_response_date(process_name)
            create_reslut = dict(create_reslut, **json.loads(response.read().decode("utf-8")))
            return create_reslut
    
    except urllib.error.HTTPError as e:
        print(e.read())  
        create_reslut = creating_response_date(process_name, False, e.code)
        return create_reslut

        
# リッチメニュに画像をアップロード
def upload_rich_menu_image(rich_menu_id: str, is_bool: bool):
    process_name = "upload"
    upload_reslut = None
    url = f"https://api-data.line.me/v2/bot/richmenu/{rich_menu_id}/content"
    headers = {
        "Authorization": "Bearer " + const.ACCESS_TOKEN,
        "Content-Type": "image/png"
    }
    # S3から画像データを取得
    image_data = get_img_from_s3(is_bool)
    if type(image_data) == dict:
        return image_data

    # API Call
    request = urllib.request.Request(url, headers=headers, data=image_data, method='POST')
    
    try:
        with urllib.request.urlopen(request) as response:
            upload_reslut = creating_response_date(process_name)
            return upload_reslut

    except urllib.error.HTTPError as e:
        print(e.read())  
        upload_reslut = creating_response_date(process_name, False, e.code)
        return upload_reslut


# S3から画像データを取得
def get_img_from_s3(is_bool: bool):
    get_obj_response_date = None
    object_name = ''

    # 引数のbool値で取得する画像データを切り替え
    if is_bool:
        object_name = const.IMAGE_NAME_1
    else:
        object_name = const.IMAGE_NAME_2
    
    s3 = boto3.client('s3')
    
    # S3から画像データ取得
    try:
        obj = s3.get_object(Bucket=const.BUCKET_NAME, Key=object_name)
        body = obj['Body'].read()
        return body
    except:
        get_obj_response_date = const.RESPONSE_DEFALUT_DATE
        get_obj_response_date["body"] = "S3からの画像取得に失敗"
        get_obj_response_date["statusCode"] = 400
        return get_obj_response_date

    
# デフォルトのリッチメニュを設定
def set_default_rich_menu(rich_menu_id: str):
    set_reslut = None
    process_name = "set"
    url = f"https://api.line.me/v2/bot/user/all/richmenu/{rich_menu_id}"
    headers = {
        "Authorization": "Bearer " + const.ACCESS_TOKEN
    }
    
    # API Call
    request = urllib.request.Request(url, headers=headers, method="POST")
    
    try:
        with urllib.request.urlopen(request) as response:
            set_reslut = creating_response_date(process_name)
            return set_reslut
            
    except urllib.error.HTTPError as e:
        print(e.read())  
        set_reslut = creating_response_date(process_name, False, e.code)
        return set_reslut
```
::::

## 3. S3：リッチメニュに使用する画像データを格納
- バケットを作成して、リッチメニュ用の画像データを格納する。
※画像サイズには規定があるので注意。(アスペクト比を確認してなくてやや詰まった、、)
[アスペクト比計算ツール](https://ocadweb.com/aspectratiotools)
```txt
- 画像フォーマット：JPEGまたはPNG
- 画像の幅サイズ（ピクセル）：800以上、2500以下
- 画像の高さサイズ（ピクセル）：250以上
- 画像のアスペクト比（幅/高さ）：1.45以上
- 最大ファイルサイズ：1MB
```

## 4. API Gateway：API作成
- APIを作成。APIタイプはHTTP APIを選択。(REST APIでも可)
- 作成したAPIを、Lambdaのトリガーに設定する。設定したらAPIエンドポイントが発行されるので控えておく。
![](https://storage.googleapis.com/zenn-user-upload/b52a70cfc4b7-20230228.png)

## 5. APIをCallするアプリの作成
APIをCallするボタンだけのシンプル画面構成。
APIのCallにはDioパッケージを使用。
API作成時に発行されたエンドポイントをコード内に記載。

:::: details main.dart
```dart:APIをCallするためのボタンを実装するだけのコード
import 'package:dio/dio.dart';
import 'package:flutter/material.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';

void main() async {
  await dotenv.load(fileName: ".env");
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  // 変数:apiGatewayEndpointに、API作成時に発行されたエンドポイントを指定。
  // 以下サンプルでは、flutter_dotenvを使用して環境変数として指定。
  final String? apiGatewayEndpoint = dotenv.env['API_GATEWAY_ENDPOINT'];
  final dio = Dio();

  bool isVisible = false;
  bool isLoading = false;
  Response<dynamic>? callApiResult;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("APIGateway call test"),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            _buildElevatedButton("Call API ①", true),
            const SizedBox(height: 30),
            _buildElevatedButton("Call API ②", false),
            const SizedBox(height: 30),
            const Text("↓【APIGateway call result】↓"),
            const SizedBox(height: 30),
            _buildResponseDisplay(isVisible, callApiResult),
            isLoading ? const CircularProgressIndicator() : Container(),
          ],
        ),
      ),
    );
  }

  Widget _buildElevatedButton(String buttonText, bool isSwitch) {
    return ElevatedButton(
      child: Text(buttonText),
      onPressed: () async {
        _callApi(isSwitch);
      },
    );
  }

  Widget _buildResponseDisplay(bool isVisible, Response? callApiResult) {
    Widget? displayText;
    if (callApiResult == null) {
      displayText = const Text("API呼び出しに失敗しました");
    }
    return isVisible ? (displayText ?? Text("$callApiResult")) : Container();
  }

  Future<Response?> _callApi(bool isSwitch) async {
    // ボタン押下時の初期化処理
    setState(() {
      isVisible = false;
      callApiResult = null;
      isLoading = true;
    });

    try {
      callApiResult = await dio.get('$apiGatewayEndpoint?is_swithc=$isSwitch');
    } catch (e) {
      print(e);
      throw Exception(e);
    } finally {
      setState(() {
        isVisible = true;
        isLoading = false;
      });
    }
    return callApiResult;
  }
}

```
::::

# プチ詰まりポイント
- クエリパラメータは文字列扱いなので、lambda側でboolに変換する処理が必要。
- Dioの仕様に詰まった。200以外のエラコードはDioErrorとしてスローされる。
- Dioの戻り値として、文字列か、{"body", "statusCode"}をKeyに持つJson形式である必要が基本的にはあり。（設定次第ではバイナリとかも可能）
- Lambdaでの出力形式は以下のドキュメント参照。
  [プロキシ統合のための Lambda 関数の出力形式](https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-output-format)


# おわり
- フロント部分をFlutterにしたのはただの好み。(Dioの学習も兼ねて）
- フロント部分から直接MessagingAPIをCallすれば済む話だが、AWSの勉強を兼ねてなのであしからず。

# 参考記事
[・curlでシンプルなリッチメニュの作成](https://zenn.dev/tsukatsuka1783/articles/curl_line_rich_menu)
[・【Flutter Web】flutter_dotenv：デプロイ時に'.env'が読み込まれない](https://zenn.dev/tsukatsuka1783/articles/64c9e06d516a3e)