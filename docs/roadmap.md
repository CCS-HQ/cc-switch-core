# Roadmap and next plan

This roadmap orders the remaining shared work by the real consumer call
boundaries already recorded in [consumer migration acceptance](consumer-migration.md).
It does not authorize full-product inspection or migration, and it does not
replace the per-slice acceptance rules in the
[full-product readiness plan](full-product-readiness.md). Current progress is
tracked in [status](status.md).

Every implemented slice keeps the existing gates: one real caller at a time,
removal of the replaced implementation, compatibility fixtures against the
previous consumer behavior, and two independent blind reviews. No `is_lite`,
`cli_mode`, or product flag is introduced in any phase.

## Phases

### Phase 0: Close the MCP execution chain

Goal: mark migration stage 1 (MCP) complete.

- Migrate CLI `claude_mcp::set_mcp_servers_map` to Core's `PreserveFields`
  encoding, keeping the host's single `server` unwrap and UI-field filtering.
- Replace the CLI import's whole-product snapshot save with
  `McpTransactionGuard::read_servers` merging against fresh catalog rows.
- Route the non-Gemini `McpService::toggle_app` native writes through
  `execute_mcp_write_with_content_limit`; hosts bind Claude's MCP file from the
  App contract, not `LogicalTarget::ClaudeSettings`.
- Adopt `commit_preserving_on_error` in Lite's `upsert_with_live`,
  `toggle_with_live` and `delete_with_live`, then in the CLI standalone MCP
  workflow with its own observed native inputs.
- Run CLI/Lite MCP contention tests in both directions before claiming
  cross-consumer acceptance.

Exit: every MCP row in [status](status.md) reads `done` for Core and Consumer.

### Phase 1: Gemini execution and the shared live lock

Goal: finish Gemini provider migration step 3 and retire the old
unconditional native restore.

- Make `gemini_mcp::set_mcp_servers_map` observable and recoverable: retain the
  first observation through publication, keep every follow-up receipt, and use
  `OperationReceipt::try_coalesce_last_write` only for one shared recovery boundary.
- Connect the ordinary Gemini switch in `run_staged_transaction` /
  `apply_prepared_post_commit_action` to Core execution and the provider + MCP
  composite guard from `McpTransactionGuard::from_provider_transaction`. Reverse
  receipts in publication order, then roll back the database; keep auth-flag and
  Skill-tail compensation host-owned. Pass the three red CLI gates and delete the
  per-entry MCP tail.
- Adopt `fs::SharedLiveConfigLock` in Lite `LiveConfig::lock_file` and in CLI
  `switch_gemini_coordinated`, acquired after the database write transaction and
  before the first native observation.
- Verify database/file lock order and resource identities with CLI/Lite
  contention tests, including incomplete recovery and a provider-owned file
  written more than once. Preserve host size limits through
  `execute_dependency_ordered_plan_with_content_limit`.
- Keep force-write a separate caller until its no-old-env-read contract is decided.

Exit: Gemini rows in [status](status.md) read `done`; the CLI's old native
backup restore is removed.

### Phase 2: Remaining provider Apps

Goal: complete migration stage 2 (Providers).

- Migrate native import and projection behind the registered adapter one App at a
  time, in this order: Claude, OpenCode, Hermes, Pi, OpenClaw, GrokBuild, Claude
  Desktop. Each change covers current modes, authentication, unknown fields,
  multi-file failure behavior and proxy-related host policy.
- Establish verified model-fetch defaults for Claude Desktop and GrokBuild, or
  record why none exists.
- Audit remaining App-specific matches in CLI/Lite against the registry and
  capability declarations; record intentional product boundaries separately.

Exit: every App's provider row reads `done`; no duplicate provider codec remains
in a consumer.

### Phase 3: Skill deployment composition

Goal: complete migration stage 3 (Skills) through the four gates in
[Skill deployment composition](skill-deployment-composition.md).

1. Capture copy/link/Auto, repeated toggle, existing entries, missing sources,
   native controls and Pi conflicts as bounded tests around the current CLI and
   Lite services. Tests only.
2. Implement one shared deployment path for Claude with all three CLI modes and
   Lite operations on the same installed Skill, composed with Store's scoped
   selection write. Resolve representation compatibility first if a new
   ownership format is required.
3. Extend to the remaining Skill Apps one at a time, including Gemini/Hermes
   native controls and unified discovery.
4. Remove duplicate writers: set-apps, install/reuse, import, uninstall, sync
   and storage migration.

Exit: the three red CLI Skill gates pass; Lite understands managed
representations; no product-local deployment engine remains.

### Phase 4: Architecture acceptance and 0.2.0

Goal: migration stage 4 and the first tagged release.

- Trace real CLI and Lite calls against the registry, adapters, capability
  declarations and per-App conformance tests; record intentional product
  boundaries versus unfinished shared behavior.
- Add a CHANGELOG, tag `v0.1.0` at the current baseline, and publish `0.2.0`
  once phases 0-1 land. Define the MSRV policy and a rusqlite upgrade path that
  all consumers adopt together, because the `bundled` feature requires one
  SQLite version per process.
- Split the files above 3,000 lines by App or capability without changing the
  public API. Box the `Err` payload of `commit_preserving_on_error` so a Clippy
  upgrade does not break CI.
- Keep [status](status.md) current and reduce the running log in
  [consumer migration acceptance](consumer-migration.md) to per-capability sections.

### Phase 5: Full desktop product

Requires separate authorization before any inspection. When granted: compare
read/import results first, migrate one App or capability on an isolated branch
with sanitized baseline fixtures, run one production writer per resource, and
open PRs only with explicit approval.

## Next plan

The next iteration is Phase 0 plus low-risk hygiene. Items 1-3 change only the
CLI migration branch and need no new Core API. Item 4 updates the Lite pin.
Item 6 is Core-only and can start immediately.

| # | Change | Acceptance |
| --- | --- | --- |
| 1 | CLI Claude MCP toggle uses `PreserveFields` | Byte comparison against the previous setter, nested `server` key retained, UI fields still filtered, snapshot restore, double review |
| 2 | CLI import merges through `read_servers` | Same-ID policy and diagnostics unchanged; stale whole-product snapshot removed; ignored read error poisons commit |
| 3 | CLI `toggle_app` executes through `execute_mcp_write_with_content_limit` | Registry-driven publication and recovery for every declared MCP target; oversized and malformed expectations rejected before resource access |
| 4 | Lite `*_with_live` adopt `commit_preserving_on_error` | Native recovery under the live-file lock, explicit database rollback, both recovery errors reported, successful retry |
| 5 | CLI/Lite MCP contention tests | Both acquisition directions, unchanged data on failure, release after success, native failure and commit failure |
| 6 | Hygiene | `docs/status.md` updated per merged slice; `v0.1.0` tag and CHANGELOG; boxed `Err` for `commit_preserving_on_error`; CI stays green on 1.85.0 |

Each item records the consumer call site replaced, the intended full-product
boundary, API/default/wire/schema impact, tested Core/Store revisions and the
rollback path, per the [per-change record](full-product-readiness.md#per-change-record-and-completion-language).
