## Shopee 지급확정 API 명세표 (전체 프로세스 기반)

---

### ✅ 1.

**POST**  
`https://open.sandbox.test-stable.shopee.com/auth`  
**Remarks**  
Shopee 판매자 인증을 위한 고객 동의 창 (Consent URL)을 호출. 사용자 동의 후 `code`가 redirect URI로 반환.

**Request**  
방문하는 URL 호출 (빈 body)

| Parameters | 타입               | 필수여부 | 설명                              |
| ---------- | ---------------- | ---- | ------------------------------- |
| partner_id | String           | Y    | Shopee 파트너 ID                   |
| redirect   | URL              | Y    | 인증 완료 후 되돌 경로                   |
| timestamp  | Integer (UNIX 초) | Y    | UNIX timestamp                  |
| sign       | String           | Y    | HMAC-SHA256 서명 (base string 기본) |

**Response(JSON)**  
개발자 redirect URI로 `code`, `shop_id`, `partner_id` 프리시트 발어:

```http
https://paygos.ibk.co.kr/shopee/oauth/callback?code=abc1234567&shop_id=12345678&partner_id=999999abc1234567&shop_id=12345678&partner_id=999999
```

---

### ✅ 2.

**POST**  
`https://openplatform.sandbox.test-stable.shopee.sg/api/v2/auth/token/get`  
**Remarks**  
동의 후 발행된 `code`를 기본으로 access token 및 refresh token 첫 번째 발급

**Request**

|Headers|타입|필수여부|설명|
|---|---|---|---|
|Content-Type|String|Y|application/json|

|Parameters|타입|필수여부|설명|
|---|---|---|---|
|code|String|Y|동의 완료 후 받은 `code` 값|
|partner_id|Integer|Y|Shopee 개발자 파트너 ID|
|shop_id|Integer|Y|동의 완료 시 받은 shop_id|
|partner_key|String|Y|Shopee 앱 secret key|
|sign|String|Y|서명 (SHA256)|
|timestamp|Integer (UNIX 초)|Y|요청 시간|

**Response(JSON)**

```json
{
  "access_token": "abc_token_xyz",
  "refresh_token": "refresh_token_xyz",
  "expire_in": 86400,
  "token_type": "Bearer"
}
```

---

### ✅ 3.

**POST**  
`https://openplatform.sandbox.test-stable.shopee.sg/api/v2/auth/access_token/get`  
**Remarks**  
저장된 refresh_token을 사용하여 새로운 access token 재발급

**Request**

|Headers|타입|필수여부|설명|
|---|---|---|---|
|Content-Type|String|Y|application/json|

| Parameters    | 타입               | 필수여부 | 설명                  |
| ------------- | ---------------- | ---- | ------------------- |
| refresh_token | String           | Y    | 저장된 refresh_token 값 |
| partner_id    | Integer          | Y    | Shopee 개발자 파트너 ID   |
| shop_id       | Integer          | Y    | 판매자 샵 ID            |
| partner_key   | String           | Y    | Shopee 앱 secret key |
| sign          | String           | Y    | SHA256 서명           |
| timestamp     | Integer (UNIX 초) | Y    | 요청 시간               |

**Response(JSON)**

```json
{
  "access_token": "new_access_token_123456",
  "expire_in": 86400
}
```

---

### ✅ 4.

**POST**  
`https://openplatform.sandbox.test-stable.shopee.sg/api/v2/payment/get_payout_detail`  
**Remarks**  
지정된 기간 내 확정 지금된 금액(payout_amount)을 조회

**Request**

|Headers|타입|필수여부|설명|
|---|---|---|---|
|Content-Type|String|Y|application/json|

|Parameters|타입|필수여부|설명|
|---|---|---|---|
|partner_id|String|Y|Shopee 파트너 ID|
|timestamp|Integer (UNIX 초)|Y|요청 시간|
|access_token|String|Y|access_token|
|shop_id|Integer|Y|판매자 샵 ID|
|payout_time_from|Integer (UNIX 초)|Y|조회 시작 시간|
|payout_time_to|Integer (UNIX 초)|Y|조회 종료 시간|
|page_size / page_no|Int|N|페이지|

**Response(JSON)**

```json
{
  "response": {
    "payout_list": [
      {
        "payout_info": {
          "payout_amount": 25678.64,
          "payout_time": 1651842208,
          "payee_id": "279016275538"
        },
        "escrow_list": [],
        "offline_adjustment_list": []
      }
    ]
  }
}
```

---

### ✅ 5.

**POST**  
`https://openplatform.sandbox.test-stable.shopee.sg/api/v2/payment/get_escrow_list`  
**Remarks**  
지금 시점 ±2주 이내 release된 주문(order_sn) 목록을 조회

**Request**

|Headers|타입|필수여부|설명|
|---|---|---|---|
|Content-Type|String|Y|application/json|

|Parameters|타입|필수여부|설명|
|---|---|---|---|
|partner_id|String|Y|Shopee 파트너 ID|
|timestamp|Integer (UNIX 초)|Y|요청 시간|
|access_token|String|Y|access_token|
|payee_id|String|Y|get_payout_detail의 payee_id 값|
|release_time_from|Integer (UNIX 초)|Y|release 조회 시작 시간|
|release_time_to|Integer (UNIX 초)|Y|release 조회 종료 시간|

**Response(JSON)**

```json
{
  "response": {
    "escrow_list": [
      {
        "order_sn": "220404NF3CFFNY",
        "escrow_release_time": 1651849648,
        "escrow_amount": 20865
      }
    ]
  }
}
```

---

### ✅ 6.

**POST**  
`https://openplatform.sandbox.test-stable.shopee.sg/api/v2/payment/get_escrow_detail`  
**Remarks**  
특정 주문(order_sn)에 대한 상세 정산 내역(판매가, 수수료 등)을 조회

**Request**

|Headers|타입|필수여부|설명|
|---|---|---|---|
|Content-Type|String|Y|application/json|

|Parameters|타입|필수여부|설명|
|---|---|---|---|
|partner_id|String|Y|Shopee 파트너 ID|
|timestamp|Integer (UNIX 초)|Y|요청 시간|
|access_token|String|Y|access_token|
|payee_id|String|Y|판매자 ID|
|order_sn|String|Y|조회할 주문번호|

**Response(JSON)**

```json
{
  "response": {
    "order_sn": "220404NF3CFFNY",
    "buyer_username": "shopee_user",
    "item_list": [
      {
        "item_name": "Example Product",
        "item_price": 25000
      }
    ],
    "income": {
      "order_income": 50000,
      "commission_fee": 2000,
      "service_fee": 1000
    }
  }
}
```
