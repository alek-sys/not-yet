---
name: not-yet
description: Keep a Mac awake for exactly the lifetime of an agent job, long-running command, build, test, download, render, or other task, then release the wake assertion or request sleep when finished. Use on macOS when the user asks to keep the laptop or computer awake, prevent idle sleep while work runs, avoid an indefinite Caffeine session or guessed timer, inspect active wake leases, let the Mac sleep after work, or put the Mac to sleep after the last job completes.
---

# Not Yet

Use `scripts/not-yet` from this skill directory. It creates independent
per-job leases with macOS `caffeinate`; one agent cannot release another
agent's lease.

## Choose a lifecycle

- Prefer `run -- command` when launching the whole job. The lease follows the
  wrapper process and is released after success, failure, or interruption.
- For an already-running agent turn, start `hold` in a persistent command
  session. Retain the command session ID and finish it at every terminal
  outcome, including failure and blocked outcomes.
- Run `status` to inspect active leases and clean stale records.

## Run a command under a wake lease

```bash
scripts/not-yet run --name "full test suite" -- make test
```

Preserve the wrapped command's exit status. Do not add a guessed duration.

## Hold a lease for the current agent job

Start this as a long-running TTY command with a short initial yield. Add
`--sleep-after` at the start only if the user explicitly asked to put the Mac
to sleep when the job finishes:

```bash
scripts/not-yet hold --name "implement login flow"
scripts/not-yet hold --name "implement login flow" --sleep-after
```

Wait for `HOLDING`, retain the running command's session ID, and continue the
job. Do not background this command: some agent harnesses reap background
processes when a tool call ends.

After all work and verification are finished, send Ctrl-C to that same command
session. The handle releases its own lease. With `--sleep-after`, it then waits
through the delivery grace period and sleeps only if no other lease is active.

For Codex, use the command-session input operation (for example,
`write_stdin`) to send `\u0003`. The grace period lets the final agent response
reach the user. Never add `--sleep-after` merely because the user asked to let
the Mac sleep normally.

## Inspect or request sleep

```bash
scripts/not-yet status
scripts/not-yet sleep
```

The standalone `sleep` command refuses while any lease is active, then waits
through the same delivery grace period before checking again. It is an
explicit, state-changing operation; use it only when the user requested system
sleep.

## Observe macOS limits

- The tool uses `caffeinate -i -s`: keep the system working while allowing the
  display to turn off. The `-s` assertion applies only on AC power.
- Do not promise closed-lid operation. Standard `caffeinate` assertions do not
  reliably override clamshell sleep on every Mac and power state.
- Do not change persistent `pmset` settings or install privileged helpers.
- On non-macOS systems, report the unsupported platform and do not improvise a
  platform-specific replacement.
