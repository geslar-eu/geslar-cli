# Changelog

All notable changes to `@geslar/cli`. Format: [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

## [0.4.0] – RELEASE-DATE

A new CLI for the current Geslar platform. Versions 0.1.0 – 0.3.2 are deprecated and do not work with it.

### Added
- `geslar login` (browser, default), `geslar login --device` (device code) and `geslar login --api-key [--stdin]`; `geslar signin` is an alias. `geslar logout`, `geslar whoami [--json]`, `geslar status [--json]`.
- `geslar unlock [--ttl 30m|1h|…]` (default 30 minutes, at most 8 hours; also locks after 15 minutes without use) and `geslar lock`. The master password is asked for in the terminal only.
- `geslar read <geslar://ref>` and `geslar item list [vault] [--json]`. `--unlock` asks for the master password for that one command and stores nothing.
- `geslar run [-e KEY=geslar://…]… [-f secrets-file] [--no-masking] [--unlock] -- <command>` and `geslar inject -f secrets-file [--unlock]`. Every reference is resolved before anything starts or is printed. `run` masks the literal secret value in the command's output; `inject` output is not masked.
- `geslar://<vault>/<item>[/<field>]` references with `personal`, `family`, `company`/`work`, organization and Vault names, an `id:` item qualifier, and fields `password`, `username`, `url`, `notes`, `totp` and custom labels. A name collision is always an error that lists the candidates.
- Local state: the session is kept in an encrypted file; its key lives in the OS keychain, or — where there is none — in a key file protected by a local password. `--key-storage <auto|keychain|file>` / `GESLAR_KEY_STORAGE`.
- Croatian and English output (`--lang hr|en`, `GESLAR_LANG`, or the system locale).
- Exit codes: 0 success, 1 general, 2 authentication, 3 plan or policy, 4 unresolvable reference, 5 access denied, 6 Vault locked, 7 local key-file password, 126/127 command not executable/not found, 128+N killed by signal N.
- Requires Node.js 22 or newer, and a Premium, Family, Business or Enterprise plan. Distributed through npm only.

### Known limitations
- Online-only; no offline copy of the Vault.
- Masking in `run` covers the literal value only, not encoded forms (for example base64).
- Every `read`, `run` and `inject` fetch of an item is recorded in the audit log (`item.accessed`); the server does not see the value or the field.
- `read`, `run` and `inject` need `geslar unlock` or `--unlock`, which needs a terminal. A mode for CI pipelines is planned, not available.

## [0.3.2] – 2026-07-25

0.3.1 shipped a stale build (0.2.0 dist) despite package.json/CLI_VERSION correctly saying 0.3.1 — `dist/` is gitignored, per-checkout build output, and nothing forced a fresh rebuild immediately before the actual `npm publish` ran. 0.3.2 is a fix-forward: the 0.3.1 fixes are correctly built this time, plus a guard so this class of mistake can't silently recur.

### Fixed
- Republished with a correctly rebuilt `dist/index.js` — all three 0.3.1 fixes (unlock in the README quickstart, honest `mcp init` pin wording, `--scope user` default) are now actually present in the shipped package, not just in source.

### Added
- `prepublishOnly` npm lifecycle script: rebuilds `dist/` from source and then fails loudly (before `npm publish` can pack anything) if the freshly-built `dist/index.js --version` doesn't exactly match `package.json`'s version. Cross-platform (plain Node, no shell script) — verified to actually fire via `npm publish --dry-run` and to actually fail when the two versions are made to mismatch on purpose.

## [0.3.1] – 2026-07-25

Three fixes found by an actual post-publish MCP smoke test of 0.3.0 (real signin, real agent, real Claude Code, real published package) — none are new features.

### Fixed
- `README.md`'s MCP quickstart snippet was missing `geslar unlock` — `mcp init`'s preflight (unlike `agent create`, which prompts for the master password inline in a TTY) has no such fallback and fails closed with "Vault is locked" if no grant already exists. Snippet now reads `unlock` → `agent create` → `mcp init` (unlock first so `agent create` doesn't prompt a second time).
- `README.md` claimed `mcp init` config is always "version-pinned, never `@latest`" — true only for the npx fallback. When a global `geslar` binary is present on PATH, `mcp init` correctly prefers it (`command: "geslar"`), which tracks whatever's currently installed rather than pinning a version. Reworded to describe actual behavior instead of an absolute pin guarantee.
- `geslar mcp init --client claude-code` called `claude mcp add-json` without `--scope`, which defaults to `local` (the one project directory `mcp init` happened to be run from) on Claude Code's own CLI — not the intended machine-wide "available in every project" registration. Now passes `--scope user` explicitly; a new `--scope <user|local>` flag on `mcp init` allows overriding it.

## [0.3.0] – 2026-07-24

### Added
- `geslar mcp serve [--allow-shell] [--return-stdout-tail] [--root <dir>] [--pass-env <vars>]` — a stdio MCP server (`@modelcontextprotocol/sdk`) exposing 6 tools to MCP clients (Claude Code, Claude Desktop, Cursor, …) under an agent profile: `geslar_list_vaults`, `geslar_list_items`, `geslar_whoami` (read-only, no pre-grant needed), `geslar_provision_env`, `geslar_run` (write/execute, pre-grant required). No tool ever returns a secret value; `geslar_run`'s stdout is withheld by default (`--return-stdout-tail` opts into a scrubbed tail, which lowers the guarantee — see limitations). Agent context only — refuses to start under a human session.
- `geslar mcp allow <vault> [--ttl] [--max-uses]` / `geslar mcp allow-cmd <pattern> [--ttl] [--max-uses]` / `geslar mcp grants` / `geslar mcp revoke <id>|--all` — the pre-grant subsystem: write/execute MCP tools require an out-of-band approval (outside the agent's own channel), OS-wrapped and signed locally, TTL-bound (default 1h, max 8h).
- `geslar mcp init <agent> [--client claude-code|claude-desktop|cursor|json]` — onboarding: detects/prompts for an installed MCP client, generates a version-pinned config entry (never `@latest`), merges it atomically into the client's config (backup-before-write), and never writes a token — only `GESLAR_AGENT=<name>`.
- Cross-process-safe unlock grant: the `geslar unlock` grant is now also written to an always-available encrypted file store, alongside the primary OS keychain — some OS keychains (notably macOS) scope read access by calling-app identity, which could otherwise block `geslar mcp serve` (spawned by a different parent process, e.g. an MCP client) from reading a grant written by a separate `geslar unlock` invocation.
- `geslar_run` cancellation now kills the whole process tree, not just the direct child — both on `timeout_s` and on the MCP client cancelling its own request. Fixes two real gaps found only once this ran on real macOS/Linux CI (this repo's dev machine is Windows): the POSIX process-group signal alone doesn't reach a descendant that escapes the group via its own `setsid()`, and BSD `ps -e` (macOS) doesn't mean "all processes" the way GNU/Linux's does — both fixed with an OS-portable PID-lineage walk (`ps -Ao pid,ppid`).
- macOS now has real CI coverage for `@geslar/cli` (`cli-macos` job) — the two fixes above were both caught by that job's first real run, not assumed correct from Linux/Windows testing alone.

### Known limitations (new)
- `geslar_run`'s exfiltration boundary: an agent allowed to run a command CAN exfiltrate a secret from its environment by any channel the command itself has (network, encoding into exit code/timing) — this is categorical to every injection-execution tool (`op run` included), not something the pre-grant model or output scrubber prevents. What IS guaranteed: no MCP tool response ever contains a secret value. Real defenses are least-privilege agent scope, pre-grant, audit, and revocation.
- `--return-stdout-tail`'s scrubber is best-effort (low-entropy/short values aren't recognized) — a cosmetic last line of defense for accidental leaks, not a security boundary against a deliberately-exfiltrating command.
- Under `mcp serve` (same as the existing agent-context limitation), `family`/`company`/`work` semantic vault tokens don't resolve — direct vault name and `personal` are unaffected.
- `geslar mcp serve` never prompts for the master password under any circumstance — a locked vault always returns an actionable error (`run \`geslar unlock\` first`), never a silent hang or prompt.

## [0.2.0] – 2026-07-22

### Added
- `geslar agent create <name> --vault <name>... [--allow-read] [--expires 90d|never] [--show-token]` — create a scoped, machine-usable agent profile (TTY required, fresh TOTP; recovery codes rejected). Default capabilities are `run`+`inject`; `--allow-read` additionally grants `read` after an explicit typed confirmation, since it lets `geslar read` print secret values under that profile. The token is saved locally (OS keychain, `agent-token` namespace, distinct from the signed-in-user and unlock-grant namespaces) and shown once, unless `--show-token` prints it to stdout as well.
- `geslar agent list` / `geslar agent revoke <name>` — list your agent profiles (status, capabilities, expiry) and revoke one, which also clears its local token.
- `--agent <name>` flag / `GESLAR_AGENT` env var — run any command (`list`, `read`, `run`, `inject`) as an agent profile instead of your own account. `read` is blocked with a clear message (exit 5) unless the active profile has the `read` capability — checked live against the server on every invocation, never cached locally. Agent context cannot create or revoke agent profiles (human session required for both).

### Known limitations (new)
- Under agent context, the `family`/`company`/`work` semantic vault-address tokens don't resolve (`GET /v1/orgs` isn't on the agent allowlist, so org type is unavailable) — addressing a granted vault directly by its own name is unaffected.
- `geslar agent list` shows vault names from a local cache saved at create time (the server intentionally reports only a count, never names) — a profile created on a different machine shows its count/status/expiry but not vault names.

## [0.1.0] – 2026-07-18

### Added
- `geslar signin` / `signout` / `whoami` — RFC 8628 device-flow authentication, OS-backed token storage (Windows DPAPI, macOS Keychain, Linux secret-tool/libsecret, with an encrypted-file fallback).
- `geslar unlock` / `lock` — session grant for headless use, so `run`/`inject` don't need a master-password prompt on every invocation.
- `geslar list [vault]` / `read <ref> [-o file]` — resolve a `geslar://` reference and print (or write) its value.
- `geslar run --env-file <f> [-e K=ref]... -- <cmd> [args...]` / `inject -i <tpl> -o <out>` — resolve `geslar://` references into a child process's environment or a template file. Fail-closed: every reference is resolved before anything spawns or is written.
- `geslar://<vault>/<item>[/<field>]` reference resolution, entirely local after decrypt (the server never learns which fields are requested). The personal vault accepts the canonical `personal` token or the legacy `Osobno` alias; `family` and `company`/`work` address your family/business org's default vault (deterministic only with exactly one qualifying org); any org's default vault is also addressable by the org's own name. All case-insensitive; a token colliding with a real vault/org name is always an ambiguity error, never a silent guess. `list [vault]` accepts the same address space.
- Proprietary EULA (`LICENSE.md`, ships in the npm package).
- Output scrubbing: every diagnostic line the CLI itself prints is scrubbed of any resolved secret value; the one deliberate value-emission channel (`read`'s stdout, or `-o file`) is unaffected.
- Tier gate: `read`/`run`/`inject` require a Premium, Family, Business, or Enterprise plan (`agentic_cli`); enforced server-side (fail-closed) as well as client-side for a fast, friendly failure.
- Standalone binaries for Windows x64 and Linux x64 (Node Single Executable Applications) — no separate Node.js install required. macOS is npm-only for now (Apple notarization is pending).

### Known limitations
See the "Known limitations" section of the docs for the full list (offline use, CI-without-human-approval, child-process env visibility, Alpine/busybox caveats, etc.).
