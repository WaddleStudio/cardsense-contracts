# CardSense Contracts — VIBE_SPEC
### Updated: 2026-03-20

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
| `stackability` | Stackability nullable | Explicit coexistence / exclusivity metadata for deterministic stacking |
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
public class RecommendationRequest {
    Integer amount;
    String category;
    List<String> cardCodes;
    List<String> registeredPromotionIds;
    List<BenefitUsage> benefitUsage;
    String location;
    LocalDate date;
    RecommendationScenario scenario;
    RecommendationComparisonOptions comparison;
}

public class RecommendationScenario {
    Integer amount;
    String category;
    LocalDate date;
    String location;
    String channel;
    String merchantName;
    String merchantId;
    String paymentMethod;
    Integer installmentCount;
    Boolean newCustomer;
    String customerSegment;
    String membershipTier;
    List<String> tags;
    Map<String, String> attributes;
}

public class RecommendationComparisonOptions {
    ComparisonMode mode;
    Boolean includePromotionBreakdown;
    Boolean includeBreakEvenAnalysis;
    Integer maxResults;
    List<String> compareCardCodes;
}

public enum ComparisonMode {
    BEST_SINGLE_PROMOTION,
    STACK_ALL_ELIGIBLE
}

public class BenefitUsage {
    String promoVersionId;
    Integer consumedAmount;
}
```

## API DTO: RecommendationResponse
```java
public class RecommendationResponse {
    String requestId;
    RecommendationScenario scenario;
    RecommendationComparisonSummary comparison;
    List<CardRecommendation> recommendations;
    LocalDateTime generatedAt;
    String disclaimer;
}

public class CardRecommendation {
    String cardCode;
    String cardName;
    String bankCode;
    String bankName;
    String cashbackType;
    BigDecimal cashbackValue;
    Integer estimatedReturn;
    Integer matchedPromotionCount;
    String rankingMode;
    String reason;
    String promotionId;
    String promoVersionId;
    LocalDate validUntil;
    List<Condition> conditions;
    List<PromotionRewardBreakdown> promotionBreakdown;
    String applyUrl;
}
```

## Stackability Object
```java
public class Stackability {
    String benefitLayer;
    String relationshipMode;
    String groupId;
    Integer priority;
    List<String> requiresPromoVersionIds;
    List<String> excludesPromoVersionIds;
    List<String> stackWithPromoVersionIds;
    String notes;
}
```

### Stackability Rules
- `benefitLayer` tells the engine whether the promotion is a base reward, bonus, milestone, welcome reward, installment reward, merchant-funded booster, or manual-only artifact.
- `relationshipMode=ALWAYS_STACKABLE` means the promotion can coexist unless an explicit exclusion blocks it.
- `relationshipMode=MUTUALLY_EXCLUSIVE` means promotions in the same `groupId` must not be summed together.
- `relationshipMode=CONDITIONAL` means the engine must resolve `requiresPromoVersionIds`, `excludesPromoVersionIds`, and optional `stackWithPromoVersionIds` before stacking.
- `relationshipMode=MANUAL_REVIEW` means the contract intentionally opts out of automatic deterministic stacking.
- Any semantic change to stackability metadata requires a new `promoVersionId`.
