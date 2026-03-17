# CardSense Contracts — VIBE_SPEC
### Aligned with CardSense-Spec.md v1.0 | Updated: 2026-03-18

## Purpose
This repository defines the **single source of truth (SSOT)** for data contracts
shared between CardSense API and Extractor.

This repo contains **no runtime code**.
It exists to prevent schema drift and enable parallel development.

---

## Scope (DO)
- Define Java records for normalized credit card promotions, cards, and banks
- Define enums (category, channel, cashback type, frequency, bank code)
- Define API request/response DTOs
- Provide valid and invalid JSON examples for testing
- Publish to Maven Local / GitHub Packages for downstream repos

## Out of Scope (DO NOT)
- No crawling or extraction logic
- No API endpoints
- No database code or migrations
- No LLM prompts or inference logic

---

## Required Project Structure

```
cardsense-contracts/
├── src/main/java/com/cardsense/contracts/
│   ├── model/
│   │   ├── Bank.java                  ← Java record
│   │   ├── Card.java                  ← Java record
│   │   └── Promotion.java             ← Java record (core)
│   ├── dto/
│   │   ├── RecommendationRequest.java
│   │   └── RecommendationResponse.java
│   └── enums/
│       ├── Category.java
│       ├── CashbackType.java
│       ├── CardType.java
│       ├── FrequencyLimit.java
│       └── BankCode.java
├── src/test/resources/
│   ├── promotion-example.valid.json
│   └── promotion-example.invalid.json
├── build.gradle.kts
└── README.md
```

---

## Core Model: Promotion

The `Promotion` record MUST define:

| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | Primary key |
| `cardId` | UUID | FK to cards |
| `version` | int | Auto-increment per promo; same card+category+valid_from = unique |
| `promoVersionId` | String (UUID) | Immutable version identifier; new value for any semantic change |
| `title` | String (max 200) | Human-readable promo title |
| `category` | Category enum | DINING, TRANSPORT, ONLINE, OVERSEAS, SHOPPING, GROCERY, ENTERTAINMENT, OTHER |
| `cashbackType` | CashbackType enum | PERCENT, FIXED, POINTS |
| `cashbackValue` | BigDecimal(5,2) | Percentage: 3% = 3.00 |
| `minAmount` | int | Minimum spend threshold (TWD), default 0 |
| `maxCashback` | Integer (nullable) | Reward cap (TWD) |
| `frequencyLimit` | FrequencyLimit enum | MONTHLY, QUARTERLY, YEARLY, ONCE, NONE |
| `requiresRegistration` | boolean | Whether user must register/activate to receive benefit |
| `validFrom` | LocalDate | ISO-8601 |
| `validUntil` | LocalDate | ISO-8601 |
| `conditions` | List<String> | Structured conditions (JSONB in DB) |
| `excludedConditions` | List<String> | Explicit exclusions |
| `sourceUrl` | String (URI) | Original bank page URL |
| `sourceText` | String | Raw promo text — retained in staging for auditability |
| `rawTextHash` | String | SHA-256 of source text; same hash = same semantic content |
| `summary` | String (max 300) | Human-readable summary |
| `extractorVersion` | String | e.g. "extractor-1.0.0" |
| `extractionModel` | String | e.g. "gemini-flash-2.0" |
| `extractedAt` | Instant | ISO-8601 datetime |
| `confidence` | BigDecimal(3,2) | 0.00–1.00 |
| `verified` | boolean | Human-verified, default false |
| `status` | String | ACTIVE, EXPIRED, REVOKED |
| `createdAt` | Instant | Auto-set |
| `updatedAt` | Instant | Auto-set |

### Constraints
- No free-form text except `summary` and `sourceText`
- Monetary values are integers (TWD, smallest unit)
- Missing data must be explicit (null, not absent)
- `cashbackValue` is always percentage format (3% = 3.00)
- Date format: YYYY-MM-DD (ISO-8601)

---

## Core Model: Card

| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | PK |
| `bankId` | UUID | FK to banks |
| `name` | String (max 100) | e.g. "中信 LINE Pay 卡" |
| `cardType` | CardType enum | VISA, MASTERCARD, JCB |
| `annualFee` | int | TWD, default 0 |
| `applyUrl` | String (nullable) | Affiliate link |
| `status` | String | ACTIVE, DISCONTINUED |

## Core Model: Bank

| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | PK |
| `code` | BankCode enum | CTBC, ESUN, TAISHIN, CATHAY, MEGA, FUBON, FIRST, SINOPAC, TPBANK, UBOT |
| `nameZh` | String | "中國信託" |
| `nameEn` | String | "CTBC" |
| `website` | String | Bank homepage |

---

## API DTO: RecommendationRequest

```java
public record RecommendationRequest(
    String category,        // Category enum value
    Integer amount,         // Transaction amount (TWD)
    List<String> cardCodes, // User's card codes (optional, empty = all)
    String location,        // Location (optional)
    LocalDate date          // Transaction date (optional, default = today)
) {}
```

## API DTO: RecommendationResponse

```java
public record RecommendationResponse(
    String requestId,
    List<CardRecommendation> recommendations,
    LocalDateTime generatedAt,
    String disclaimer       // Legal disclaimer (always present)
) {}

public record CardRecommendation(
    String cardName,
    String bankName,
    String cashbackType,
    BigDecimal cashbackValue,
    Integer estimatedReturn,  // Estimated reward (TWD)
    String reason,            // Human-readable recommendation reason
    String promotionId,       // Traceable to promotions table
    String promoVersionId,    // Immutable version for audit
    LocalDate validUntil,
    List<String> conditions,
    String applyUrl           // Affiliate link (optional, nullable)
) {}
```

---

## Enum Definitions

### Category
```
DINING, TRANSPORT, ONLINE, OVERSEAS, SHOPPING, GROCERY, ENTERTAINMENT, OTHER
```

### CashbackType
```
PERCENT, FIXED, POINTS
```

### FrequencyLimit
```
MONTHLY, QUARTERLY, YEARLY, ONCE, NONE
```

### CardType
```
VISA, MASTERCARD, JCB, AMEX, UNIONPAY
```

### BankCode
```
CTBC, ESUN, TAISHIN, CATHAY, MEGA, FUBON, FIRST, SINOPAC, TPBANK, UBOT
```

---

## Versioning Rules

- `promoVersionId`: new UUID for any semantic field change
- Same `rawTextHash` + re-run → no-op (skip insert)
- Old versions are **immutable** — never update existing records
- Any schema field rename/removal = **BREAKING CHANGE** → requires major version bump
- `version` integer tracks logical version count per promo_id

---

## Legal: Disclaimer Requirement

All downstream consumers (API responses, B2C pages) MUST include:
> 「CardSense 提供信用卡優惠比較資訊，不構成金融建議。實際回饋依各銀行公告為準，請以銀行官網資訊為最終依據。」

This string SHOULD be defined as a constant in this contracts repo.

---

## Success Criteria
- Valid example passes Java record deserialization
- Invalid example fails validation with clear error
- API and Extractor repos can implement against these contracts without guessing intent
- Published to Maven Local; both downstream repos declare dependency

---

## Agent Instructions
- You may only modify files in this repository
- Do NOT introduce runtime logic or database code
- Treat this repo as a constitution, not a playground
- All enum values must exactly match what Extractor produces and API consumes
