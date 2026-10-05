# Contributing

Thank you for wanting to help. Geslar CLI's source code is **not public**: it lives in a private repository of Geslar d.o.o. This repository contains the public texts only (README, CHANGELOG, security policy, issue templates), so contribution here works a little differently from a typical open-source project. This page sets out plainly what is useful and what is not possible.

## What is genuinely useful

**Bug reports.** By far the most valuable thing you can send us. Use the [issue templates](https://github.com/geslar-eu/geslar-cli/issues/new/choose). A report with the CLI version (`geslar --version`), the operating system, the Node.js version and the exact commands you ran, with secrets redacted, is worth more than a long description. Include the exit code if there was one.

**Feature requests.** Tell us the problem you are trying to solve, not only the feature you have in mind. We often find a better answer that way.

**Documentation corrections.** If something in the [README](./README.md) is wrong, unclear or out of date, open an issue and quote the passage. Documentation errors in a security tool are security-relevant and we treat them that way.

## What we cannot accept

**Pull requests containing code.** The source is not in this repository, so there is nothing here to patch. We are not able to merge code contributions, and we would rather say so up front than leave a PR open for months.

**Anything containing a real secret.** Never paste a password, a token (a `gsm_…` token of an agent or a service account, or the value of `GESLAR_SERVICE_TOKEN`), an API key, `.env` contents or Vault content into an issue, a comment or a screenshot. Redact first. If you have already posted one, revoke it immediately (for an agent or service account: in the Geslar web app, *Settings → Agents & automation*) and tell us. Deleting the comment is not enough, because it stays in the edit history.

## Security reports

Do not use the issue tracker. Follow [SECURITY.md](./SECURITY.md).

## Before you open an issue

Please check the [CHANGELOG](./CHANGELOG.md) and the [README](./README.md), in particular [What this does not protect against](./README.md#what-this-does-not-protect-against): some behaviour that looks like a bug is a documented and deliberate design decision, for example that `geslar inject` output is not masked, or that an agent profile cannot print a value.

Reproduce on the latest published version if you can (`npm install -g @geslar/cli`), and say which version you tested.

## Code of conduct

Participation in this repository is covered by our [Code of Conduct](./CODE_OF_CONDUCT.md).
