## YouTube 수익 확인을 위한 Google API 명세서 (AdSense + YouTube Analytics 기반)

### 📌 YouTube 정산 프로세스 요약

1. **AdSense 계정 ID 조회**: accounts.list() 호출
    
2. **확정 지급 내역 조회**: AdSense API로 유튜브 확정 지급액 필터링
    
3. **추정 수익 상세 분석**: YouTube Analytics API로 날짜별/영상별 추정 수익 조회
    
4. **실제 지급 vs 추정 수익 비교**: 총합 기준으로 지급 오차 확인
    

---

### ✅ 1. 확정 지급 내역 조회

**GET**  
`https://adsense.googleapis.com/v2/accounts/{account}/payments`

**Remarks**  
YouTube 수익을 포함한 실제 지급 확정 금액을 조회. `youtube-YYYY-MM-DD` 형식의 항목만 필터링하여 YouTube 수익만 추출.

**Request**

**▶ Path Parameters**

|파라미터|타입|필수|설명|
|---|---|---|---|
|account|String|Y|AdSense 계정 ID (`accounts/pub-xxxxxxxxxxxx`)|

**Response(JSON)**

```json
{
  "payments": [
    {
      "name": "accounts/pub-1234567890/payments/youtube-2024-06-21",
      "date": { "year": 2024, "month": 6, "day": 21 },
      "amount": "$82.57"
    },
    {
      "name": "accounts/pub-1234567890/payments/youtube-unpaid",
      "amount": "$9.34"
    }
  ]
}
```

|필드명|설명|
|---|---|
|name|지급 구분 이름. `youtube-YYYY-MM-DD` 형식이면 YouTube 확정 지급임|
|date|지급일 (year, month, day 형식)|
|amount|실제 지급 금액 (문자열 통화 포함, 예: "$82.57")|

---

### ✅ 2. YouTube 추정 수익 상세 조회

**GET**  
`https://youtubeanalytics.googleapis.com/v2/reports`

**Remarks**  
YouTube Analytics API를 사용하여 월별 추정 수익을 날짜/영상/국가별로 세분화하여 조회 가능

**Request**

**▶ Query Parameters**

|파라미터|타입|필수|예시값|설명|
|---|---|---|---|---|
|ids|String|Y|`channel==MINE`|인증된 채널 식별자|
|startDate|String|Y|`2024-06-01`|조회 시작일|
|endDate|String|Y|`2024-06-30`|조회 종료일|
|metrics|String|Y|`estimatedRevenue,estimatedAdRevenue,views`|조회할 수익 관련 지표|
|dimensions|String|N|`day`, `video`, `country` 등|분석 단위: 일/영상/국가 등|
|currency|String|N|`USD`|통화 단위|

**Response(JSON)**

```json
{
  "columnHeaders": [
    { "name": "day" },
    { "name": "estimatedRevenue" },
    { "name": "views" }
  ],
  "rows": [
    ["2024-06-01", "2.34", "1132"],
    ["2024-06-02", "2.01", "1100"]
  ]
}
```

|필드명|설명|
|---|---|
|columnHeaders|각 열의 이름 (지표명/구분자)|
|rows[][0]|날짜 (dimensions에 따라 day, video 등으로 대체 가능)|
|rows[][1~n]|지표 값 (예: estimatedRevenue, views 등)|

---

### ✅ 종합 비교 기준

|목적|사용 API|기준 설명|
|---|---|---|
|확정된 YouTube 지급액 확인|`adsense.accounts.payments.list`|`name` 필드에 `youtube-` 포함된 항목만 필터링|
|추정 수익 상세 분석|`youtubeAnalytics.reports.query`|dimensions + metrics 조합으로 세부 분석 가능|
|월별 비교|두 API의 동일 기간 데이터를 합산/비교|예상 수익 vs 실제 지급 비교하여 차이 분석|

---

### ✅ 실전 흐름 요약

```
1. accounts.list() → AdSense 계정 얻기
2. payments.list(parent=account) → 유튜브 지급 내역 필터링
3. reports.query(startDate=전월1일, endDate=전월말일) → YouTube 상세 수익 조회
4. 두 값을 비교하여 실제 지급과 추정 수익 비교 가능
```

---

