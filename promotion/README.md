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

## Versioning Rules
- `promoVersionId` must change for any semantic change.
- `promoId` should remain stable for the same logical promotion.
- `rawTextHash` is used for source-text change detection.
- Published versions are immutable.

## Validation
All extractor output must validate against `promotion-normalized.schema.json` before it is loaded into downstream systems.
