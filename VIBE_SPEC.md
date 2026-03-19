# CardSense Contracts — VIBE_SPEC
### Updated: 2026-03-19

## Purpose
This repository defines the single source of truth for data contracts shared between CardSense API and Extractor.

This repo contains no runtime code.

## Core Model: Promotion

| Field | Type | Notes |
|-------|------|-------|
| `promoId` | String | Stable logical promotion id |
| `promoVersionId` | String | Immutable version id |
| `title` | String | Human-readable promo title |
| `cardCode` | String | Stable card code |
| `cardName` | String | Card display name |
| `cardStatus` | String | ACTIVE, DISCONTINUED |
| `annualFee` | int | TWD |
| `applyUrl` | String nullable | Affiliate link |
| `bankCode` | BankCode | CTBC, ESUN, ... |
| `bankName` | String | Localized bank name |
| `category` | Category | DINING, TRANSPORT, ONLINE, OVERSEAS, SHOPPING, GROCERY, ENTERTAINMENT, OTHER |
| `channel` | String nullable | ONLINE, OFFLINE, ALL |
| `cashbackType` | CashbackType | PERCENT, FIXED, POINTS |
| `cashbackValue` | BigDecimal(5,2) | 3% = 3.00 |
| `minAmount` | int | Minimum spend |
| `maxCashback` | Integer nullable | Reward cap |
| `frequencyLimit` | FrequencyLimit | MONTHLY, QUARTERLY, YEARLY, ONCE, NONE |
| `requiresRegistration` | boolean | Whether benefit needs registration |
| `validFrom` | LocalDate | ISO-8601 |
| `validUntil` | LocalDate | ISO-8601 |
| `conditions` | List<Condition> | Structured conditions |
| `excludedConditions` | List<Condition> | Structured exclusions |
| `sourceUrl` | String | Original bank page URL |
| `rawTextHash` | String | SHA-256 of source text |
| `summary` | String | Human-readable summary |
| `extractorVersion` | String | e.g. extractor-0.1.0 |
| `extractedAt` | Instant | ISO-8601 datetime |
| `confidence` | BigDecimal(3,2) | 0.00–1.00 |
| `status` | String | ACTIVE, EXPIRED, REVOKED |

### Condition Object
```java
public record Condition(
    String type,
    String value,
    String label
) {}
```

## API DTO: RecommendationRequest
```java
public record RecommendationRequest(
    String category,
    Integer amount,
    List<String> cardCodes,
    List<String> registeredPromotionIds,
    List<BenefitUsage> benefitUsage,
    String location,
    LocalDate date
) {}

public record BenefitUsage(
    String promoVersionId,
    Integer consumedAmount
) {}
```

## API DTO: RecommendationResponse
```java
public record CardRecommendation(
    String cardName,
    String bankName,
    String cashbackType,
    BigDecimal cashbackValue,
    Integer estimatedReturn,
    String reason,
    String promotionId,
    String promoVersionId,
    LocalDate validUntil,
    List<Condition> conditions,
    String applyUrl
) {}
```
