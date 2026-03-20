# Recommendation Contracts

This directory defines the formal JSON schema for CardSense recommendation request / response payloads.

## Files
- `recommendation-request.schema.json`: Scenario-driven request contract with backward-compatible legacy top-level fields.
- `recommendation-response.schema.json`: Card-level recommendation response contract with comparison metadata and promotion breakdown.
- `recommendation-request.example.valid.json`: Valid request example.
- `recommendation-response.example.valid.json`: Valid response example.

## Design Rules
- Request compatibility is preserved: callers may use top-level `amount/category/location/date`, nested `scenario`, or both.
- `comparison.mode` makes the comparison strategy explicit rather than implicit.
- Response ranking is card-level even when a representative promotion is retained for backward compatibility.
- Break-even output is advisory metadata and must not override deterministic ranking for the current transaction amount.

## Relationship With Promotion Schema
`STACK_ALL_ELIGIBLE` must not rely on heuristic coexistence forever. It should be backed by explicit promotion metadata from `promotion-normalized.schema.json`, specifically the `stackability` object.