# CardSense Contracts

CardSense 平台的共用資料契約庫。

本 repo 定義 CardSense API 和 Extractor 共用的資料契約、schema、DTOs、列舉，作為所有資料結構的唯一真實來源（SSOT），防止 schema 漂移並支援平行開發。

## 本 repo 的職責
- 定義正規化優惠資料模型與 JSON schema
- 定義 API 請求/回應 DTOs
- 定義列舉型別：`Category`、`CashbackType`、`FrequencyLimit`、`CardType`、`BankCode`
- 提供合法/非法 JSON 範例供下游 repo 測試

## 目錄
- `promotion/`: normalized promotion schema、範例與 stackability metadata 設計
- `recommendation/`: recommendation request / response 正式 JSON schema 與範例
- `taxonomy/`: category、channel、frequency taxonomy

## 本 repo 不做的事
- 不包含爬蟲或資料擷取邏輯
- 不包含 API 端點實作
- 不包含資料庫程式碼或 migration
- 不包含任何 runtime 邏輯

## Promotion 契約重點
- 使用 camelCase 欄位命名
- `conditions` / `excludedConditions` 採用結構化 condition object
- `cashbackValue` 一律以百分比格式表示，例如 `3.00`
- `promoVersionId` 為語義版本識別碼，任何語義變更都必須更新
- Extractor 與 API 必須以同一份 schema 交換資料

## Recommendation API 契約方向
API request / response 目前已演進為 scenario-driven contract，重點如下：

- request 同時支援 legacy top-level `amount/category/location/date` 與 nested `scenario`
- `scenario` 可攜帶通路、商戶、付款方式、新戶狀態、會員層級、tags 與自由擴充 attributes
- `comparison.mode` 明確區分 `BEST_SINGLE_PROMOTION` 與 `STACK_ALL_ELIGIBLE`
- response 以 card-level recommendation 為主，保留代表 promotion 以維持相容性
- response 可回傳 `promotionBreakdown` 與 `breakEvenAnalyses`，讓前端或下游系統解釋比較結果

這代表 Recommendation contract 不應再被視為「單 promotion 排名結果」，而是「特定情境下的卡片比較結果」。

目前 shared contracts 已補上 `promotion.stackability`，用來顯式描述：
- 哪些優惠永遠可疊加
- 哪些優惠屬於互斥群組
- 哪些優惠需要依賴其他優惠才可生效
- 哪些優惠需要人工判斷，不應進 deterministic stack mode

## 核心列舉型別

| 列舉 | 值 |
|------|-----|
| Category | DINING, TRANSPORT, ONLINE, OVERSEAS, SHOPPING, GROCERY, ENTERTAINMENT, OTHER |
| CashbackType | PERCENT, FIXED, POINTS |
| FrequencyLimit | MONTHLY, QUARTERLY, YEARLY, ONCE, NONE |
| CardType | VISA, MASTERCARD, JCB, AMEX, UNIONPAY |
| BankCode | CTBC, ESUN, TAISHIN, CATHAY, MEGA, FUBON, FIRST, SINOPAC, TPBANK, UBOT |
