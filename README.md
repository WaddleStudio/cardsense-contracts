# CardSense Contracts

CardSense 平台的共用資料契約庫，作為所有資料結構的唯一真實來源（SSOT），防止 schema 漂移並支援平行開發。

## 職責

**負責**：
- 定義正規化 promotion schema 與 JSON Schema
- 定義 recommendation request / response DTOs
- 定義列舉型別（Category、CashbackType、FrequencyLimit、CardType、BankCode）
- 定義 `promotion.stackability` metadata 規格
- 提供合法 / 非法 JSON 範例供下游 repo 測試

**不負責**：
- 爬蟲或資料擷取邏輯
- API 端點實作
- 資料庫程式碼或 migration
- 任何 runtime 邏輯

## 技術棧

- JSON Schema
- Markdown 文件

## 快速開始

```bash
cd cardsense-contracts
# 瀏覽 promotion/ 目錄下的 normalized promotion schema
# 瀏覽 recommendation/ 目錄下的 request / response schema
# 瀏覽 taxonomy/ 目錄下的 category / subcategory / channel / frequency 定義
```

本 repo 無 runtime 依賴，直接閱讀 schema 檔案即可。下游 repo 以這裡的 schema 作為 normalize / validate 的契約來源。

## 專案結構

```text
cardsense-contracts/
├── benefit-plan/       # benefit-plan.schema.json（權益切換卡方案定義）
├── promotion/          # normalized promotion schema、範例、stackability metadata
├── recommendation/     # recommendation request / response JSON schema 與範例
└── taxonomy/           # category、subcategory、channel、frequency taxonomy
```

## 設計重點

### Promotion 契約

- 使用 camelCase 欄位命名
- `conditions` / `excludedConditions` 採用結構化 condition object
- `cashbackValue` 一律以百分比格式表示（如 `3.00`）
- `promoVersionId` 為語義版本識別碼，任何語義變更都必須更新
- Extractor 與 API 必須以同一份 schema 交換資料

### Recommendation 契約

- request 同時支援 legacy top-level 欄位與 nested `scenario`
- `scenario` 可攜帶通路、商戶、付款方式、會員層級、tags 與自由擴充 attributes
- 固定使用疊加優惠計算（`STACK_ALL_ELIGIBLE`），已移除 `BEST_SINGLE_PROMOTION`
- response 以 card-level recommendation 為主，可回傳 `promotionBreakdown`、`breakEvenAnalyses`、`activePlan`

### Stackability Metadata

描述優惠間的可疊加關係：
- 永遠可疊加
- 互斥群組
- 依賴其他優惠才可生效
- 需人工判斷，不進 deterministic stack mode

### 核心列舉

| 列舉 | 值 |
|------|-----|
| Category | DINING, TRANSPORT, ONLINE, OVERSEAS, SHOPPING, GROCERY, ENTERTAINMENT, OTHER |
| Subcategory | GENERAL 及 taxonomy/subcategory-taxonomy.json 中定義的細分類 |
| CashbackType | PERCENT, FIXED, POINTS |
| FrequencyLimit | MONTHLY, QUARTERLY, YEARLY, ONCE, NONE |
| CardType | VISA, MASTERCARD, JCB, AMEX, UNIONPAY |
| BankCode | CTBC, ESUN, TAISHIN, CATHAY, MEGA, FUBON, FIRST, SINOPAC, TPBANK, UBOT |

### Taxonomy 原則

- `category` 保持精簡與穩定，作為 API / UI / recommendation 主骨架
- `subcategory` 承接銀行別、通路別與情境別的細節差異
- 新需求預設先進 `subcategory`
- 只有當某個情境長期需要直接成為 API / UI 主入口時，才考慮升格為新的 `category`

## 與其他子專案的關係

- **cardsense-extractor**：以本 repo 的 promotion schema 為 normalize / validate 的契約來源
- **cardsense-api**：以本 repo 的 recommendation schema 為 request / response 格式依據

## 已知限制

- `POINTS` 型別尚未定義銀行別點數折現規則，目前沿用 `PERCENT` 計算路徑
- `stackability` metadata 設計已完成，但部分銀行的實際標註資料仍在累積中
