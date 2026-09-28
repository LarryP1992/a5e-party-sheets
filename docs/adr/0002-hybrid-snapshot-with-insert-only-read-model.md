# 0002 — Hybrid JSONB snapshot with an insert-only normalized read model

**Status:** accepted · **Date:** 2026-09-27

## Context

Every upload is an enriched Foundry VTT export (~1 MB JSON, `{ source, derived, meta }`). Foundry is the source of truth for every number in Phase 1. ETL bugs are certain, and the Sheet must render from typed, queryable columns. A Phase 2 character-management site with two-way Foundry sync is the known next effort, not part of this build.

## Decision

- Store each upload **verbatim** as `snapshot.payload jsonb`, deduplicated per character by a SHA-256 content hash, with `meta` fields promoted to typed columns. Snapshots are immutable and kept forever.
- Build **normalized rows per snapshot, insert-only**, keyed by `snapshot_id`, as a read model. Rows are never updated; a rebuild deletes and reinserts one snapshot's rows in a transaction, recording `etl_version` on the snapshot.
- `character.current_snapshot_id` points at the latest snapshot; history is a query.
- Items are a supertype table with thin 1:1 subtype tables (`spell`, `object`, `feature`, `maneuver`, `class_level`); the full prepared item block stays as JSONB.
- Derived values take the plain column name; source values sit beside them wherever the source is a rule input. Foundry identifiers are retained on every row, and a `character_source` table separates our Character from its Foundry identities.

## Alternatives considered

- Normalized tables only: an ETL bug would lose data with no way back.
- JSONB only: every Sheet query becomes JSONB operators plus GIN indexes, and nothing teaches SQL.
- Mutable rows per character with a version column: quietly kills insert-only, needs update and conflict logic, and stops the read model being rebuildable.
- Designing Phase 2's write model now: speculation frozen into migrations.

## Consequences

- Storage grows with every upload (TOAST keeps it small); dedupe stops accidental duplicates.
- "What changed since last session" is a join between two snapshots' rows.
- **Obligation to Phase 2:** this schema loses nothing (verbatim payload, identifiers, source beside derived, provenance blocks whole) but is shaped for reading only. Phase 2 adds a separate mutable write model; it does not mutate these tables.
- Bigint identity keys are enumerable; every URL is membership-gated, so this leaks nothing. UUIDs become a fresh ADR if sync ever needs client-generated ids.
