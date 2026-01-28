# Troubleshooting
## DM works but remote alarm doesn't
Check the log:
  tail -n 200 ~/.clawdbot/nibble_vega_alarm.log

## paplay: open(): No such file or directory
Often means the remote sound argument got corrupted (positional args/quoting).
Fix: pass TASK/SOUND safely through SSH using shell-escaped tokens (printf %q).
