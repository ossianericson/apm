---
description: "How to manage multi-step agent sessions: task record, per-step updates, crash recovery, and separating task state from conversation history."
applyTo: "**"
---

# Session Discipline

## Create a task record before any work starts

The first action in any multi-step session must be creating a task record --
a plain text file or structured list of numbered steps with status markers.

Do not start editing files, running commands, or making changes before
the task record exists. If the session is interrupted, the task record is
the only reliable way to resume from the correct position.

## Update the task record after every step

Mark each step complete (or failed) immediately after it finishes.
Do not batch updates. Do not defer to the end of the session.

This is crash-recovery critical: if the session is interrupted, the next
agent invocation reads the task record and resumes from the first incomplete step.

Example:

  Step 1 -- create task record     [done]
  Step 2 -- create branch          [done]
  Step 3 -- edit file A            [done]
  Step 4 -- run tests              [in progress]
  Step 5 -- commit                 [not started]

If step 4 fails, the next agent sees exactly where to resume.

## Separate task state from conversation history

Conversation history is ephemeral. It is not a reliable record of what was done.
The task record is the durable artefact. When resuming a session, read the task
record -- not the chat history -- to determine current state.

Never rely on "I remember we did X earlier in this conversation" as a substitute
for a written task record. The conversation may be summarised, truncated, or
started fresh.

## Resume protocol

When resuming a failed or interrupted session:

1. Read the task record completely before doing anything else.
2. Skip steps already marked done.
3. Retry steps marked failed.
4. Continue from the first step marked not started or in progress.
5. Run any required verification steps before the final commit.

## Why

A session without a task record produces two failure modes:
- Silent incompletion: some steps were skipped but nobody knows which.
- Restart from scratch: the next agent repeats already-completed work.

Both are more expensive than writing five lines of status before starting.
