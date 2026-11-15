# rewind

An undo log for AI agents. For every action an agent takes, `rewind` records
the action that would undo it (create a bucket and delete the bucket, post a
message and delete the message, push a commit and revert it). One command
then rolls back a whole agent session across the filesystem, git, cloud APIs
and SaaS tools, newest action first.

> **Status: design stage.** There is no code yet. The commands, catalog
> format and output below are the proposal, and they will change. Part of the
> [OpenAstra](https://openastra.org) research agenda, area X: AI Resilience &
> Recovery.

## The problem

When an agent goes wrong, the cleanup is manual: someone reads the transcript,
works out what it touched, and undoes each change by hand, often across five
systems. Some actions can't be undone at all, and nobody knew that until after
they ran. Distributed systems solved a version of this decades ago with the
saga pattern (Garcia-Molina and Salem, 1987): every step in a long
transaction has a compensating step. Agents need the same thing.

## How it works

1. **Record.** `rewind` wraps the agent's tools (as an MCP proxy or a library)
   and logs each call with what it needs to reverse it: IDs it created,
   previous values it overwrote, snapshots of files it changed.
2. **Compensate.** An open catalog maps each action to its compensating
   action, for example `s3.CreateBucket` to `s3.DeleteBucket`, or
   `github.create_issue` to `github.close_issue`.
3. **Classify up front.** Actions are marked `reversible`, `compensable`
   (can be offset but not erased, like a refund) or `irreversible` (a sent
   email). Irreversible actions can require approval *before* they run.
4. **Roll back.** `rewind <session>` runs compensations newest first, checks
   each one worked, and stops and reports if one fails.

## Example (proposed)

```console
$ rewind plan session-42
session-42  agent: release-bot  23 actions  2026-11-16 09:12-09:31

 23  git.push origin release/2.4        -> git revert (3 commits)      reversible
 22  slack.post #releases               -> slack.delete message        reversible
 ...
  9  stripe.refund ch_3Q...             -> none                        IRREVERSIBLE (was approved)
  1  fs.write CHANGELOG.md              -> restore snapshot            reversible

22 of 23 actions can be undone. Run `rewind apply session-42` to proceed.
```

## The compensation catalog

A machine-readable catalog of actions and their compensations for common APIs
(filesystem, git, GitHub, AWS, GCP, Azure, Slack, Jira, Stripe), versioned
and open to contributions. Each entry lists what must be recorded at call time
for the undo to work.

## Planned surfaces

- MCP proxy and Python/TypeScript libraries that record calls
- CLI: `rewind log`, `rewind plan`, `rewind apply`
- Compensation catalog as JSON, published like OCBC and OEBC
- Pre-execution approval hook for irreversible actions

## Maps to

| Framework | Items |
|-----------|-------|
| OWASP Agentic Top 10 | Tool misuse, cascading failures |
| NIST SP 800-53 Rev. 5 | CP-10 System Recovery and Reconstitution, IR-4 Incident Handling, CM-3 Configuration Change Control |

## Design principles

- **Know before you act.** Reversibility is decided before a call runs, not after.
- **Verify every undo.** A compensation isn't done until its effect is checked.
- **Stop on failure.** Never keep rolling back past a step that failed.

## Roadmap

1. Action log format and the filesystem and git compensations
2. MCP proxy recorder
3. Compensation catalog v0 (GitHub, Slack, AWS core services)
4. `plan` and `apply` with verification
5. Pre-execution approval for irreversible actions

## Related OpenAstra projects

- [Unasked](https://github.com/sadayamuthu/unasked): exact, verified undo for code edits; Rewind extends that to every system
- [Fuse](https://github.com/sadayamuthu/fuse): stops the agent; Rewind cleans up after it
- [AgentRisk](https://github.com/sadayamuthu/agentrisk): uses reversibility in its score

## License

To be decided before the first release.
