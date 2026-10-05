<!-- Source: the product text of this README is apps/cli/README.md of geslar-eu/geslar-platform at commit 53da9958ded20c5d899ca16a5863ad038c6086fa (pull request #368, branch docs/cli-readme-other-mcp-clients; after its merge the merge commit on main has the same text), with the trial-release wording removed for 1.0.0. Change it THERE and copy it; the sections "About this repository", "Support" and "License" belong to this repository only. -->

# Geslar CLI

[![npm](https://img.shields.io/npm/v/@geslar/cli)](https://www.npmjs.com/package/@geslar/cli)
[![node](https://img.shields.io/node/v/@geslar/cli)](https://nodejs.org)
[![license](https://img.shields.io/badge/license-proprietary-blue)](./LICENSE.md)

Command-line client for [Geslar](https://geslar.app): sign in, read a secret, run a command with secrets in its environment, and give AI tools and CI jobs narrowly scoped access to a Vault.

Requires a Geslar account on a Premium, Family, Business or Enterprise plan. Agents and CI service accounts (machine identities) need a plan that includes them; on a Family plan they belong to the plan's owner. The server never sees secret values. Each item you read is recorded in your account's audit log.

## About this repository

The Geslar CLI is **not open source**: its source code is in a private repository of Geslar d.o.o. This repository is the public home of the product and contains no code and no binaries. It exists for:

- **Issue tracking**: bug reports and feature requests
- **Security reporting**: see [SECURITY.md](./SECURITY.md)
- **Release notes**: [CHANGELOG.md](./CHANGELOG.md)
- **This README**, a copy of the one that ships in the npm package

The CLI is distributed through **npm only** (`npm install -g @geslar/cli`). There are no downloads, installers or release assets here; do not install it from anywhere else.

## Install

Requires Node.js 22 or newer.

```sh
npm install -g @geslar/cli
geslar --version
```

## Sign in

```sh
geslar login            # opens your browser (alias: geslar signin)
geslar login --device   # device code, for machines without a browser
geslar status           # current sign-in
geslar logout
```

## Unlock and read

Reading secrets needs your master password, asked for in the terminal only.

```sh
geslar unlock                      # stay unlocked for 30 minutes (--ttl 1h; at most 8h)
geslar read geslar://personal/GitHub/password
geslar read --unlock geslar://personal/GitHub/password   # ask for the password for this command only
geslar item list
geslar lock
```

A reference is `geslar://<vault>/<item>[/<field>]`; the field defaults to `password`. `personal` is your personal organization; `family`, `company` and `work` are keywords for organizations of that kind (`company` and `work` mean a Business or Enterprise organization), so a Vault can also be named directly. Use `%2F` for a literal `/` in a name. An ambiguous name is an error that lists the candidates.

## Run a command with secrets

```sh
geslar run -e DB_PASSWORD=geslar://personal/Database/password -- ./deploy.sh
geslar run -f secrets.env -- node app.js
```

Every reference is resolved before the command starts; if one fails, the command is not started. Values reach the command only through its environment. The command's output is masked by default (`--no-masking` turns this off); masking covers the literal value only, not encoded forms such as base64.

`geslar inject -f secrets.env` prints `KEY=value` lines for `source` or a dotenv loader. A value that contains characters outside `A-Za-z0-9_@%+=:,./-` is written in POSIX single quotes (`KEY='va lue'`, a `'` becomes `'\''`). Lines that are not references pass through unchanged. The output is **not** masked.

## PowerShell

`--` is required: everything after it is the command, and without it `geslar` would read your command's own options as its own (`geslar run -e KEY=geslar://… node --version` prints the version of `geslar`, not of `node`). PowerShell swallows a bare `--` before it reaches `geslar`; put it in quotes (`'--'`):

```powershell
geslar run -e DB_PASSWORD=geslar://personal/Database/password '--' node app.js
```

## Where the local key lives

The local key that seals your session is kept in the OS keychain where there is one: Windows Credential Manager, the macOS Keychain, or on Linux a Secret Service (GNOME Keyring, KWallet, KeePassXC). `--key-storage` (or `GESLAR_KEY_STORAGE`) chooses:

- `auto` (the default): the keychain when it works. If the keychain cannot be used at all, the CLI says so and keeps the key in a file protected by a local password instead. Once a key file exists, the file keeps being used.
- `keychain`: only the OS keychain; an error if it does not work.
- `file`: the key file, even where a keychain works.

```sh
geslar login --key-storage file     # or: GESLAR_KEY_STORAGE=file
```

**Linux without a Secret Service:** the keychain library does not fail there; it falls back to the kernel keyring (keyutils). The CLI notices this the first time it stores the key and warns you: the key lives in the kernel keyring of this login session and does **not** survive a restart of the computer, so you would sign in again; use `--key-storage file` to keep it. An agent profile cannot be created in that situation at all (see [Agents](#agents-ai-tools)).

The local password (file mode) is separate from your master password and is asked for in the terminal. Switching between keychain and file requires `geslar logout` first.

## Agents (AI tools)

An agent is a machine identity of kind "Agent (AI tool)" that you create in the web app under *Settings → Agents & automation → Agents & CI*: you choose the Vaults it may reach and a lifetime, and the web app shows its token (`gsm_…`) **once**. The agent never sees your password or any other Vault; it can list the names and types of items in the Vaults you give it, never their values.

```sh
geslar agent add ci-bot            # a person only, in a terminal: paste the token at the hidden prompt
geslar agent list                  # profiles on this machine and their state on the server
geslar agent remove ci-bot         # delete the local profile
geslar agent remove ci-bot --revoke   # also revoke the identity on the server (from your signed-in session)
```

- Enrol within **15 minutes** of creating the token; an agent that is not enrolled in time is revoked.
- The token is read only from the hidden prompt (or `--stdin` from a pipe), never from an argument or the environment. The profile is stored sealed on this machine; the token cannot be shown again. If the local key is lost (for example after a restart on a system without a persistent key store), `agent list` says so and you create a new agent.
- On Linux an agent profile needs a **persistent** key store — a Secret Service such as GNOME Keyring, KWallet or KeePassXC. Without one, `agent add` stops before the token is used (exit code 7). On Windows and macOS the OS keychain is used.
- Create a **separate Vault** for the agent and give the identity access to that Vault only.

An agent profile is selected with `--agent <name>` or `GESLAR_AGENT`:

```sh
geslar --agent ci-bot item list              # names and types only, never a secret field
geslar --agent ci-bot run --reason "run the test suite" -e DB_PASSWORD=geslar://Test-agent/Database/password -- npm test
```

Under an agent profile, and with a service account, a Vault is given by its **name**: the keywords `personal`, `osobno`, `family`, `company` and `work` are not resolved (a Vault that is itself named like that is found by its name; otherwise it is an error).

An agent profile deliberately **cannot print a value**: `read` and `inject` are refused. `run` works only for a command that you have allowed (see [MCP](#mcp-ai-clients)), always with masking, and with a reason of 1–200 characters that the person who approves can see.

### Approvals

The first time an agent needs to read items of a Vault, a person has to approve it. `run` stops with exit code **8**, prints a link and a short code, and starts nothing:

```sh
geslar --agent ci-bot approval request Test-agent --reason "run the test suite" --ttl 1h --uses 5
geslar --agent ci-bot approval status <requestId>
geslar --agent ci-bot approval list
```

- Open the link (or *Settings → Agents & automation → Approvals*), check that the short code is the same as in the terminal, read the reason, and approve with your password and a fresh 2FA code — every time. You can approve **less** than was asked (shorter time, fewer reads), never more.
- A request is valid for 10 minutes. The CLI does not wait for the decision and has no way to approve or deny anything; it never opens the browser by itself.
- A service account (see [CI mode](#ci-mode)) is not asked per read: its scope and expiry are the approval.

| Exit code | Meaning |
|---|---|
| 0 | success |
| 1 | usage or other error |
| 2 | not signed in, or the identity is invalid, revoked or expired |
| 3 | the plan does not include this |
| 4 | a `geslar://` reference cannot be resolved |
| 5 | access denied or outside the scope (indistinguishable from "does not exist"), or refused for an agent (`read`, `inject`, a command that is not allowed) |
| 6 | the Vault is locked |
| 7 | the local key store or key-file password is unusable |
| 8 | a person must approve first — wait, do not retry blindly |
| 126, 127 | `run`: the command cannot be executed / was not found |

## MCP (AI clients)

`geslar mcp serve` is a stdio [MCP](https://modelcontextprotocol.io) server over an agent profile, for clients such as Claude Code, Claude Desktop and Cursor. `geslar mcp init` writes the entry into the client's configuration for you:

```sh
geslar mcp init ci-bot                        # Claude Code (default; --scope user or local)
geslar mcp init ci-bot --client cursor        # claude-code, claude-desktop, cursor, or json (print only)
```

- The entry is only `{ command, args, env: { GESLAR_AGENT: "ci-bot" } }` — never the token. It starts `geslar mcp serve` if `geslar` is on your PATH, otherwise `npx -y @geslar/cli@<exact version> mcp serve`, where the version is exactly the one you have installed, never `latest`.
- A configuration file that cannot be parsed (for example one with comments), or is a symbolic link, is **not touched**: the entry is printed for you to paste. Otherwise a backup is written first, other servers are kept, and running it again changes nothing.
- For Claude Code, `mcp init` runs `claude mcp add-json` where it can and otherwise prints the exact command. **Claude Desktop on Windows** is not yet verified on a real client: the entry for it uses `cmd /c npx …` when `geslar` is not on the PATH — check that the server shows as running and report problems.
- `mcp serve` needs the OS keychain (no key file) and refuses to start while `GESLAR_SERVICE_TOKEN` is set.

The server offers five tools: `geslar_list_vaults`, `geslar_list_items` (names and types only), `geslar_whoami`, `geslar_provision_env` (writes `KEY=geslar://…` **reference** lines into an env file under the server's root, never values) and `geslar_run`. `geslar_run` starts a program with secrets in **its** environment and answers only with the exit code, how it ended, the duration and the number of output lines; the model gets no output unless you start the server with `--return-stdout-tail` (a filtered tail, best effort, **not** a security boundary). References are accepted in `env` and `env_file` only, never in the command, its arguments or the directory.

You decide which commands the agent may start, in a terminal, with a typed confirmation:

```sh
geslar mcp allow-cmd "npm test"          # lasts 1 hour by default (--ttl up to 8h; --max-uses n)
geslar mcp grants                        # active patterns
geslar mcp revoke --all                  # or: geslar mcp revoke <id or pattern>
```

A pattern lasts one hour unless you set `--ttl` (at most 8h), and `--max-uses` limits how many times it can be used. Keep patterns narrow (`npm test`, not `npm *`). **A pattern is not a security boundary:** a command that can run arbitrary code (a shell, `node`, `python`, `npx` and the like) can read the secrets in its own environment and send them anywhere. What protects your Vault is the Vault you gave the agent, its time and read limits, and the approval by a person with 2FA — not the pattern.

### Other MCP clients

Any MCP client that can start a local stdio server should work: `geslar mcp serve` speaks the MCP protocol over standard input and output only (no network port) and relies on nothing that is specific to one client. The configuration of a client names a command, its arguments and an environment; for Geslar they are:

- command `geslar`, arguments `mcp serve`, environment `GESLAR_AGENT=<profile name>` (instead of the environment variable you can add `--agent <profile name>` to the arguments);
- when `geslar` is not on the PATH: command `npx`, arguments `-y @geslar/cli@<exact version> mcp serve`, the same environment.

The profile must exist (`geslar agent add <profile name>`) and the OS keychain must be usable: the server never asks for a password. `mcp init` can print the entry in the shape that many clients read:

```sh
geslar mcp init ci-bot --client json
```

```json
{
  "mcpServers": {
    "geslar": {
      "command": "geslar",
      "args": [
        "mcp",
        "serve"
      ],
      "env": {
        "GESLAR_AGENT": "ci-bot"
      }
    }
  }
}
```

Where that goes and what the fields are called is in the documentation of your client; `mcp init` does not guess it. On Windows the entry that `mcp init` writes for Claude Desktop starts `npx` through `cmd` (`cmd /c npx -y …`), which is not yet verified on a real client (see above); a client that starts commands without a shell may need the same, and installing `geslar` globally avoids the question.

**Tested with:** the official MCP TypeScript SDK client (pinned in the CLI, run in its test suite against the real `mcp serve` over a stdio pipe and a fake Geslar server) and MCP Inspector 2.9.0 in CLI mode (`initialize`, `tools/list` with the five tools, and calls of `geslar_whoami` and `geslar_list_vaults`, against a fake Geslar server). Both check the protocol; neither is a client that people use. **`mcp init` configures:** Claude Code, Claude Desktop and Cursor, and prints the generic entry above for anything else. If your client works, or does not, tell us which one and which version in an [issue](https://github.com/geslar-eu/geslar-cli/issues).

## CI mode

A CI job uses a **service account** (created in the web app under *Settings → Agents & automation → Agents & CI*, kind "Service (CI)") and its token. Put the whole `gsm_…` token in the secret store of your CI and hand it to the job as the environment variable `GESLAR_SERVICE_TOKEN`:

```yaml
# GitHub Actions: the token goes in `secrets`, never in `vars`, and never as an input of a step
- run: npm install -g @geslar/cli@<exact version> --ignore-scripts
- run: geslar run -e DB_PASSWORD=geslar://CI/Database/password -- ./deploy.sh
  env:
    GESLAR_SERVICE_TOKEN: ${{ secrets.GESLAR_SERVICE_TOKEN }}
```

Under an agent profile, and with a service account, a Vault is given by its **name** (`CI` above is the name of a Vault): the keywords `personal`, `osobno`, `family`, `company` and `work` are not resolved (a Vault that is itself named like that is found by its name; otherwise it is an error).

What happens:

- `read`, `run`, `inject`, `item list` and `whoami` / `status` use the token when it is set and no `--agent` / `GESLAR_AGENT` is given. No `login`, no `unlock`, no terminal, no prompt: the session lives **in memory only**, and nothing is written to disk or to a keychain (it works in a container with neither). `--unlock` makes no sense with a token and is refused.
- **Only a service account's token works from the environment.** If the server says the token belongs to an agent, the CLI stops before it asks for any key; agents use `geslar agent add` on a machine of their own.
- The child process of `geslar run` never receives the token or any other `GESLAR_*` credential, and the token is masked in the child's output. The token is in no message or error either.
- The scope and the expiry of the account are the approval: there is no per-read approval for a service account.
- If the server refuses the token — it is unknown, revoked or expired; the server does not say which — `geslar` exits with code 2 and says exactly that: create a new service account in the browser and update the secret. (An agent profile gets a different message, because it is fixed differently: `geslar agent add` again.)

Advice: one service account per repository, read-only, with the shortest expiry that suits you and a rotation date; one `geslar run` per job rather than one per step (every start exchanges the token once; the server limits exchanges per hour and per address); a CI *environment* with protection for production secrets; never run it for pull requests from forks (GitHub does not give them your secrets, and `pull_request_target` with code from the pull request must not be used). Honest limit: while `geslar` runs, the token is in its own process environment (on Linux, `/proc/<pid>/environ` is readable by the same user); it is not passed on to anything it starts.

## What this does not protect against

- A person at a terminal can print any value they may read; masking covers the literal value only.
- Anything that runs as you can read what you can read. The CLI is not a sandbox against a malicious process on your own account.
- An agent that is allowed to run a program that can run code can send that program's secrets elsewhere (see [MCP](#mcp-ai-clients)); limit what you give it and approve with care.

## Support

- **Bugs and feature requests**: [open an issue](https://github.com/geslar-eu/geslar-cli/issues/new/choose)
- **Security vulnerabilities**: do **not** open a public issue; follow [SECURITY.md](./SECURITY.md)
- **Never paste a secret, a token (`gsm_…` included) or the contents of a Vault into an issue.** Redact before you post.

## License

Proprietary. Copyright © Geslar d.o.o. See [LICENSE.md](./LICENSE.md).

---

Geslar d.o.o. · OIB 43268748957 · Miroslava Krleže 13, 51000 Rijeka, Croatia
