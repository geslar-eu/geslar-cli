# Geslar CLI

[![npm](https://img.shields.io/npm/v/@geslar/cli)](https://www.npmjs.com/package/@geslar/cli)
[![node](https://img.shields.io/node/v/@geslar/cli)](https://nodejs.org)
[![license](https://img.shields.io/badge/license-proprietary-blue)](./LICENSE.md)
[![docs](https://img.shields.io/badge/docs-docs.geslar.app-black)](https://docs.geslar.app/cli/reference)

Command-line client for [Geslar](https://geslar.app): sign in, read a secret, or run a command with secrets in its environment — for scripts and for the terminal.

Decryption happens on your own machine. Your master password and the decrypted values never go to the Geslar server.

```sh
npm install -g @geslar/cli
geslar login
geslar unlock
geslar run -e DB_PASSWORD=geslar://Work/Database/password -- ./start-server.sh
```

---

## About this repository

The Geslar CLI is **not open source**. Its source code lives in a private repository at Geslar d.o.o.

This repository is the public home of the product. It exists for:

- **Issue tracking** — bug reports and feature requests
- **Security reporting** — see [SECURITY.md](./SECURITY.md)
- **Release notes** — [CHANGELOG.md](./CHANGELOG.md)

Full product documentation lives at **[docs.geslar.app/cli/reference](https://docs.geslar.app/cli/reference)**.

## Requirements

- **Node.js 22 or newer**
- A **Geslar** account on a Premium, Family, Business, or Enterprise plan

The CLI is distributed through npm only.

## Install

```sh
npm install -g @geslar/cli
geslar --version
```

## Sign in

```sh
geslar login            # opens your browser (alias: geslar signin)
geslar login --device   # device code, for machines without a browser
geslar login --api-key  # personal API key, from a hidden prompt (or one line of stdin with --stdin)
geslar status           # who you are signed in as, and whether the Vault is unlocked
geslar logout
```

An API key signs you in; it does not unlock your Vault. Device-code sign-in is off until you turn it on in your Geslar security settings; personal API keys can be disabled there.

## Unlock and read

Reading secrets needs your master password, asked for in the terminal only — never from an environment variable, an argument, or a file.

```sh
geslar unlock                      # stay unlocked for 30 minutes (--ttl 1h, at most 8h)
geslar read geslar://Work/GitHub/password
geslar read --unlock geslar://Work/GitHub/password   # ask for the password for this command only
geslar item list
geslar lock
```

An unlocked Vault also locks itself after 15 minutes without use. `--unlock` keeps the key in the memory of that one command and stores nothing.

A reference is `geslar://<vault>/<item>[/<field>]`. The field defaults to `password`; others are `username`, `url`, `notes`, `totp` (the current code) and custom field labels. `<vault>` is `personal`, `family`, `company` or `work`, an organization name, or a Vault name. Use `%2F` for a literal `/` in a name. An ambiguous name is an error that lists the candidates; `id:<item id>` in place of the item name selects one item exactly.

## Run a command with secrets

```sh
geslar run -e DB_PASSWORD=geslar://Work/Database/password -- ./deploy.sh
geslar run -f secrets.env -- node app.js
```

Every reference is resolved before the command starts; if one fails, the command is not started. Values reach the command only through its environment. The command's output is masked by default (`--no-masking` turns this off). Masking hides the literal value only — not encoded forms such as base64.

`geslar inject -f secrets.env` prints `KEY=value` lines for `source` or a dotenv loader. Its output is **not** masked.

In PowerShell, quote the separator when the command has options of its own: `geslar run -e DB_PASSWORD=geslar://Work/Database/password '--' node -p "1"`.

## Where the local key is kept

After sign-in the CLI keeps its session in an encrypted file in your user profile. The key that seals it lives in your operating system's keychain (Windows Credential Manager, macOS Keychain, Linux Secret Service). Where no keychain is available, the CLI uses a key file protected by a local password you choose (not your master password). `--key-storage <auto|keychain|file>` or `GESLAR_KEY_STORAGE` selects the mode.

## What the server sees

The server never sees your master password, a decrypted value, or which field you read.

It **does** see which item `read`, `run` and `inject` fetch: every fetch is recorded as an `item.accessed` event in the audit log.

## What this does and does not protect against

We would rather you read this before installing than after.

- A command you start with `geslar run` receives the secrets in its environment. If that command is malicious or compromised, it can send them anywhere it can reach. This is true of every tool that injects secrets into a process.
- Masking is a guard against accidental leaks in output, not a security boundary. It matches the literal value only.
- While the Vault is unlocked, other processes running as your operating-system user may be able to use the stored unlock. Run `geslar lock` when you are done, or use `--unlock` for a single command.
- Environment variables of a running process can be read by other processes of the same user. On a shared machine, isolate the process.
- The CLI is online-only; it keeps no offline copy of your Vault.

Further known limitations are listed in the [documentation](https://docs.geslar.app/cli/limitations).

## Planned

Not part of the current release: a non-interactive mode for CI pipelines, agent profiles, and an MCP server. Today `read`, `run` and `inject` need `geslar unlock` or `--unlock`, which needs a terminal.

## Documentation

- [CLI reference](https://docs.geslar.app/cli/reference) — `geslar://` syntax, commands, exit codes
- [Quickstart](https://docs.geslar.app/cli/quickstart) and [known limitations](https://docs.geslar.app/cli/limitations)
- [Help article](https://geslar.app/pomoc/geslar-cli) on geslar.app

## Support

- **Bugs and feature requests** — [open an issue](https://github.com/geslar-eu/geslar-cli/issues)
- **Security vulnerabilities** — do **not** open a public issue; follow [SECURITY.md](./SECURITY.md)
- **Never paste a secret, token, or vault contents into an issue.** Redact before you post.

## License

Proprietary. Copyright © Geslar d.o.o. See [LICENSE.md](./LICENSE.md).

---

Geslar d.o.o. · OIB 43268748957 · Miroslava Krleže 13, 51000 Rijeka, Croatia
