---
description: "Classify every agent action by reversibility before executing it. Gate hard-to-reverse and destructive actions."
applyTo: "**"
---

# Agent Trust Tiers

Before executing any action, classify it by reversibility. The classification
determines whether the agent may proceed autonomously or must stop and confirm.

## Tiers

| Tier | Examples | Agent behaviour |
|------|----------|-----------------|
| Read-only | Reading files, searching, listing | Always safe -- no confirmation needed |
| Reversible local edit | Creating or editing files, running tests | Safe -- proceed, but record what changed |
| Hard to reverse | Deleting files, dropping database rows, resetting state | Stop and confirm with the human before executing |
| Destructive / shared system | Pushing to a shared branch, sending messages, deploying, modifying CI | Human gate required -- never automate without explicit instruction |

## Rules

- Classify before acting, not after. If uncertain which tier an action falls into,
  treat it as one tier higher (more cautious).
- Never use a destructive shortcut to unblock a reversible task.
  Example: do not git reset --hard to fix a merge conflict -- resolve it.
- When an action is hard to reverse, describe what will happen and wait for
  confirmation before proceeding. Do not execute and then report.
- Combining a reversible action with a hard-to-reverse action in one step
  elevates the whole step to hard-to-reverse.

## Why

An agent that cannot classify its own actions by risk will eventually cause
data loss or unintended side effects that are expensive to recover from.
The classification is cheap; the recovery is not.
