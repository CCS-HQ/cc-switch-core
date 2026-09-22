# Shared migration status

Snapshot as of 2026-09-19 at Core/Store `199d9f7`. This matrix summarizes the
running record in [consumer migration acceptance](consumer-migration.md); that
document remains the authoritative per-slice evidence. Update this file in the
same change that moves a row.

The three states follow the [full-product readiness plan](full-product-readiness.md):

- **Core:** the shared API exists, with contract tests and double blind review.
- **Consumer:** a real CLI and/or Lite caller uses it and the replaced
  implementation is removed. Pins: CLI `6e28a23`, Lite `7b7cfae`.
- **Full product:** not authorized; no row can be marked verified there yet.

Legend: `done`, `partial`, `pending`, `design` (documented, no API), `-` (not
applicable).

## MCP

| Slice | Core | Consumer | Next real caller |
| --- | --- | --- | --- |
| OpenCode/Hermes entry conversion | done | done (CLI) | - |
| Codex MCP entry projection and tolerant transport decoding | done | done (CLI) | - |
| Codex native MCP document operations | done | done (CLI) | - |
| Gemini entry codec policies | done | done (CLI `read_mcp_servers_map` / `set_mcp_servers_map`) | - |
| Minimum connection validation | done | done (CLI `validate_server_spec`) | - |
| Native entry snapshot capture/restore in host documents | done | pending | CLI standalone Claude toggle |
| Claude `PreserveFields` encode policy | done | pending | CLI `claude_mcp::set_mcp_servers_map` |
| `McpTransactionGuard::read_servers` | done | pending | CLI import merge against fresh rows |
| `execute_mcp_write_with_content_limit` | done | pending | CLI non-Gemini `McpService::toggle_app` |
| `commit_preserving_on_error` | done | pending | Lite `upsert/toggle/delete_with_live`, then CLI standalone MCP workflow |
| Provider + MCP composite transaction (`from_provider_transaction`) | done | pending (three red CLI gates) | CLI `switch_gemini_coordinated` step 3 |
| CLI/Lite MCP contention tests | - | pending | Both consumers |

## Providers

| Slice | Core | Consumer | Next real caller |
| --- | --- | --- | --- |
| Codex auth observation, bearer routing, credential codec | done | done (CLI) | - |
| Codex two-file auth/config writes via Core execution | done | done (CLI) | - |
| Codex native import through registered adapter | done | done (CLI) | - |
| Claude model-key migration and metadata cleanup | done | done (CLI) | - |
| Gemini read/import through registered adapter | done | done (CLI) | - |
| Gemini env field selection and settings overlay | done | done (CLI) | - |
| Gemini literal env rendering | done | done (CLI) | - |
| Gemini execution baseline (step 3.1) | done (tests/docs only) | - | - |
| Gemini MCP publication observable at `set_mcp_servers_map` (step 3.2a) | done (executor, receipts, coalesce) | pending | CLI `gemini_mcp::set_mcp_servers_map` |
| Gemini switch through `run_staged_transaction` (step 3.2b) | done (executor) | pending | CLI `apply_prepared_post_commit_action` |
| `SharedLiveConfigLock` | done | pending | Lite `LiveConfig::lock_file`, CLI `switch_gemini_coordinated` |
| Cross-consumer lock order and resource identity (step 3.3) | - | pending | Both consumers |
| Native import/projection for Claude, OpenCode, Hermes, Pi, OpenClaw, GrokBuild, Claude Desktop | done (adapters, conformance) | pending | One App per change |

## Model fetch

| Slice | Core | Consumer | Notes |
| --- | --- | --- | --- |
| Declarative request/response rules (`ModelFetchSpec`) | done | done (CLI/TUI) | Lite PR #41 |
| Registry defaults with host overrides | done | done (CLI) | Lite PR #42; Claude Desktop and GrokBuild have no default yet |
| Worker channel and response-only decoder | done | done (CLI) | Lite PR #43; stage accepted |

## Skills

| Slice | Core | Consumer | Notes |
| --- | --- | --- | --- |
| Catalog snapshots, reference plans, complete switch planner | done | done (Lite), partial (CLI catalog only) | Existing Core API |
| Store guarded catalog writes | done | done (CLI catalog) | - |
| Deployment composition gates 1-4 | design | pending (three red CLI gates) | [Skill deployment composition](skill-deployment-composition.md) |

## Stage summary

| Stage | State |
| --- | --- |
| 1. MCP | Codecs and document operations verified; execution and composite adoption pending |
| 2. Providers | Codex complete; Gemini read/prepare complete, execution pending; other Apps pending |
| 3. Skills | Design only |
| 4. Architecture acceptance | Not started |
| Full product | Not authorized |

## Engineering health

| Item | Observation |
| --- | --- |
| Tests | 489 pass locally on Rust 1.97.1; CI pins 1.85.0 on three platforms |
| Lints | fmt and `cargo doc` clean; newer Clippy reports `result_large_err` on `commit_preserving_on_error` |
| Releases | No tags, no CHANGELOG; consumers pin commit SHAs |
| Dependencies | rusqlite 0.31 (`bundled`); `json5` for value decoding and `json-five` for format-preserving edits |
| Largest files | `src/mcp.rs`, `crates/cc-switch-store/src/{lib,skill}.rs`, `src/projection.rs` each exceed 3,000 lines |
