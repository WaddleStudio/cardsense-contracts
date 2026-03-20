# Promotion Contracts

This directory defines the normalized promotion payload shared by CardSense API and Extractor.

## Files
- `promotion-normalized.schema.json`: JSON Schema for the normalized promotion object.
- `promotion-example.valid.json`: Valid camelCase example.
- `promotion-example.invalid.json`: Invalid example for negative-path tests.

## Current Shape
The normalized promotion model uses camelCase fields aligned with the latest VIBE spec and current API / extractor implementation.

Key fields:
- `promoId`, `promoVersionId`
- `cardCode`, `cardName`, `cardStatus`, `annualFee`, `applyUrl`
- `bankCode`, `bankName`
- `category`, `channel`
- `cashbackType`, `cashbackValue`, `minAmount`, `maxCashback`
- `frequencyLimit`, `requiresRegistration`
- `stackability`
- `validFrom`, `validUntil`
- `conditions`, `excludedConditions`
- `sourceUrl`, `summary`, `rawTextHash`, `extractorVersion`, `extractedAt`, `confidence`, `status`

## Structured Conditions
`conditions` and `excludedConditions` are arrays of structured objects, not magic strings.

Condition object shape:
```json
{
  "type": "LOCATION_ONLY",
  "value": "TAIPEI",
  "label": "限 TAIPEI 適用"
}
```

Recommended types:
- `TEXT`
- `LOCATION_ONLY`
- `LOCATION_EXCLUDE`
- `CATEGORY_EXCLUDE`
- `MIN_SPEND`
- `REGISTRATION_REQUIRED`
- `FREQUENCY_LIMIT`

## Stackability Metadata
`stackability` makes promotion coexistence explicit so downstream recommendation engines do not need to guess whether `STACK_ALL_ELIGIBLE` is safe.

Shape:

```json
{
  "benefitLayer": "BASE",
  "relationshipMode": "MUTUALLY_EXCLUSIVE",
  "groupId": "ctbc-demo-online-base-2026q2",
  "priority": 100,
  "requiresPromoVersionIds": [],
  "excludesPromoVersionIds": ["ver_other_base"],
  "stackWithPromoVersionIds": ["ver_bonus_001"],
  "notes": "主優惠與同組 base benefit 擇一，但可與指定加碼並存。"
}
```

Semantics:
- `benefitLayer`: distinguishes base reward, bonus reward, milestone reward, welcome reward, installment reward, or manual-only logic.
- `relationshipMode=ALWAYS_STACKABLE`: can coexist with other eligible promotions unless an explicit exclusion blocks it.
- `relationshipMode=MUTUALLY_EXCLUSIVE`: promotions in the same `groupId` cannot all apply together; the engine must choose one deterministic winner.
- `relationshipMode=CONDITIONAL`: coexistence depends on `requiresPromoVersionIds`, `excludesPromoVersionIds`, and optional `stackWithPromoVersionIds` allow-list.
- `relationshipMode=MANUAL_REVIEW`: do not auto-stack in deterministic recommendation mode.
- `priority`: deterministic tie-break helper for future engines when multiple candidates in the same exclusivity group remain eligible.

## Versioning Rules
- `promoVersionId` must change for any semantic change.
- `promoId` should remain stable for the same logical promotion.
- `rawTextHash` is used for source-text change detection.
- Published versions are immutable.
- Any change to `stackability` metadata is a semantic change and therefore requires a new `promoVersionId`.

## Validation
All extractor output must validate against `promotion-normalized.schema.json` before it is loaded into downstream systems.
