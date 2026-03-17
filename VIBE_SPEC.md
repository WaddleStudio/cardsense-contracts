# CardSense Contracts — VIBE_SPEC

## Purpose
This repository defines the **single source of truth (SSOT)** for data contracts
shared between CardSense API and Extractor.

This repo contains **no runtime code**.
It exists to prevent schema drift and enable parallel development.

---

## Scope (DO)
- Define JSON Schema for normalized credit card promotions
- Define taxonomies (category, channel, frequency)
- Define versioning and breaking-change rules
- Provide valid and invalid examples

## Out of Scope (DO NOT)
- No crawling or extraction logic
- No API endpoints
- No database code
- No LLM prompts or inference logic

---

## Required Files

promotion/
├─ promotion-normalized.schema.json
├─ promotion-example.valid.json
├─ promotion-example.invalid.json
└─ README.md

taxonomy/
├─ category-taxonomy.json
├─ channel-taxonomy.json
└─ frequency-taxonomy.json


---

## Core Schema Requirements

`promotion-normalized.schema.json` MUST define:

- promo_id               (string, stable logical id)
- promo_version_id       (string, immutable version id)
- card_id                (string)
- bank                   (string)
- categories             (array[string])
- channel                (string)
- start_date             (ISO-8601 date)
- end_date               (ISO-8601 date)
- min_amount             (integer, >= 0)
- reward_type            (cashback | points | miles)
- reward_rate            (number, 0.0–1.0)
- reward_cap             (integer | null)
- frequency_limit        (monthly | quarterly | yearly | once)
- requires_registration  (boolean)
- excluded_conditions    (array[string])
- source_url             (string, uri)
- summary                (string, max 300 chars)
- raw_text_hash          (string)
- extractor_version      (string)
- extracted_at           (ISO-8601 datetime)
- confidence             (number, 0.0–1.0)

### Constraints
- No free-form text except `summary`
- Monetary values are integers (smallest currency unit)
- Missing data must be explicit (null vs missing)
- Schema must be strict (no additionalProperties)

---

## Versioning Rules

- promo_id: logical identity (bank + card + campaign)
- promo_version_id: new value for any semantic change
- Same raw_text_hash → same semantic content
- Any schema field rename/removal = BREAKING CHANGE

---

## Success Criteria
- Schema validates valid example
- Schema rejects invalid example
- API & Extractor can implement without guessing intent

---

## Agent Instructions
- You may only modify files in this repository
- Do NOT introduce runtime logic
- Treat this repo as a constitution, not a playground