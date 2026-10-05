# Security Policy

Geslar CLI handles credentials. We take reports about it seriously and we would rather hear about a problem early and informally than late and perfectly written up.

## Reporting a vulnerability

**Do not open a public issue for a security problem.**

Use either of these:

- **GitHub private vulnerability reporting** on this repository: **[Report a vulnerability](https://github.com/geslar-eu/geslar-cli/security/advisories/new)**. It opens a private advisory visible only to you and to Geslar d.o.o.
- **Email** `security@geslar.app`.

Please include, as far as you have it: the CLI version (`geslar --version`), your operating system and Node.js version, and the steps to reproduce. **Redact any real secret, token (a `gsm_…` token included) or Vault content.** A redacted report is more useful to us than one we have to handle as an incident.

## What to expect

These are targets we work to, not guarantees:

- We aim to **acknowledge** your report **within 3 working days**.
- We aim to give an **initial assessment**, with our view of severity and whether we consider it in scope, **within 10 working days**.
- We aim to **update you at least every 14 days** until the report is closed.
- We are happy to **credit you** in the release notes and in the published advisory, under whatever name or handle you prefer, unless you ask us not to.

We ask you to give us a reasonable window to ship a fix before publishing. We will agree a disclosure date with you rather than impose one, and we will not ask you to stay quiet indefinitely.

## Supported versions

Geslar CLI is distributed on npm as [`@geslar/cli`](https://www.npmjs.com/package/@geslar/cli).

| Version | Supported |
|---|---|
| 1.0.x | yes |
| 0.x (0.1.0 to 0.3.2) | no |
| 1.0.0-rc.x (trial versions) | no |

Security fixes are shipped as a new 1.0.x version; older versions are not patched in place. Please report against the latest published version if you can reproduce it there.

## Scope

**In scope**

- The `geslar` CLI (`@geslar/cli`).
- The **MCP server** (`geslar mcp serve`, `geslar mcp init`, `geslar mcp allow-cmd`) and its five tools, including what an agent can learn, start or read through them.
- **Agent profiles and approvals** as the CLI handles them: the sealed profile, the token prompt, the approval request and its exit code 8.
- **CI mode** (`GESLAR_SERVICE_TOKEN`): the in-memory service session, and any way the token or a secret could reach disk, the child process of `geslar run`, an output or an error message.
- Local storage of keys and sessions: the OS keychain entry, the password-protected key file, and the sealed session and unlock records.
- `geslar://` reference resolution, `geslar run` (environment injection and output masking), `geslar inject`, and the sanitising of names and error text before they reach a terminal or an agent.

**Out of scope**

- The limitations the README states in [What this does not protect against](./README.md#what-this-does-not-protect-against), in particular that a command you start with `geslar run` (or allow an agent to start) can send the secrets in its own environment anywhere it can reach, and that masking covers the literal value only. If you have found a way around a control we *do* claim, that is very much in scope. If you are demonstrating a limitation we already state, tell us anyway, but expect us to point at the README.
- Attacks that need an already compromised operating-system account, malware running as the user, root or administrator access, or physical access to an unlocked machine. Geslar CLI is not a sandbox against a malicious process running as you.
- Vulnerabilities in third-party dependencies with no demonstrated impact on Geslar CLI.
- Social engineering of Geslar staff or users, and denial of service through volume.
- The Geslar web application and the browser extension. Those are separate products; you may report them through the same channel and we will route them.

## Safe harbour

We will not pursue or support legal action against anyone who reports a vulnerability to us in good faith, keeps to the scope above, avoids privacy violations and service degradation, and does not access, modify or retain data belonging to anyone else. If you are unsure whether something is in bounds, ask us first.

We do not currently run a paid bug bounty programme.
