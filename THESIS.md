# Rewind: thesis

**In one line:** if every agent action records its compensating action at
call time, cleaning up after an agent becomes one command instead of an
afternoon.

## The problem

When an agent goes wrong, someone reads the transcript, works out what it
touched and undoes each change by hand, across several systems. Some actions
turn out to be irreversible, and nobody knew until after they ran.

## The bet

1. **The saga pattern fits agents.** Every step of a long-running
   transaction having a compensating step (Garcia-Molina and Salem, 1987) maps
   cleanly onto agent tool calls.
2. **Undo data must be captured at call time.** IDs created and values
   overwritten are cheap to record before or during the call and impossible
   to reconstruct afterwards.
3. **Classifying reversibility up front is the real win.** Knowing an action
   is irreversible before it runs lets you require approval, which prevents
   more damage than any rollback.
4. **An open compensation catalog** for common APIs can be built once and
   shared, like OCBC and OEBC.

## Why now

Agents now act across git, cloud and SaaS in a single session, and the
proxy layer needed to record calls is being built anyway for other
OpenAstra projects.

## What success looks like

- A session across filesystem, git, GitHub and Slack rolls back correctly
  with verification on each step.
- The catalog covers the most-used actions of the core APIs, with external
  contributions.
- AgentRisk uses its reversibility classes in scores.

## What would prove us wrong

- Too many real actions are compensable only with side effects (emails
  already read, webhooks already fired), so rollback gives false confidence.
- Concurrent changes by humans or other agents make compensations unsafe
  more often than not.
- Maintaining the catalog for fast-moving APIs is more work than the
  community will sustain.

## Not a bet on

Stopping agents (Fuse) or sandboxing calls before they run (Airlock).

## Where it fits

Area X, AI Resilience & Recovery. Records on the shared proxy core;
extends [Unasked](https://github.com/sadayamuthu/unasked)'s verified undo;
feeds [AgentRisk](https://github.com/sadayamuthu/agentrisk).
