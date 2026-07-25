# Security Policy

Geslar CLI handles credentials. We take reports about it seriously and we would rather hear about a problem early and informally than late and perfectly written up.

## Reporting a vulnerability

**Do not open a public issue for a security problem.**

Use GitHub's private vulnerability reporting on this repository:

**[→ Report a vulnerability](https://github.com/geslar-eu/geslar-cli/security/advisories/new)**

This opens a private advisory visible only to you and to Geslar d.o.o. If you cannot use GitHub, write to `security@geslar.app`.

Please include, as far as you have it: the CLI version (`geslar --version`), your operating system, the MCP client if one is involved, and the steps to reproduce. **Redact any real secret, token, or vault content** — a redacted report is more useful to us than one we have to handle as an incident.

## What to expect

- **Acknowledgement within 3 working days.**
- An initial assessment, with our view of severity and whether we consider it in scope, **within 10 working days**.
- Progress updates at least every 14 days until the report is closed.
- Credit in the release notes and in the published advisory, under whatever name or handle you prefer, unless you ask us not to.

We ask you to give us a reasonable window to ship a fix before publishing. We will agree a disclosure date with you rather than impose one, and we will not ask you to stay quiet indefinitely.

## Supported versions

Geslar CLI is distributed on npm as [`@geslar/cli`](https://www.npmjs.com/package/@geslar/cli). Security fixes are shipped in a new version on the current release line; older versions are not patched in place.

Always report against the latest published version if you can reproduce there.

## Scope

**In scope**

- The `geslar` CLI and its MCP server (`geslar mcp serve`)
- Local token, unlock-grant, and pre-grant storage
- `geslar://` reference resolution, `run`, and `inject`
- Configuration written by `geslar mcp init`

**Out of scope**

- The known limitations documented in [README.md](./README.md#what-this-does-and-does-not-protect-against) and in the [reference documentation](https://docs.geslar.app/cli/reference) — in particular, that a command you have explicitly allowed an agent to run can exfiltrate secrets from its own environment. If you have found a way around a control we *do* claim, that is very much in scope; if you are demonstrating a limitation we already state, tell us anyway, but expect us to point at the documentation.
- Attacks that require an already-compromised operating system account, malware running as the user, or physical access to an unlocked machine. Geslar CLI is not a sandbox against a malicious process running as you.
- Vulnerabilities in third-party dependencies with no demonstrated impact on Geslar CLI.
- Social engineering of Geslar staff or users, and denial of service through volume.
- The Geslar web application, browser extension, and mobile apps — those are separate products; report those through the same channel and we will route them.

## Safe harbour

We will not pursue or support legal action against anyone who reports a vulnerability to us in good faith, keeps to the scope above, avoids privacy violations and service degradation, and does not access, modify, or retain data belonging to anyone else. If you are unsure whether something is in bounds, ask us first.

We do not currently run a paid bug bounty programme.
