# CardSense Contracts

CardSense 平台的共用資料契約庫。

本 repo 定義 CardSense API 和 Extractor 共用的 **Java records、enums、DTOs**，作為所有資料結構的唯一真實來源（SSOT），防止 schema 漂移並支援平行開發。

## 本 repo 的職責
- 定義正規化優惠資料模型（`Promotion`、`Card`、`Bank` Java records）
- 定義 API 請求/回應 DTOs（`RecommendationRequest`、`RecommendationResponse`）
- 定義列舉型別：`Category`、`CashbackType`、`FrequencyLimit`、`CardType`、`BankCode`
- 提供合法/非法 JSON 範例供下游 repo 測試
- 發佈至 Maven Local / GitHub Packages 供依賴引用

## 本 repo 不做的事
- 不包含爬蟲或資料擷取邏輯
- 不包含 API 端點實作
- 不包含資料庫程式碼或 migration
- 不包含 LLM prompt 或推理邏輯
- 不包含任何 runtime 邏輯

## 技術棧
| 元件 | 選擇 |
|------|------|
| 語言 | Java 21 |
| 建構 | Gradle (Kotlin DSL) |
| 發佈 | Maven Local / GitHub Packages |

## 專案結構

```
cardsense-contracts/
├── src/main/java/com/cardsense/contracts/
│   ├── model/          ← Promotion, Card, Bank (Java records)
│   ├── dto/            ← RecommendationRequest, RecommendationResponse
│   └── enums/          ← Category, CashbackType, FrequencyLimit, CardType, BankCode
├── src/test/resources/
│   ├── promotion-example.valid.json
│   └── promotion-example.invalid.json
├── build.gradle.kts
├── VIBE_SPEC.md        ← Agent 導向的完整規格
└── README.md
```

## 核心列舉型別

| 列舉 | 值 |
|------|-----|
| Category | DINING, TRANSPORT, ONLINE, OVERSEAS, SHOPPING, GROCERY, ENTERTAINMENT, OTHER |
| CashbackType | PERCENT, FIXED, POINTS |
| FrequencyLimit | MONTHLY, QUARTERLY, YEARLY, ONCE, NONE |
| CardType | VISA, MASTERCARD, JCB, AMEX, UNIONPAY |
| BankCode | CTBC, ESUN, TAISHIN, CATHAY, MEGA, FUBON, FIRST, SINOPAC, TPBANK, UBOT |

## 版本規則
- `promoVersionId`：任何語義欄位變更 → 產生新 UUID
- 相同 `rawTextHash` → 語義內容相同 → 跳過寫入
- 欄位重新命名/移除 = **破壞性變更** → 主版本號遞增
- 舊版本不可變（immutable）

## 法律免責聲明常數

所有下游消費者（API 回應、B2C 頁面）**必須**包含以下免責聲明：

> 「CardSense 提供信用卡優惠比較資訊，不構成金融建議。實際回饋依各銀行公告為準，請以銀行官網資訊為最終依據。」

此字串在本 repo 中定義為常數，確保全平台一致。

## 使用方式

```kotlin
// build.gradle.kts（下游 repo）
dependencies {
    implementation("com.cardsense:cardsense-contracts:1.0.0")
}
```

## 關聯 Repository
| Repo | 角色 | 依賴 contracts？ |
|------|------|-----------------|
| [cardsense-api](https://github.com/skywalker6666/cardsense-api) | 對外推薦 API | ✅ 是 |
| [cardsense-extractor](https://github.com/skywalker6666/cardsense-extractor) | 優惠擷取 pipeline | ✅ 是 |
| [fleet-command](https://github.com/skywalker6666/fleet-command) | 專案規格書庫（CardSense-Spec.md） | — |

---

*Owner: Alan | 隸屬 [CardSense](https://github.com/skywalker6666?tab=repositories&q=cardsense) 平台*
