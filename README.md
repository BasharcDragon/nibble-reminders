# Nibble Reminders 🦞🔔
Discord DM reminders via clawdbot + reliable alarms on local or remote Linux machines using systemd user timers + SSH.

This repo exists because shell quoting is haunted.

## What it does
- DM-only reminders
- DM + alarm on the same machine
- DM + alarm on a remote machine (e.g., Vega) triggered reliably at fire-time

## Key fix
Remote alarms are triggered by a *local systemd user timer* (accurate) that SSH'es the remote host at fire-time.
Task/sound are passed safely through SSH using shell-escaped tokens to avoid positional-arg/quoting corruption.

See `bash/.bash_aliases_nibble.template` for a template setup.
