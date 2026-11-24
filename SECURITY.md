# Security Policy

## Reporting a vulnerability

Please don't report security problems in a public issue, pull request or
discussion. Use
[**Report a vulnerability**](https://github.com/sadayamuthu/agentrewind/security/advisories/new)
on the repository's Security tab instead. Only the maintainers can see the
report.

Include what you can of:

- the AgentRewind version (`agentrewind version`) or commit, your Go version and OS
- how AgentRewind was used, and its configuration with secrets removed
- steps to reproduce
- what happened and what you expected

You'll get a reply when the report has been read, and again when a fix is
ready. A fix ships as a patch release; the advisory names you unless you'd
rather it didn't.

## Supported versions

AgentRewind is at the design stage and has no releases yet. Once it does,
fixes land in the latest release only, until 1.0.

## What counts

AgentRewind records an agent's tool calls with the data needed to undo them, and `agentrewind apply` runs compensating actions with the credentials you configure.

AgentRewind is a security tool, so anything that breaks one of these promises
is a security issue:

- **Rollback stops on failure.** A rollback that continues past a failed or unverified compensation is in scope.
- **Action logs are protected.** Logs contain the values needed to undo actions, including overwritten data. A path that writes them anywhere except the configured log location is in scope.
- **Irreversible actions are flagged before they run.** An action the catalog marks irreversible that runs without the configured approval is in scope.

These are by design, and not vulnerabilities:

- Some actions can only be offset, not erased (a refund, a sent email). AgentRewind tells you which.
- A compensation can conflict with changes others made since; `agentrewind plan` shows what will run.

A wrong result that isn't covered above is a bug, not a vulnerability. Report
it in a normal issue; see [CONTRIBUTING.md](CONTRIBUTING.md).
