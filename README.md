# Not Yet

Keep your Mac awake while an agent or command finishes, then let it sleep
normally—or put it to sleep when the work is done.

Not Yet uses job lifecycles instead of an indefinite wake setting or a guessed
timer. It is both a Codex skill and a small macOS CLI built on the system
`caffeinate` command.

## Install as a Codex skill

```bash
git clone https://github.com/alek-sys/not-yet.git ~/.codex/skills/not-yet
```

New Codex sessions can then invoke `$not-yet`. For example:

```text
Use $not-yet to keep this Mac awake until the current job is finished.
```

## Use the CLI

Run a command under a wake lease:

```bash
./scripts/not-yet run --name "full test suite" -- make test
```

The wake assertion ends with the command and the command's exit status is
preserved.

Hold a lease for an already-running job:

```bash
./scripts/not-yet hold --name "agent refactor"
```

Keep that command in the foreground. Press Control-C when the job finishes to
release its lease and restore the normal macOS sleep policy.

If the Mac should go to sleep after the job:

```bash
./scripts/not-yet hold --name "overnight agent run" --sleep-after
```

After Control-C, Not Yet waits five seconds so an agent can deliver its final
response, checks that no other lease is active, and then requests system sleep.

Inspect active jobs or request sleep directly:

```bash
./scripts/not-yet status
./scripts/not-yet sleep
```

`sleep` refuses while another Not Yet lease is active.

## How it behaves

- Creates an independent wake lease per job, so one agent cannot release
  another agent's lease.
- Prevents system sleep while allowing the display to turn off.
- Cleans stale lease records by comparing them with live macOS power
  assertions.
- Changes no persistent power-management settings and installs no privileged
  helper.
- Supports macOS only.

The system-sleep assertion applies only on AC power. Standard `caffeinate`
assertions also do not reliably override closed-lid sleep on every Mac and
power state.

See [SKILL.md](SKILL.md) for the agent workflow and
[`scripts/not-yet`](scripts/not-yet) for the CLI implementation.
