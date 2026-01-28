# Nibble Reminders 🦞🔔
Reliable Discord DM reminders + *remote alarm ringing* across Linux machines.

Built from the trenches: clawdbot (Discord DMs) + systemd user timers (accurate scheduling) + SSH (remote ring).

## Why this exists
“DM delivered but SCHEDULER_HOST didn’t ring” is a special kind of pain.
This repo documents the exact failure modes and the fix that made it deterministic.

## What it does
- **DM-only** reminders via clawdbot cron jobs (session `isolated`)
- **Local alarm** on the scheduler machine (remote-host)
- **Remote alarm** on another machine (SCHEDULER_HOST) via SSH, with correct audio/DBus environment
- Cancel last alarm (local + remote)

## The winning design
### 1) Use clawdbot cron for Discord DMs
Cron jobs in the `isolated` session were reliable. The `main` session had weird “skipped/special rules”.

### 2) Use systemd user timers for accurate scheduling
For timer drift, we forced accuracy:
- `AccuracySec=1s`
- `RandomizedDelaySec=0`

### 3) For remote alarms, schedule locally, ring remotely at fire-time
**Relative alarms** (`10s`, `5m`, etc.) are most reliable when:
- a local systemd user timer fires on remote-host
- at fire-time it SSH’es into SCHEDULER_HOST and plays the sound immediately

This avoids background subshells dying early and other “shell lifecycle” weirdness.

## The real bug (and why it was so sneaky)
We initially passed task and sound as positional args (`$1`, `$2`) through nested shells and SSH.
It looked fine… until logs proved the remote `$2` was literally `"ring"` (from `"SCHEDULER_HOST ring test ✅"`),
so `paplay` tried to open a file named `ring` and failed.

Then we tried env vars (`env TASK="$TASK" ...`) but SSH quoting split the task into words.

### Final fix: shell-escape values before shipping over SSH
We pass task/sound using shell-escaped tokens (`printf %q`) and then run remote bash:

- `task_q="$(printf %q "$TASK")"`
- `ssh host "TASK=$task_q SOUND=$sound_q bash -s" ...`

Result: task strings with spaces/emojis stay intact, and the remote sound path arrives correctly every time.

## Install
1) Copy the template file into place:
```bash
cp -av bash/.bash_aliases_nibble.template ~/.bash_aliases_nibble


eof

## Quiet by default (debug logging)
Remote alarm scripts are quiet by default. To enable debug logs:

```bash
export NIBBLE_DEBUG=1

PY
PY

Quiet by default (debug logging)
Remote alarm scripts are quiet by default. To enable debug logs:
export NIBBLE_DEBUG=1

Logs (when enabled):
~/.clawdbot/nibble_SCHEDULER_HOST_alarm.log

Less loud
Change the sound file or lower volume. Example:
export NIBBLE_ALARM_SOUND=/usr/share/sounds/freedesktop/stereo/message.oga
