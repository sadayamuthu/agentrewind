<div align="center">

# AgentRewind

<h3>An undo log for AI agents</h3>

[![Status](https://img.shields.io/badge/status-design%20stage-orange?style=flat-square)](ROADMAP.md)
[![Go 1.26+](https://img.shields.io/badge/go-1.26+-00ADD8?style=flat-square)](https://go.dev/dl/)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue?style=flat-square)](LICENSE)

---

[How it works](#how-it-works) · [Thesis](THESIS.md) · [Roadmap](ROADMAP.md) · [Contributing](CONTRIBUTING.md) · [Security](SECURITY.md)

</div>

---

## What is AgentRewind?

An undo log for AI agents. For every action an agent takes, `agentrewind` records
the action that would undo it (create a bucket and delete the bucket, post a
message and delete the message, push a commit and revert it). One command
then rolls back a whole agent session across the filesystem, git, cloud APIs
and SaaS tools, newest action first.

> **Status: design stage.** There is no code yet. The commands, catalog
> format and output below are the proposal, and they will change. Part of the
> [OpenAstra](https://openastra.org) research agenda, area X: AI Resilience &
> Recovery.

---

## The problem

When an agent goes wrong, the cleanup is manual: someone reads the transcript,
works out what it touched, and undoes each change by hand, often across five
systems. Some actions can't be undone at all, and nobody knew that until after
they ran. Distributed systems solved a version of this decades ago with the
saga pattern (Garcia-Molina and Salem, 1987): every step in a long
transaction has a compensating step. Agents need the same thing.

---

## How it works

1. **Record.** `agentrewind` wraps the agent's tools (as an MCP proxy or a library)
   and logs each call with what it needs to reverse it: IDs it created,
   previous values it overwrote, snapshots of files it changed.
2. **Compensate.** An open catalog maps each action to its compensating
   action, for example `s3.CreateBucket` to `s3.DeleteBucket`, or
   `github.create_issue` to `github.close_issue`.
3. **Classify up front.** Actions are marked `reversible`, `compensable`
   (can be offset but not erased, like a refund) or `irreversible` (a sent
   email). Irreversible actions can require approval *before* they run.
4. **Roll back.** `agentrewind <session>` runs compensations newest first, checks
   each one worked, and stops and reports if one fails.

---

## Example (proposed)

```console
$ agentrewind plan session-42
session-42  agent: release-bot  23 actions  2026-11-16 09:12-09:31

 23  git.push origin release/2.4        -> git revert (3 commits)      reversible
 22  slack.post #releases               -> slack.delete message        reversible
 ...
  9  stripe.refund ch_3Q...             -> none                        IRREVERSIBLE (was approved)
  1  fs.write CHANGELOG.md              -> restore snapshot            reversible

22 of 23 actions can be undone. Run `agentrewind apply session-42` to proceed.
```

---

## The compensation catalog

A machine-readable catalog of actions and their compensations for common APIs
(filesystem, git, GitHub, AWS, GCP, Azure, Slack, Jira, Stripe), versioned
and open to contributions. Each entry lists what must be recorded at call time
for the undo to work.

---

## Planned surfaces

- MCP proxy and Python/TypeScript libraries that record calls
- CLI: `agentrewind log`, `agentrewind plan`, `agentrewind apply`
- Compensation catalog as JSON, published like OCBC and OEBC
- Pre-execution approval hook for irreversible actions

---

## Maps to

| Framework | Items |
|-----------|-------|
| OWASP Agentic Top 10 | Tool misuse, cascading failures |
| NIST SP 800-53 Rev. 5 | CP-10 System Recovery and Reconstitution, IR-4 Incident Handling, CM-3 Configuration Change Control |

---

## Design principles

- **Know before you act.** Reversibility is decided before a call runs, not after.
- **Verify every undo.** A compensation isn't done until its effect is checked.
- **Stop on failure.** Never keep rolling back past a step that failed.

---

## Roadmap

The plan, from the first working version to 1.0, is in [ROADMAP.md](ROADMAP.md). The reasoning behind the project, and what would make us stop, is in [THESIS.md](THESIS.md).

---

## Built on switchyard

`agentrewind` will be a Go binary built on [switchyard](https://github.com/sadayamuthu/switchyard), OpenAstra's shared MCP proxy core: the recorder is a switchyard hook that logs each call and the data needed to undo it. It stacks with the other switchyard tools by chaining (`agent → agentfuse → agentlock → agenttaint → agentrewind → MCP server`).

---

## Related OpenAstra projects

- [Unasked](https://github.com/sadayamuthu/unasked): exact, verified undo for code edits; AgentRewind extends that to every system
- [AgentFuse](https://github.com/sadayamuthu/agentfuse): stops the agent; AgentRewind cleans up after it
- [AgentRisk](https://github.com/sadayamuthu/agentrisk): uses reversibility in its score

---

## Contributing

Design feedback, real-world cases and fixes are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for how to help while the project is at the design stage, and [GOVERNANCE.md](GOVERNANCE.md) for how decisions are made.

---

## Security

Please report vulnerabilities privately through [GitHub security advisories](https://github.com/sadayamuthu/agentrewind/security/advisories/new), not in a public issue. See [SECURITY.md](SECURITY.md) for what to include and what's in scope.

---

## License

[Apache License 2.0](LICENSE)

---

<div align="center">

**AgentRewind: an undo log for AI agents**

AgentRewind is managed by [OpenAstra](https://openastra.org)

[GitHub](https://github.com/sadayamuthu/agentrewind) · [Thesis](THESIS.md) · [Roadmap](ROADMAP.md)

</div>
