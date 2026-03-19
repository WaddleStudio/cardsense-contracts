# CardSense Contracts

CardSense 平台的共用資料契約庫。

本 repo 定義 CardSense API 和 Extractor 共用的資料契約、schema、DTOs、列舉，作為所有資料結構的唯一真實來源（SSOT），防止 schema 漂移並支援平行開發。

## 本 repo 的職責
- 定義正規化優惠資料模型與 JSON schema
- 定義 API 請求/回應 DTOs
- 定義列舉型別：`Category`、`CashbackType`、`FrequencyLimit`、`CardType`、`BankCode`
- 提供合法/非法 JSON 範例供下游 repo 測試

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

## 核心列舉型別

| 列舉 | 值 |
|------|-----|
| Category | DINING, TRANSPORT, ONLINE, OVERSEAS, SHOPPING, GROCERY, ENTERTAINMENT, OTHER |
| CashbackType | PERCENT, FIXED, POINTS |
| FrequencyLimit | MONTHLY, QUARTERLY, YEARLY, ONCE, NONE |
| CardType | VISA, MASTERCARD, JCB, AMEX, UNIONPAY |
| BankCode | CTBC, ESUN, TAISHIN, CATHAY, MEGA, FUBON, FIRST, SINOPAC, TPBANK, UBOT |
