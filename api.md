# PhotoCloak API仕様 ver 0.0.1

PhotoCloak APIの開発者向けのドキュメントです。

## エンドポイント
本番になります。

---
## API一覧
### 認証
* [認証 API](#認証-api)

### フォトクローク
* [フォトクローク・新規作成 API](#フォトクローク・新規作成-api)
* [フォトクローク・作成完了 API](#フォトクローク・作成完了-api)
* [フォトクローク・削除 API](#フォトクローク・削除-api)
* [フォトクローク・検索 API](#フォトクローク・検索-api)
* [フォトクローク・取得 API](#フォトクローク・取得-api)
* [フォトクローク・フォロー追加 API](#フォトクローク・フォロー追加-api)
* [フォトクローク・フォロー解除 API](#フォトクローク・フォロー解除-api)

### 画像
* [画像取得 API](#画像取得-api)
* [画像アップロード API](#画像アップロード-api)
* [画像ダウンロード API](#画像ダウンロード-api)
* [画像削除 API](#画像削除-api)

---
## 認証 API
各種APIを利用するためには、OAuth2.0により規定された認可を行い事前にアクセストークンを取得します。  
API呼び出しの際、Authorizationヘッダーでアクセストークンを送信して認証します。

### 事前に必要なもの
認証認可を行うためには、以下の情報を事前に入手している必要があります。
* client_id　: 弊社にて発行するクライアントID
* client_secret　: 弊社にて発行するクライアントSecretキー

client_idとclient_secretの取得には、Developersへのご利用登録と利用申請が必要です。
弊社APIサービスサポート窓口へご連絡ください。

### 認可とアクセストークンの入手
クライアントプログラムは、事前に発行されたclient_id、client_secretを利用してアクセストークンを入手します。
#### ***Method*** : POST
#### ***Url*** : /oauth2/token
#### ***Request Header***
Authorization:に「"Basic " + base64encode(client_id + ":" + client_secret)」を設定します。
```
Authorization: Basic czZCaGRSa3F0MtQUUwOC0xMDM0MTVFOENGRTc=
Content-Type: application/x-www-form-urlencoded
```
#### ***x-www-form-urlencoded***
下記パラメータをリクエストボディにapplication/x-www-form-urlencoded形式で指定します。
##### ***username+passwordでの認証の場合***
API側では、作成ユーザーをusernameにて識別します。 トークンを取得する時は同一のusername、passwordを使用してください。

* grant_type [string]　: password固定
* username [string]　: client_id内で一意となる作成ユーザー識別文字列
* password(任意) [string]　: 作成ユーザーのパスワード。設定することを推奨します

#### ***Response (OK)***
```
{
    "access_token": "tc8lOyNW1Vsqx1EwXNUML6YdFJptCLblKR_sKfU2ClkikNpK3K9mK6s",
    "token_type": "bearer",
    "expires_in": 3599,
    "refresh_token": "a7949da3309848368e27b51b6039841f",
}
```
* access_token [string]　: 弊社サーバーが発行するアクセストークン
* token_type [string]　: トークンタイプ(bearer)
* expires_in [number]　: アクセストークンの有効期間を表す秒数
* refresh_token [string]　: 弊社サーバーが発行するリフレッシュトークン

#### ***Response (Bad Request)***
```
{
    "error": "invalid_request",
    "error_description": "リクエストに必要なパラメーターが含まれていません。"
}
```
| エラーコード | 意味|
|:-----------|:------------|
|invalid_request|リクエストに必要なパラメーターが含まれていない, サポートされないグラントタイプ以外のパラメーター値が含まれている, パラメーターが重複している, 異常値が設定されている|
|invalid_client|クライアント認証に失敗した|
|unsupported_grant_type|グラントタイプが認可サーバーによってサポートされていない|

### APIへのアクセス
入手したアクセストークンを使用して、各種APIにアクセスすることができます。  
アクセストークンの指定は共通で、APIを呼び出す際に、 Authorizationリクエストヘッダにアクセストークンを指定します。  
Authorization:に「"Bearer " + access_token」を設定します。  
※　トークンの有効期限は、1時間です。  
※　通信する経路は全て暗号化するため、httpsを利用します。
```
Authorization: Bearer tc8lOyNW1Vsqx1EwXNUML6YdFJptCLblKR_sKfU2ClkikNpK3K9mK6s
```
#### ***Response (Bad Request)***
アクセストークンが無効だった場合、APIの呼び出しは失敗し、エラー内容がレスポンスとして返却されます。
```
{
    "error": "invalid_token",
    "error_description": "不正なアクセストークンです。"
}
```
| エラーコード | 意味|
|:-----------|:------------|
|invalid_request|リクエストに必要なパラメーターが含まれていない, パラメーターが重複している, 異常値が設定されている|
|invalid_token|不正なアクセストークン, トークンの有効期限切れ|

---
## API呼び出しに関するエラーについて
### ***ステータスコード***

HTTP ステータスコードとともに結果を返します。

| ステータスコード | 意味|
|:-----------|:------------|
|200 (OK)|OK。リクエストが成功した。|
|400～|エラー|
|500～|サーバ内部エラー。処理中に例外が発生。こちらのエラーの場合はお問い合わせください。|

### ***レスポンスボディ***
エラーの場合、詳細がわかるようにレスポンスボディを返します。
```
　エラーの場合:
　{
　　"errors": [
        {
            "errorCode": "area_overflow",
            "message": "エリアでオバーフローが発生しました",
            "moreInfo": "page=7,areaID=TEXT01,orverString=です。"
        },...
    ]
　}
```
* errors : エラーのリスト
 * errorCode [string] : エラーコード
 * message [string] : エラーメッセージ
 * moreInfo [string] : エラーの場所を示す詳細な情報

---
## 画像アップロード API
画像のアップロードのみを行います。  
エリアへの配置は「画像配置/更新 API」で行います。

### ***Method*** : POST
### ***Url*** : /v1/{editKey}/images
### ***Request***
* editKey : 作品キー取得 APIにて発行したキーを指定してください。

### ***Request Body***
アップロードする画像ファイルを含めてください。ファイルが複数ある場合は一つ目のみが適用されます。
※最大画像サイズは20MB、対応するフォーマットはjpegのみです。

### ***Response***

```
{
    "imageId":"2-158-4-528-20160209191142-277116579"
}
```
* imageId [string] : アップロードした画像を識別するID

| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|画像を読み込めませんでした。画像が壊れているか、画像に対応しておりません。|invalid_file|
|406 (Not Acceptable)|指定されたEditKeyが見つかりません。|notacceptable_editkey|
|406 (Not Acceptable)|指定されたEditKeyが見つかりません。(注文済の作品は編集できません))|notacceptable_editkey|
|413 (Request Entity Too Large)|ファイルサイズが大きすぎます。|toolarge_file|
|415 (Unsupported Media Type)|ファイル形式が不明です。|unsupported_file|
|415 (Unsupported Media Type)|画像ファイルを選択してください。|unsupported_file|
```
【エラーの例】
{
    "errors": [
        {
            "errorCode": "unsupported_file",
            "message": "ファイル形式が不明です。",
            "moreInfo": "ご利用できる画像ファイル形式はjpeg（jpg）のみです。"
        },...
    ]
}
```

---
## アップロード済画像一覧取得 API
### ***Method*** : GET
### ***Url*** : /v1/{editKey}/images/
### ***QueryString***
* order : 画像のソート順を指定してください。
 + page : ページ(＆エリア)順にソート
 + upload(defalut) : アップロード順にソート
* detail : 結果の返却方法を指定してください。
 + true : 詳細モード
 + false(default) : 簡略モード
* filter : 返される画像の種類にフィルターをかけます。
 + user : ユーザーによってアップされた画像のみレスポンスに含まれます。
 + 指定なし(default) : 使用しているすべての画像をレスポンスに含めます。  
 ※固定で配置されている画像もレスポンスに含まれます。

### ***Request***
* editKey : 作品キー取得 APIにて発行したキーを指定してください。

### ***Response***
#### 詳細モード
```
{
	"images":[
    	{
        	"imageId":"2-158-4-528-20160209191142-277116579",
        	"page": "0",
        	"areaId": "PHOTO01",
        	"width": "1024"
        	"height": "768"
        	"rotate": "90"
        },
        {
        	"imageId":"2-158-4-528-20160201456546-678855441",
        	"page": "",
        	"areaId": "",
        	"width": ""
        	"height": ""
        	"rotate": ""
        },...
    ]
}

```
※未配置画像の場合は、"page""areaId""width""height""rotate"は空となる。
* imageId [string]: アップロードした画像を識別する画像ID。
* page [int]: ページ番号。
* areaId [string]: 配置されているイメージエリアのID。
* width  [int]: 画像の幅(px)。
* height [int]: 画像の高さ(px)。
* rotate [int]: 画像の回転角度。

#### 簡略モード
```
{
	"images":[
    	{
        	"imageId":"2-158-4-528-20160209191142-277116579"
        },
        {
        	"imageId":"2-158-4-528-20160201456546-678855441"
        },...
    ]
}

```
* imageId [string]: アップロードした画像を識別する画像ID。

| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|406 (Not Acceptable)|指定されたeditKeyが見つかりません。|notacceptable_editkey|
|406 (Not Acceptable)|指定されたorderが不正です。|notacceptable_order|
|404 (Not Found)|ファイルが存在しません。|notfound_file|
```
【エラーの例】
{
    "errors": [
        {
            "errorCode": "notacceptable_editkey",
            "message": "指定されたeditKeyが見つかりません。",
            "moreInfo": "editKey:123456789"
        },...
    ]
}
```
---
## アップロード済画像取得 API
### ***Method*** : GET
### ***Url*** : /v1/{editKey}/images/{imageId}
### ***Request***
* editKey : 作品キー取得 APIにて発行したキーを指定してください。
* imageId : アップロードした画像を識別する画像ID。

### ***Response***
imageIdで指定されたアップロード済画像(jpg)データを返します(バイナリ形式)。


| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|404 (Not Found)|ファイルが存在しません。|notfound_file|
|406 (Not Acceptable)|指定されたeditKeyが見つかりません。|notacceptable_editkey|
```
【エラーの例】
{
    "errors": [
        {
            "errorCode": "notfound_file",
            "message": "ファイルが存在しません。",
            "moreInfo": "imageId:2-158-4-528-20160209191142-277116579"
        },...
    ]
}
```
---
## アップロード済画像削除 API
### ***Method*** : POST
### ***Header*** : X-HTTP-Method-Override=DELETE
### ***Url*** : /v1/{editKey}/images/{imageId}
### ***Request***
* editKey : 作品キー取得 APIにて発行したキーを指定してください。
* imageId : アップロードした画像を識別する画像ID。

### ***Response***
imageIdで指定されたアップロード済画像(jpg)データを削除します。
使用済みの画像は指定できません。


| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|404 (Not Found)|ファイルが存在しません。|notfound_file|
|406 (Not Acceptable)|指定されたeditKeyが見つかりません。|notacceptable_editkey|
|406 (Not Acceptable)|使用済みの画像は指定できません。|notacceptable_imageid|

```
【エラーの例】
{
    "errors": [
        {
            "errorCode": "notfound_file",
            "message": "ファイルが存在しません。",
            "moreInfo": "imageId:2-158-4-528-20160209191142-277116579"
        },...
    ]
}
```

---
## アップロード画像取得 API
### ***Method*** : GET
### ***Url*** : /v1/{editKey}/images/{pageNo}/{areaID}
### ***QueryString***
* h : 横幅を指定してください。
* w : 高さを指定してください。
※hかwのどちらかが指定した場合は、イメージの比率で拡縮された画像を返します。

### ***Request***
* editKey : 作品キー取得 APIにて発行したキーを指定してください。
* pageNo : ページ番号を指定してください。
* areaID : 取得したいイメージエリアのIDを指定してください。

### ***Response***
* image : アップロードした画像(jpg)。

| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|404 (Not Found)|存在しないページが指定されました。|notfound_page|
|404 (Not Found)|存在しないエリアが指定されました。|notfound_area|
|404 (Not Found)|ファイルが存在しません。|notfound_file|

---
## 画像アップロード取消 API
### ***Method*** : POST
### ***Header*** : X-HTTP-Method-Override=DELETE
### ***Url*** : /v1/{editKey}/images/{pageNo}/{areaID}
### ***Request***
* editKey : 作品キー取得 APIにて発行したキーを指定してください。
* pageNo : ページ番号を指定してください。
* areaID : 取得したいイメージエリアのIDを指定してください。

### ***Response***
| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|404 (Not Found)|存在しないページが指定されました。|notfound_page|
|404 (Not Found)|存在しないエリアが指定されました。|notfound_area|
|404 (Not Found)|ファイルが存在しません。|notfound_file|

```
【エラーの例】
{
    "errors": [
        {
            "errorCode": "notfound_file",
            "message": "ファイルが存在しません。",
            "moreInfo": "page=7,areaID=PHOTO01"
        },...
    ]
}
```

---
## 画像アップロード/更新 API
画像をアップロードし、指定したエリアに配置します。  
「画像配置/更新 API」で後から別のエリアに配置することも可能です。

### ***Method*** : POST
### ***Header*** : X-HTTP-Method-Override=PUT
### ***Url*** : /v1/{editKey}/images/{pageNo}/{areaID}
### ***Request***
* editKey : 作品キー取得 APIにて発行したキーを指定してください。
* pageNo : ページ番号を指定してください。
* areaID : 取得したいイメージエリアのIDを指定してください。

### ***Request Body***
アップロードする画像ファイルを含めてください。ファイルが複数ある場合は一つ目のみが適用されます。
※存在しない場合は、エリアに配置された画像ファイルが削除されます。  
※最大画像サイズは20MB、対応するフォーマットはjpegのみです。

### ***Response***
```
{
    "imageId":"2-158-4-528-20160209191142-277116579"
}
```
* imageId [string] : アップロードした画像を識別するID

| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|画像を読み込めませんでした。画像が壊れているか、画像に対応しておりません。|invalid_file|
|404 (Not Found)|存在しないページが指定されました。|notfound_page|
|404 (Not Found)|存在しないエリアが指定されました。|notfound_area|
|413 (Request Entity Too Large)|ファイルサイズが大きすぎます。|toolarge_file|
|415 (Unsupported Media Type)|ファイル形式が不明です。|unsupported_file|
|416 (Requested Range Not Satisfiable)|ファイルの幅が小さすぎます。|tooshort_width|
|416 (Requested Range Not Satisfiable)|ファイルの高さが小さすぎます。|tooshort_height|
```
【エラーの例】
{
    "errors": [
        {
            "errorCode": "unsupported_file",
            "message": "ファイル形式が不明です。",
            "moreInfo": "ご利用できる画像ファイル形式はjpeg（jpg）のみです。"
        },...
    ]
}
```
