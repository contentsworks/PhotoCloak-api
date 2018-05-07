# PhotoCloak API仕様 ver 1.1.0

PhotoCloak APIの開発者向けのドキュメントです。

## エンドポイント
本番になります。

## 変更履歴
2017/12/26　ver 1.0.0 新規作成  
2018/03/27　ver 1.1.0 API追加（アカウント関連、フォトクロークトークン取得、フォトクロークプレフィックス取得）

---
## API一覧
### 認証
* [認証 API](#認証-api)

### フォトクローク
* [フォトクローク新規作成 API](#フォトクローク新規作成-api)
* [フォトクローク作成完了 API](#フォトクローク作成完了-api)
* [フォトクローク削除 API](#フォトクローク削除-api)
* [マイクローク一覧 API](#マイクローク一覧-api)
* [フォトクロークフォロー追加 API](#フォトクロークフォロー追加-api)
* [フォトクロークフォロー解除 API](#フォトクロークフォロー解除-api)
* [フォトクロークプレフィックス取得 API](#フォトクロークプレフィックス取得-api)
* [フォトクローク確認 API](#フォトクローク確認-api)
* [フォトクローク検索 API](#フォトクローク検索-api)
* [フォトクローク取得 API](#フォトクローク取得-api)
* [フォトクローク詳細 API](#フォトクローク詳細-api)
* [フォトクロークトークン取得 API](#フォトクロークトークン取得-api)

### 画像
* [画像取得 API](#画像取得-api)
* [画像アップロード API](#画像アップロード-api)
* [画像ダウンロード API](#画像ダウンロード-api)
* [画像削除 API](#画像削除-api)
* [画像閲覧カウントアップ API](#画像閲覧カウントアップ-api)

### アカウント
* [アカウント取得 API](#アカウント取得-api)
* [アカウント更新 API](#アカウント更新-api)

---
## 認証 API
認証を有する各種APIを利用するためには、OAuth2.0により規定された認可を行い事前にアクセストークンを取得します。  
API呼び出しの際、Authorizationヘッダーでアクセストークンを送信して認証します。

### 事前に必要なもの
認証認可を行うためには、以下の情報を事前に入手している必要があります。
* client_id : 弊社にて発行するクライアントID
* client_secret : 弊社にて発行するクライアントSecretキー

client_idとclient_secretの取得には、Developersへのご利用登録と利用申請が必要です。  
弊社APIサービスサポート窓口へご連絡ください。

### 認可とアクセストークンの入手
クライアントプログラムは、事前に発行されたclient_id、client_secretとusername、passwordを利用してアクセストークンを入手します。
#### ***Method*** : POST
#### ***Url*** : /api/v2/auth/token
#### ***Request Header***
Authorization:に「"Basic " + base64encode(client_id + ":" + client_secret)」を設定します。
```
Authorization: Basic czZCaGRSa3F0MtQUUwOC0xMDM0MTVFOENGRTc=
Content-Type: application/x-www-form-urlencoded
```
#### ***x-www-form-urlencoded***
下記パラメータをリクエストボディにapplication/x-www-form-urlencoded形式で指定します。
##### ***username+password***
grant_typeとusername、passwordを指定してください。

* grant_type [string] : password固定
* username [string] : ユーザーID（Email）
* password [string] : パスワード

#### ***Response (OK)***
```
{
    "access_token": "tc8lOyNW1Vsqx1EwXNUML6YdFJptCLblKR_sKfU2ClkikNpK3K9mK6s",
    "token_type": "bearer",
    "expires_in": 3599,
    "refresh_token": "a7949da3309848368e27b51b6039841f",
}
```
* access_token [string] : 弊社サーバーが発行するアクセストークン
* token_type [string] : トークンタイプ(bearer)
* expires_in [number] : アクセストークンの有効期間を表す秒数
* refresh_token [string] : 弊社サーバーが発行するリフレッシュトークン

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
入手したアクセストークンを使用して、認証を有する各種APIにアクセスすることができます。  
アクセストークンの指定は共通で、APIを呼び出す際に、 Authorizationリクエストヘッダにアクセストークンを指定します。  
Authorization:に「"Bearer " + access_token」を設定します。  
※　トークンの有効期限は、1時間です。  
※　通信する経路は全て暗号化するため、httpsを利用します。  
※　認証が不要な各種APIｈへのアクセスにはアクセストークンは不要です。
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

---
## フォトクローク新規作成 API
フォトクロークの新規作成を行います。  
クローク名と紹介分を登録します。

### ***Method*** : POST
### ***Url*** : /api/v2/mycloaks/
### ***Request***
* nm : クローク名
* dc : 紹介分

### ***Response***

```
{
    "CloakId":"123456790"
}
```
* CloakId [int] : クロークを識別するID※クローク番号ではありません。

| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|
|401 (Unauthorized)|認証が不正です|*****_*****|

---
## フォトクローク作成完了 API
フォトクロークの作成完了を行います。  
クロークの各種設定を行います。

### ***Method*** : POST
### ***Url*** : /api/v2/mycloaks/complete
### ***Request***
* ci : クロークID
* sc : 公開（0：合言葉なし／1：合言葉あり）
* lk : 合言葉
* tg : フリーワード
* dl : ダウンロード可否（0：否／1：可）
* ul : アップロード可否（0：否／1：可）
* ae : 自動延長可否（0：否／1：可）
* st : 公開ステータス（0：作成中／1：公開／8：一時停止）

### ***Response***

```
{
    "CloakId":"123456790"
}
```
* CloakId [int] : クロークを識別するID※クローク番号ではありません。

| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|
|401 (Unauthorized)|〇〇〇〇〇が●●●●●です|*****_*****|

---
## フォトクローク削除 API
フォトクロークの削除を行います。

### ***Method*** : POST
### ***Url*** : /api/v2/mycloaks/delete
### ***Request***
* ci : クロークID

### ***Response***
| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|
|401 (Unauthorized)|〇〇〇〇〇が●●●●●です|*****_*****|
|406 (Not Acceptable)|〇〇〇〇〇が●●●●●です|*****_*****|


---
## マイクローク一覧 API
マイクローク一覧の取得を行います。

### ***Method*** : POST
### ***Url*** : /api/v2/mycloaks/list
### ***Request***
* vm : 表示モード（all：すべて／public：公開／private：限定公開／provisional：作成中／follor：フォロー）

### ***Response***

```
{
    "Cloak":[]
   {
       "CloakId":"123456790",
       "MemberID":"123456790",
       "Name":"123456790",
       "PrefixCode":"ume",
       "PrefixName":"梅",
       "CloakNo":"11",
       "Status":"0",
       "WorkSpaceNo":"0",
       "ExpirationDatetime":"0",
       "ViewCount":"0",
       "NoImageUrl":"",
       "LockImageUrl":"",
       "CreateDatetime":"0",
       "UpdateDatetime":"0",
       "CloakSetting":
      {
          "CloakID":"0",
          "ScopeType":"0",
          "Description":,
          "CanDownload":,
          "CanUpload":,
          "IsAutoExtensionDate":
      }
       "Images":[]
      {
          "CloakImageID":"0"
          "CloakID":"0"
          "CloakSetting":
          "MyPhotoImageID":"0"
          "Status":"0"
          "ExpirationDatetime":"0"
          "MemberID":"0"
          "Comment":
          "ImageName":"0"
          "ViewCount":"0"
          "Rotate":
      }
       "Tags":[]
      {
          "CloakID":"0",
          "Tag":"0"
      },
      {
          "CloakID":"0",
          "Tag":"0"
      }
}
```
* HttpStatus [int] : ステータスコード。
* CloakId [int] : クロークID
* MemberID [int] : 会員ID
* Name [string] : クローク名
* PrefixCode [string] : クローク番号・先頭辞
* CloakNo [int] : クローク番号・番号
* Status [int] : 公開ステータス（0：作成中／1：公開／8：一時停止）
* WorkSpaceNo [int] : ワークスペース番号
* NoImageUrl [string] : 写真なしイメージパス
* LockImageUrl [string] : 合言葉ありイメージパス
* CreateDatetime [dateTime] : 作成日時
* UpdateDatetime [dateTime] : 更新日時
* ScopeType [int] : 公開（0：合言葉なし／1：合言葉あり）
* Description [int] : 紹介文
* CanDownload [int] : ダウンロード可否（0：否／1：可）
* CanUpload [int] : アップロード可否（0：否／1：可）
* IsAutoExtensionDate [int] : 自動延長（0：無／1：有）
* CloakImageID [int] : クロークイメージID
* MyPhotoImageID [int] : マイフォトイメージID
* ExpirationDatetime [dateTime] : 有効期限
* Comment [string] : コメント
* ImageName [string] : イメージ名
* ViewCount [int] : 閲覧数
* Rotate [int] : 画像向き
* Tag [int] : タグ

| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|
|401 (Unauthorized)|〇〇〇〇〇が●●●●●です|*****_*****|

---
## フォトクロークフォロー追加 API
フォトクロークのフォローを行います。

### ***Method*** : POST
### ***Url*** : /api/v2/mycloaks/addfollow
### ***Request***
* ci : クロークID

### ***Response***
| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|
|401 (Unauthorized)|〇〇〇〇〇が●●●●●です|*****_*****|
|406 (Not Acceptable)|〇〇〇〇〇が●●●●●です|*****_*****|

---
## フォトクロークフォロー解除 API
フォトクロークのフォロー解除を行います。

### ***Method*** : POST
### ***Url*** : /api/v2/mycloaks/delfollow
### ***Request***
* ci : クロークID

### ***Response***
| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|
|401 (Unauthorized)|〇〇〇〇〇が●●●●●です|*****_*****|
|406 (Not Acceptable)|〇〇〇〇〇が●●●●●です|*****_*****|

---
## フォトクロークプレフィックス取得 API
クローク番号のプレフィックス取得を行います。

### ***Method*** : GET
### ***Url*** : /api/v2/cloaks/prefix

### ***Response***

```
{
    "Prefixs":[]
    {
        "Name":"梅"
        "Code":"ume",
    }
    "MaxCloakNo":"99",
}
```
* Name : クローク番号・先頭辞名
* Code : クローク番号・先頭辞コード
* MaxCloakNo [int] : クローク番号・番号最大数

| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|

---
## フォトクローク確認 API
フォトクロークの存在確認を行います。

### ***Method*** : GET
### ***Url*** : /api/v2/cloaks/check
### ***Request***
* pd : クローク番号・先頭辞
* no : クローク番号・番号

### ***Response***

```
{
    "ScopeType":"0",
    "CloakID":"0",
    "Name":"123456790",
}
```
* ScopeType : 公開（0：合言葉なし／1：合言葉あり）
* CloakID [int] : クロークを識別するID※クローク番号ではありません。
* Name : クローク名

| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|

---
## フォトクローク検索 API
フォトクロークの検索を行います。

### ***Method*** : GET
### ***Url*** : /api/v2/cloaks/search
### ***Request***
* tg : タグ・フリーワード
* s : ソート（limit：タイムリミット／view：閲覧数／new：新着／name：名前）
* os : OFFSET（開始位置）※デフォルト：0
* li : LIMIT（限度数）

### ***Response***

```
{
    "Cloak":[]
   {
       "CloakId":"123456790",
       "MemberID":"123456790",
       "Name":"123456790",
       "PrefixCode":"ume",
       "PrefixName":"梅",
       "CloakNo":"11",
       "Status":"0",
       "WorkSpaceNo":"0",
       "ExpirationDatetime":"0",
       "ViewCount":"0",
       "NoImageUrl":"",
       "LockImageUrl":"",
       "CreateDatetime":"0",
       "UpdateDatetime":"0",
       "CloakSetting":
      {
          "CloakID":"0",
          "ScopeType":"0",
          "Description":,
          "CanDownload":,
          "CanUpload":,
          "IsAutoExtensionDate":
      }
       "Images":[]
      {
          "CloakImageID":"0"
          "CloakID":"0"
          "CloakSetting":
          "MyPhotoImageID":"0"
          "Status":"0"
          "ExpirationDatetime":"0"
          "MemberID":"0"
          "Comment":
          "ImageName":"0"
          "ViewCount":"0"
          "Rotate":
      }
       "Tags":[]
      {
          "CloakID":"0",
          "Tag":"0"
      },
      {
          "CloakID":"0",
          "Tag":"0"
      }
}
```
* HttpStatus [int] : ステータスコード。
* CloakId [int] : クロークID
* MemberID [int] : 会員ID
* Name [string] : クローク名
* PrefixCode [string] : クローク番号・先頭辞
* CloakNo [int] : クローク番号・番号
* Status [int] : 公開ステータス（0：作成中／1：公開／8：一時停止）
* WorkSpaceNo [int] : ワークスペース番号
* NoImageUrl [string] : 写真なしイメージパス
* LockImageUrl [string] : 合言葉ありイメージパス
* CreateDatetime [dateTime] : 作成日時
* UpdateDatetime [dateTime] : 更新日時
* ScopeType [int] : 公開（0：合言葉あり／1：合言葉なし）
* Description [int] : 紹介文
* CanDownload [int] : ダウンロード可否（0：否／1：可）
* CanUpload [int] : アップロード可否（0：否／1：可）
* IsAutoExtensionDate [int] : 自動延長（0：無／1：有）
* CloakImageID [int] : クロークイメージID
* MyPhotoImageID [int] : マイフォトイメージID
* ExpirationDatetime [dateTime] : 有効期限
* Comment [string] : コメント
* ImageName [string] : イメージ名
* ViewCount [int] : 閲覧数
* Rotate [int] : 画像向き
* Tag [int] : タグ

| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|

---
## フォトクローク取得 API
フォトクロークの取得を行います。

### ***Method*** : GET
### ***Url*** : /api/v2/cloaks
### ***QueryString***
* ci : クロークID

### ***Response***

```
{
    "Cloak":
   {
       "CloakId":"123456790",
       "MemberID":"123456790",
       "Name":"123456790",
       "PrefixCode":"ume",
       "PrefixName":"梅",
       "CloakNo":"11",
       "Status":"0",
       "WorkSpaceNo":"0",
       "ExpirationDatetime":"0",
       "ViewCount":"0",
       "NoImageUrl":"",
       "LockImageUrl":"",
       "CreateDatetime":"0",
       "UpdateDatetime":"0",
       "CloakSetting":
      {
          "CloakID":"0",
          "ScopeType":"0",
          "Description":,
          "CanDownload":,
          "CanUpload":,
          "IsAutoExtensionDate":
      }
       "Images":[]
      {
          "CloakImageID":"0"
          "CloakID":"0"
          "CloakSetting":

"MyPhotoImageID":"0"
          "Status":"0"
          "ExpirationDatetime":"0"
          "MemberID":"0"
          "Comment":
          "ImageName":"0"
          "ViewCount":"0"
          "Rotate":
      }
       "Tags":[]
      {
          "CloakID":"0",
          "Tag":"0"
      },
      {
          "CloakID":"0",
          "Tag":"0"
      }
}
```
* CloakId [int] : クロークID
* MemberID [int] : 会員ID
* Name [string] : クローク名
* PrefixCode [string] : クローク番号・先頭辞
* CloakNo [int] : クローク番号・番号
* Status [int] : 公開ステータス（0：作成中／1：公開／8：一時停止）
* WorkSpaceNo [int] : ワークスペース番号
* NoImageUrl [string] : 写真なしイメージパス
* LockImageUrl [string] : 合言葉ありイメージパス
* CreateDatetime [dateTime] : 作成日時
* UpdateDatetime [dateTime] : 更新日時
* ScopeType [int] : 公開（0：合言葉なし／1：合言葉あり）
* Description [int] : 紹介文
* CanDownload [int] : ダウンロード可否（0：否／1：可）
* CanUpload [int] : アップロード可否（0：否／1：可）
* IsAutoExtensionDate [int] : 自動延長（0：無／1：有）
* CloakImageID [int] : クロークイメージID
* MyPhotoImageID [int] : マイフォトイメージID
* ExpirationDatetime [dateTime] : 有効期限
* Comment [string] : コメント
* ImageName [string] : イメージ名
* ViewCount [int] : 閲覧数
* Rotate [int] : 画像向き
* Tag [int] : タグ

| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|

---
## フォトクローク詳細 API
フォトクロークの取得を行います。  
※本APIは使用しないでください。  
　フォトクロークトークン取得APIとフォトクローク取得APIを使用してください。

### ***Method*** : POST
### ***Url*** : /api/v2/cloaks/detail/
### ***Request***
* ci : クロークID
* lk : 合言葉

### ***Response***

```
{
    "Cloak":
   {
       "CloakId":"123456790",
       "MemberID":"123456790",
       "Name":"123456790",
       "PrefixCode":"ume",
       "PrefixName":"梅",
       "CloakNo":"11",
       "Status":"0",
       "WorkSpaceNo":"0",
       "ExpirationDatetime":"0",
       "ViewCount":"0",
       "NoImageUrl":"",
       "LockImageUrl":"",
       "CreateDatetime":"0",
       "UpdateDatetime":"0",
       "CloakSetting":
      {
          "CloakID":"0",
          "ScopeType":"0",
          "Description":,
          "CanDownload":,
          "CanUpload":,
          "IsAutoExtensionDate":
      }
       "Images":[]
      {
          "CloakImageID":"0"
          "CloakID":"0"
          "CloakSetting":
          "MyPhotoImageID":"0"
          "Status":"0"
          "ExpirationDatetime":"0"
          "MemberID":"0"
          "Comment":
          "ImageName":"0"
          "ViewCount":"0"
          "Rotate":
      }
       "Tags":[]
      {
          "CloakID":"0",
          "Tag":"0"
      },
      {
          "CloakID":"0",
          "Tag":"0"
      }
}
```
* CloakId [int] : クロークID
* MemberID [int] : 会員ID
* Name [string] : クローク名
* PrefixCode [string] : クローク番号・先頭辞
* CloakNo [int] : クローク番号・番号
* Status [int] : 公開ステータス（0：作成中／1：公開／8：一時停止）
* WorkSpaceNo [int] : ワークスペース番号
* NoImageUrl [string] : 写真なしイメージパス
* LockImageUrl [string] : 合言葉ありイメージパス
* CreateDatetime [dateTime] : 作成日時
* UpdateDatetime [dateTime] : 更新日時
* ScopeType [int] : 公開（0：合言葉なし／1：合言葉あり）
* Description [int] : 紹介文
* CanDownload [int] : ダウンロード可否（0：否／1：可）
* CanUpload [int] : アップロード可否（0：否／1：可）
* IsAutoExtensionDate [int] : 自動延長（0：無／1：有）
* CloakImageID [int] : クロークイメージID
* MyPhotoImageID [int] : マイフォトイメージID
* ExpirationDatetime [dateTime] : 有効期限
* Comment [string] : コメント
* ImageName [string] : イメージ名
* ViewCount [int] : 閲覧数
* Rotate [int] : 画像向き
* Tag [int] : タグ

| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|

---
## フォトクロークトークン取得 API
フォトクロークトークン取得を行います。

### ***Method*** : POST
### ***Url*** : /api/v2/cloaks/token
### ***Request***
* ci : クロークID
* lk : 合言葉

### ***Response***

```
{
    "CloakToken":"tc8lOyNW1Vsqx1EwXNUML6YdFJptCLblKR_sKfU2ClkikNpK3K9mK6s",
    "CloakId":"99",
}
```
* CloakToken : フォトクロークトークンクローク番号・先頭辞名
* CloakId [int] : クロークID

| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|

### APIへのアクセス
入手したフォトクロークトークンを使用して、合言葉照合を有する各種APIにアクセスすることができます。  
フォトクロークトークンの指定は、APIを呼び出す際に、 リクエストヘッダ（カスタムヘッダ）にフォトクロークトークンを指定します。  
※　通信する経路は全て暗号化するため、httpsを利用します。  
※　合言葉照合が不要なAPIへのアクセスにはフォトクロークトークンは不要です。  ※　フォトクロークトークンが不正の場合は、APIでエラーが返却されます。
```
X-PhotoCloakToken: tc8lOyNW1Vsqx1EwXNUML6YdFJptCLblKR_sKfU2ClkikNpK3K9mK6s
```

---
## 画像取得 API
画像の取得を行います。  

### ***Method*** : GET
### ***Url*** : /api/v2/photos
### ***Request***
* ci : クロークID
* cd : クロークイメージID
* ps : 写真サイズ（0：標準／1：小サムネイル／1：大サムネイル）

### ***Response***
クロークイメージIDで指定された画像(jpg)データを返します(バイナリ形式)。  
※標準サイズはオーナーもしくはダウンロード許可のクロークでないと取得できない

| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|



---
## 画像アップロード API
画像のアップロードのみを行います。  

### ***Method*** : POST
### ***Url*** : /api/v2/photos
### ***Request***
* ci : クロークID

### ***Request Body***
アップロードする画像ファイルを含めてください。  
※最大画像サイズは20MB、対応するフォーマットはjpegのみです。

### ***Response***
| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|
|401 (Unauthorized)|〇〇〇〇〇が●●●●●です|*****_*****|
|406 (Not Acceptable)|〇〇〇〇〇が●●●●●です|*****_*****|

---
## 画像ダウンロード API
### ***Method*** : GET
### ***Url*** : /api/v2/photos/export
### ***QueryString***
* ci : クロークID
* cd : クロークイメージID

### ***Request***
* ci : クロークID
* cd : クロークイメージID

### ***Response***
クロークイメージIDで指定された画像(jpg)データを返します(バイナリ形式)。

| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|
|401 (Unauthorized)|〇〇〇〇〇が●●●●●です|*****_*****|
|404 (Not Found)|〇〇〇〇〇が●●●●●です|*****_*****|
|406 (Not Acceptable)|〇〇〇〇〇が●●●●●です|*****_*****|

---
## 画像削除 API
### ***Method*** : POST
### ***Url*** : /api/v2/photos/delete
### ***Request***
* ci : クロークID
* cd : クロークイメージID

### ***Response***
| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|
|401 (Unauthorized)|〇〇〇〇〇が●●●●●です|*****_*****|
|404 (Not Found)|〇〇〇〇〇が●●●●●です|*****_*****|
|406 (Not Acceptable)|〇〇〇〇〇が●●●●●です|*****_*****|


---
## 画像閲覧カウントアップ API
### ***Method*** : POST
### ***Url*** : /api/v2/photos/look
### ***Request***
* cd : クロークイメージID

### ***Response***
| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|
|404 (Not Found)|〇〇〇〇〇が●●●●●です|*****_*****|

---
## アカウント取得 API
アカウントの取得を行います。  

### ***Method*** : GET
### ***Url*** : api/v2/account

### ***Response***

```
{
    "AccountName":"海山太郎",
    "Email":"umiyama@contentsworks.co.jp",
    "MaxCloakCount":"3",
}
```
* AccountName : アカウント名／ニックネーム
* Email : メールアドレス
* MaxCloakCount [int] : 最大レンタル可能数

| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|


---
## アカウント更新 API
アカウントの更新を行います。  

### ***Method*** : POST
### ***Url*** : api/v2/account/update
### ***Request***
* an : アカウント名・ニックネーム
* cd : クロークイメージID

### ***Response***
| ステータスコード | 意味|エラーコード|
|:-----------|:------------|:------------|
|200 (OK)|成功|-|
|400 (Bad Request)|〇〇〇〇〇が●●●●●です|*****_*****|
