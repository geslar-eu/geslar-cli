# Contributing

Thank you for wanting to help. Because Geslar CLI's source code is not public, contribution here works a little differently from a typical open-source project — so this page sets out plainly what is useful and what is not possible.

## What is genuinely useful

**Bug reports.** By far the most valuable thing you can send us. Use the [issue templates](https://github.com/geslar-eu/geslar-cli/issues/new/choose). A report with the CLI version, the operating system, the MCP client if one is involved, and the exact commands you ran is worth more than a long description.

**Feature requests.** Tell us the problem you are trying to solve, not only the feature you have in mind. We often find a better answer that way.

**Documentation corrections.** If something at [docs.geslar.app](https://docs.geslar.app/cli/reference) is wrong, unclear, or out of date, open an issue and quote the passage. Documentation errors in a security tool are security-relevant and we treat them that way.

**MCP client configurations.** If you have Geslar CLI running with a client we do not list — Windsurf, Zed, LM Studio, Goose, or anything else — send us the configuration that worked. That is how the supported-client list grows.

## What we cannot accept

**Pull requests containing code.** The source is not in this repository, so there is nothing here to patch. We are not able to merge code contributions, and we would rather say so up front than leave a PR open for months.

**Anything containing a real secret.** Never paste a password, token, API key, or vault content into an issue, a comment, or a screenshot. Redact first. If you have already posted one, revoke it immediately and tell us — deleting the comment is not enough, because it stays in the edit history.

## Security reports

Do not use the issue tracker. Follow [SECURITY.md](./SECURITY.md).

## Before you open an issue

Please check the [CHANGELOG](./CHANGELOG.md) and the [known limitations](https://docs.geslar.app/cli/reference) first — some behaviour that looks like a bug is a documented and deliberate design decision, particularly around what the MCP tools refuse to return.

Reproduce on the latest published version if you can (`npm install -g @geslar/cli@latest`), and say which version you tested.

## Code of conduct

Participation in this repository is covered by our [Code of Conduct](./CODE_OF_CONDUCT.md).
