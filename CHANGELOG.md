# Changelog

All notable changes to `@geslar/cli`. Format: [Keep a Changelog](https://keepachangelog.com/).

<!-- Source: the [1.0.0] entry is apps/cli/CHANGELOG.md of geslar-eu/geslar-platform at commit c04cac2c986d3b706a9ad3e742d6bd72a1098a39, translated to English. -->

## [Unreleased]

## [1.0.0] – RELEASE-DATE

The first stable version on the new Geslar platform. This list covers everything since the last published version, **0.3.2** (the old CLI, which depended on the old API). The trial version `1.0.0-rc.1` was published under the npm tag `next`.

### Added

**Sign-in and session**
- `geslar login` (browser, with PKCE and a DPoP-bound refresh token), `geslar login --device` (device code, for machines without a browser), `geslar status`, `geslar logout`.
- The key is kept in the OS keychain (Windows, macOS, Linux Secret Service) or, where there is none, in a sealed local store with a key file protected by a local password (`--key-storage file`).
- Croatian and English messages with automatic language detection; `--json` output is language-neutral.

**Unlock and read**
- `geslar unlock` / `geslar lock` (the password is asked for in the terminal only, without echo; `--ttl`, at most 8 h) and `--unlock` for a single command.
- `geslar read` and `geslar item list` with references `geslar://<vault>/<item>[/<field>]`: the keywords `personal`, `family`, `company` and `work`, `%2F` for a slash in a name, and an ambiguous name is an error that lists the candidates.
- `geslar run` and `geslar inject` (`-e KEY=geslar://…`, `-f file`): every reference is resolved before the command starts, values reach the command only through its environment, and the command's output is masked (`--no-masking` turns it off).

**Agents (AI tools)**
- Agent profiles: `geslar agent add | list | remove [--revoke]`, selected with `--agent <name>` or `GESLAR_AGENT`. The token (`gsm_…`) is read only from the hidden prompt or `--stdin`, never from an argument or the environment; the profile is stored sealed.
- An agent cannot print a value: `read` and `inject` are refused to it, and `run` works only for an allowed command, with masking, and with a reason of 1 to 200 characters that the approving person can see.
- Approvals: `geslar approval request | status | list`. The first read of a Vault's items needs a person: `run` stops with exit code 8, prints a link and a short code, and starts nothing. The person approves in the web app with their password and a fresh 2FA code, every time, and can approve less than was asked, never more. A request is valid for 10 minutes; the CLI does not wait for the decision and cannot approve anything.

**MCP**
- `geslar mcp serve`: a stdio MCP server over an agent profile with five tools: `geslar_list_vaults`, `geslar_list_items` (names and types only), `geslar_whoami`, `geslar_provision_env` (writes only `KEY=geslar://…` references, never values) and `geslar_run` (answers only with the exit code, how it ended, the duration and the number of output lines; a filtered tail of the output only with `--return-stdout-tail`).
- `geslar mcp init` writes the entry into a client's configuration (Claude Code, Claude Desktop, Cursor, or print-only JSON), without the token and with a backup; a configuration that cannot be parsed, or is a symbolic link, is left untouched.
- `geslar mcp allow-cmd | grants | revoke`: the commands an agent may start are approved by a person in the terminal with a typed confirmation (1 h by default, at most 8 h, `--max-uses`).

**CI mode**
- `GESLAR_SERVICE_TOKEN`: the token of a service account from the environment gives a session that lives in memory only, with no sign-in, unlock, terminal, keychain or disk. It works for `read`, `run`, `inject`, `item list` and `whoami` / `status`. It is closed against an agent token: the session is refused unless the server says exactly `service`. The child of `run` never receives the token or any other `GESLAR_*` credential, and the token is masked.

**Exit codes**
- 0 success · 1 usage or other error · 2 not signed in, or the identity is invalid, revoked or expired · 3 the plan does not include this · 4 a reference cannot be resolved · 5 access denied or outside the scope (the same as "does not exist") · 6 the Vault is locked · 7 the key store or the key-file password is unusable · 8 a person must approve first · 126 and 127 for `run` (the command cannot be executed / was not found).

**Platforms**
- Linux Secret Service: the real OS keychain also works across processes. An agent profile needs a persistent key store: `agent add` and `mcp serve` stop with exit code 7 before the token is used when there is no Secret Service. A lost profile key is named instead of the profile silently disappearing.
- Windows: stopping the whole process tree (`run`, `geslar_run`) does not rely on `taskkill` alone; it has three layers.
- The licences of all third-party packages are built into the package (`THIRD_PARTY_LICENSES.txt`).

### Changed
- **Output is cleaned of control characters.** Names of Vaults, items, organizations and identities, and error texts, are printed without terminal escape sequences (ESC/ANSI), control, bidi and invisible characters: terminal sequences are removed whole, other characters become the replacement character (U+FFFD), so a name cannot recolour, move or forge a line of output. The same applies to names in the MCP tools and to the output tail that `geslar_run` returns to an agent. Secret values are never changed.
- `--json` writes C1 control characters, bidi and other format characters inside strings as `\uXXXX`. The JSON is still valid and equal after parsing.
- The message about Linux keyutils now says what is true: the key does not survive a restart of the computer. The claim about a logout is gone.
- An invalid `-e` argument (a value that is not `KEY=geslar://…`) now adds the PowerShell advice: put `'--'` in quotes.
- A service token that the server refuses (unknown, revoked or expired; the server does not say which) has its own message, telling you to create a new service account and update the secret; the message for agents stays for agents.
- The `invalid_command` error now says in its `hint` field that `command` must be a plain program name and the arguments go in `args`.

### Removed
- **`geslar mcp serve --allow-shell` is removed.** `geslar_run` never starts a shell any more: `command` must be a plain program name (letters, digits, `. _ + -`) and every argument goes in `args`. A command line (`npm run build`, or anything with `;`, `&`, `|`, a backtick, `$(`, `<`, `>` or a newline) is refused as `invalid_command` before anything starts, with an `allow-cmd` permission or without. The child is always started with `shell: false`. The trial version `1.0.0-rc.1` still has this option; the stable version does not. Migration: instead of `{"command": "npm run build"}` send `{"command": "npm", "args": ["run", "build"]}`.
- **Signing in with a personal API key is turned off.** `geslar login --api-key` is hidden from the help and is refused with a pointer to `geslar login`; the server keeps those keys off as well, and sessions signed in that way are signed out with a message. The switch `GESLAR_ENABLE_PERSONAL_KEY_LOGIN=1` exists only in the CLI and does not change anything on the server.

### Security
- An agent's access is always approved by a person (password and a fresh 2FA code), never by the CLI. An agent receives a secret value only in the environment of a command the person has allowed, and with masking.
- The token of a service account is in no output and no error message.

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
