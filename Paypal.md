## PayPal 정산 데이터 수신 명세서 (OAuth 인증 및 거래 내역 조회 기반)

### 📌 PayPal 정산 프로세스 요약

1. **고객 동의창 생성**: PayPal 계정 권한 요청 URL 생성 및 인증 수행
    
2. **authorization_code 수신**: redirect_uri를 통해 code 수신
    
3. **access_token 발급**: code를 이용해 access_token 및 refresh_token 발급
    
4. **거래 내역 조회**: access_token을 사용해 최근 30일 거래 내역 조회
    

---

### ✅ 1. 고객 동의창 생성

**GET (Redirect to)**  
`https://www.sandbox.paypal.com/signin/authorize`

**Remarks**  
PayPal 사용자 인증을 위해 페이고스에서 구성한 URL로 브라우저 리디렉션. code 반환을 통해 이후 access_token 교환.

**Request**

**▶ Query Parameters**

|파라미터|타입|필수|설명|
|---|---|---|---|
|client_id|String|Y|PayPal Developer 앱의 Client ID|
|response_type|String|Y|고정값 `code`|
|redirect_uri|String|Y|인증 완료 후 돌아올 콜백 URI|
|scope|String|Y|요청 권한 범위 (예: openid, reporting, payments)|

**Response (Redirect)**

```
https://paygos.ibk.co.kr/paypal/oauth/callback?code=abc123xyz
```

|   |   |
|---|---|
|필드명|설명|
|code|인증 완료 후 발급된 인증 코드|

---

### ✅ 2. access_token 발급

**POST**  
`https://api-m.sandbox.paypal.com/v1/oauth2/token`

**Remarks**  
앞서 받은 `authorization_code`를 이용해 access_token 및 refresh_token을 발급

**Request**

**▶ Headers**

|   |   |   |   |
|---|---|---|---|
|헤더명|타입|필수|설명|
|Authorization|String|Y|Basic base64(CLIENT_ID:CLIENT_SECRET)|
|Content-Type|String|Y|application/x-www-form-urlencoded|

**▶ Body Parameters** (x-www-form-urlencoded)

|   |   |   |   |
|---|---|---|---|
|파라미터|타입|필수|설명|
|grant_type|String|Y|고정값: `authorization_code`|
|code|String|Y|동의 과정에서 PayPal이 전달한 `code` 값|
|redirect_uri|String|Y|최초 요청에 사용한 redirect_uri 값과 동일해야 함|

**Response(JSON)**

```
{
  "access_token": "A21AAE...",
  "expires_in": 32400,
  "refresh_token": "A21AAH...",
  "token_type": "Bearer",
  "scope": "openid https://uri.paypal.com/services/reporting/search/read"
}
```

|   |   |
|---|---|
|필드명|설명|
|access_token|API 호출 시 사용하는 인증 토큰|
|expires_in|access_token 유효 시간 (초 단위, 예: 32400초 = 9시간)|
|refresh_token|장기 갱신용 토큰|
|token_type|고정값: `Bearer`|
|scope|부여된 권한 범위 목록|

---

### ✅ 3. 거래 내역 조회 (최근 30일)

**GET**  
`https://api-m.sandbox.paypal.com/v1/reporting/transactions`

**Remarks**  
access_token을 이용해 PayPal 계정의 거래 내역을 조회. 시작일~종료일 범위 지정 가능

**Request**

**▶ Headers**

|   |   |   |   |
|---|---|---|---|
|헤더명|타입|필수|설명|
|Authorization|String|Y|Bearer {access_token}|
|Content-Type|String|Y|application/json|

**▶ Query Parameters**

|   |   |   |   |
|---|---|---|---|
|파라미터|타입|필수|설명|
|start_date|String|Y|조회 시작일 (예: `2024-06-01T00:00:00Z`)|
|end_date|String|Y|조회 종료일 (예: `2024-06-30T23:59:59Z`)|
|page_size|Int|N|페이지당 결과 수 (예: 100)|
|page|Int|N|페이지 번호|

**Response(JSON)** (예시 일부)

```
{
  "transaction_details": [
    {
      "transaction_info": {
        "transaction_id": "1AB23456CD7890123",
        "transaction_initiation_date": "2024-06-01T12:00:00Z",
        "transaction_amount": {
          "value": "19.99",
          "currency_code": "USD"
        },
        "transaction_status": "S"
      },
      "payer_info": {
        "email_address": "buyer@example.com"
      }
    }
  ]
}
```

|   |   |
|---|---|
|필드명|설명|
|transaction_info.transaction_id|거래 고유 ID|
|transaction_info.transaction_initiation_date|거래 발생 일시 (ISO8601)|
|transaction_info.transaction_amount|거래 금액 (value + currency_code)|
|transaction_info.transaction_status|거래 상태 (S: 성공, P: 보류 등)|
|payer_info.email_address|구매자 이메일 주소|
