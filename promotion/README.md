# Promotion Contracts

This directory contains the core data contracts for CardSense promotions. It serves as the single source of truth for how promotion data is structured, exchanged, and validated.

## Files

- `promotion-normalized.schema.json`: Strict JSON Schema defining the structure of a normalized promotion.
- `promotion-example.valid.json`: A valid example that passes schema validation.
- `promotion-example.invalid.json`: An invalid example that fails schema validation (for testing purposes).

## Field Semantics

### Identifiers
- `promo_id`: A logical ID that uniquely identifies the promotion concept (e.g., "bank-a-card-x-dining-promo"). This ID should remain stable across minor updates.
- `promo_version_id`: A specific version of the promotion content. Any semantic change (e.g., reward rate change) MUST generate a new version ID.
- `raw_text_hash`: A hash of the raw extracted text. This is used to detect if the source content has changed.

### Reward Details
- `reward_type`: The nature of the benefit (`cashback`, `points`, `miles`).
- `reward_rate`: The benefit value as a decimal (e.g., `0.05` for 5%).
- `min_amount` & `reward_cap`: Monetary values are always integers representing the smallest currency unit (e.g., cents).

### Taxonomies
Reference the `../taxonomy/` directory for allowed values in:
- `categories` (see `category-taxonomy.json`)
- `channel` (see `channel-taxonomy.json`)
- `frequency_limit` (see `frequency-taxonomy.json`)

## Versioning Rules

We follow semantic versioning principles for the schema itself, and strict immutability for data records.

1.  **Breaking Changes**: Any change that could cause a previously valid consumer to fail.
    - Removing a field.
    - Renaming a field.
    - Changing a field's type.
    - Making an optional field required.
    - Removing a value from an enum (e.g., removing "dining" from categories).

2.  **Non-Breaking Changes**:
    - Adding a new optional field.
    - Adding a new value to an enum (e.g., adding "medical" to categories).
    - Relaxing a constraint (e.g., increasing `maxLength`).

3.  **Data Immutability**:
    - Once a `promo_version_id` is published, its data is immutable.
    - If the source data changes, a NEW `promo_version_id` must be created.

## Validation

All data produced by the extractor MUST validate against `promotion-normalized.schema.json` before being persisted or sent to the API.
