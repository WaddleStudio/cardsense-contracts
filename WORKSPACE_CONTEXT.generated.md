# cardsense-contracts Workspace Context

> DO NOT EDIT. Generated from `fleet-command/workspace/workspace.manifest.json`.

## Summary

- Workspace: `cardsense-workspace`
- Source of truth: `fleet-command`
- Repo role: `contracts`
- Purpose: Shared contracts, schemas, and taxonomies for CardSense repos

## Read First

- `README.md`
- `VIBE_SPEC.md`
- `promotion/promotion-normalized.schema.json`
- `recommendation/recommendation-request.schema.json`
- `recommendation/recommendation-response.schema.json`

## Verification

- `verify`
  - `rg -n "TRAVEL" promotion recommendation taxonomy`

## Workspace Policies

- Python package management: Use uv for Python dependency management and Python command execution across the workspace.

## Completion Reminder

- Before finishing work, follow the completion flow in `fleet-command/AGENTS.md`.
- Organize branches by work type using `feat/fix/chore/wip`.
- For one cross-repo task, keep the same slug across repos.
- Python work uses `uv` for dependency management and execution.
- Run verification first.
- Update fleet-command when workflow, architecture, or workspace rules changed.
- Re-render workspace assets when the manifest or generated context changed.
- Organize branches by repo + branch type + shared slug.
- Batch commit by repo.
- Batch push by repo.

## Workspace Notes

- Read generated repo context before scanning long status documents.
- Use fleet-command as the control plane for cross-repo workflow and conventions.
