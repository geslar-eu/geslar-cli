# Geslar CLI

[![npm](https://img.shields.io/npm/v/@geslar/cli)](https://www.npmjs.com/package/@geslar/cli)
[![node](https://img.shields.io/node/v/@geslar/cli)](https://nodejs.org)
[![license](https://img.shields.io/badge/license-proprietary-blue)](./LICENSE.md)
[![docs](https://img.shields.io/badge/docs-docs.geslar.app-black)](https://docs.geslar.app/cli/reference)

Command-line and [Model Context Protocol](https://modelcontextprotocol.io) access to the secrets in your **Geslar Škrinjar** vaults — for scripts, CI, and AI agents.

**No tool ever returns a secret value to the agent.** References are resolved locally, after decryption on your own machine, and injected straight into a child process's environment. The value never reaches the model as a tool response.

```sh
npm install -g @geslar/cli
geslar signin
geslar run -e DB_PASSWORD=geslar://Work/Database/password -- ./start-server.sh
```

---

## About this repository

The Geslar CLI is **not open source**. Its source code lives in a private repository at Geslar d.o.o.

This repository is the public home of the product. It exists for:

- **Issue tracking** — bug reports and feature requests
- **Security reporting** — see [SECURITY.md](./SECURITY.md)
- **Release notes** — [CHANGELOG.md](./CHANGELOG.md)
- **Distribution** — client install configurations and, in a future release, a Claude Code plugin marketplace

Full product documentation lives at **[docs.geslar.app/cli/reference](https://docs.geslar.app/cli/reference)**.

## Requirements

- **Node.js 20 or newer** (standalone Windows and Linux binaries are also available — see the docs)
- A **Geslar** account on a Premium, Family, Business, or Enterprise plan

## Install

```sh
npm install -g @geslar/cli
```

## MCP server

`geslar mcp serve` runs a stdio MCP server scoped to an **agent profile** — a separate, revocable machine credential with its own vault scope, capabilities, and expiry. It is not your user session, and it cannot create or revoke agent profiles.

```sh
geslar unlock --ttl 30m
geslar agent create my-agent --vault Work
geslar mcp init my-agent --client claude-code
```

`geslar mcp init` detects your MCP client, writes the configuration atomically (backing up the existing file first), and **never writes a token** into it — only `GESLAR_AGENT=<name>`. If a global `geslar` binary is on your `PATH` it points the config there; otherwise it pins an exact version via `npx -y @geslar/cli@<version>`, never an unpinned `@latest`.

### Supported clients

| Client | `--client` value | Configuration written |
|---|---|---|
| Claude Code | `claude-code` | via `claude mcp add-json --scope user` |
| Claude Desktop | `claude-desktop` | the app's own `claude_desktop_config.json` |
| Cursor | `cursor` | `~/.cursor/mcp.json` (global scope) |
| Any other MCP client | `json` | prints a snippet to paste yourself |

### Tools

Six tools, each carrying MCP annotations so the client can show you what it does before you approve it.

| Tool | Returns | Pre-grant required |
|---|---|---|
| `geslar_list_vaults` | Vaults this agent profile may access, with item counts. No values. | no (read-only) |
| `geslar_list_items` | Item and field **names** in one vault. No values. | no (read-only) |
| `geslar_whoami` | The profile's own name, scope, capabilities, expiry, lock state. No values, no owner PII. | no (read-only) |
| `geslar_provision_env` | Writes `geslar://` **references** — not resolved values — into a `.env`-style file under the server root. | yes — `geslar mcp allow` |
| `geslar_run` | Runs a command with references resolved into its environment. Reports exit status and line counts; returns no process output by default. | yes — `geslar mcp allow-cmd` |
| `geslar_audit_recent` | The profile's own recent audit events. | no (read-only) |

### The pre-grant model

Anything that writes or executes requires an approval issued **outside the agent's own channel** — you run `geslar mcp allow` or `geslar mcp allow-cmd` in your own terminal. Grants are TTL-bound (1 hour by default, 8 hours maximum), optionally use-limited, signed locally, and OS-wrapped. `geslar mcp grants` lists them; `geslar mcp revoke` clears them.

A locked vault always fails closed with an actionable error. `geslar mcp serve` never prompts for your master password under any circumstance, and refuses to start under a human session.

## What this does and does not protect against

We would rather you read this before installing than after.

**Guaranteed:** no MCP tool response ever contains a secret value. Reference resolution happens on your machine after local decryption; the server never learns which vault, item, or field a reference asks for.

**Not guaranteed:** an agent that you have allowed to run a command can, in principle, exfiltrate a secret from that command's environment through any channel the command itself has — the network, or encoding into timing or exit status. This is categorical to every secret-injection tool, and the pre-grant model does not change it. The real defenses are least-privilege agent scope, the pre-grant, the audit trail, and revocation.

`--return-stdout-tail` opts into a scrubbed tail of a command's output. The scrubber is best-effort — it does not recognize short or low-entropy values — and is a guard against accidental leakage, not a boundary against a command that is deliberately exfiltrating.

Further known limitations are listed in the [reference documentation](https://docs.geslar.app/cli/reference) and in each release's entry in [CHANGELOG.md](./CHANGELOG.md).

## Documentation

- [CLI reference](https://docs.geslar.app/cli/reference) — install, quickstart, `geslar://` reference syntax, agent profiles, MCP, known limitations
- [geslar.app](https://geslar.app) — the Geslar Škrinjar password manager

## Support

- **Bugs and feature requests** — [open an issue](https://github.com/geslar-eu/geslar-cli/issues)
- **Security vulnerabilities** — do **not** open a public issue; follow [SECURITY.md](./SECURITY.md)
- **Never paste a secret, token, or vault contents into an issue.** Redact before you post.

## License

Proprietary. Copyright © Geslar d.o.o. See [LICENSE.md](./LICENSE.md).

---

Geslar d.o.o. · OIB 43268748957 · Miroslava Krleže 13, 51000 Rijeka, Croatia
