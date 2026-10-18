# lora-log

Logs every packet a Meshtastic radio hears to a file, one JSON object per line.

I run it on a laptop with a Heltec WiFi LoRa 32 V4 plugged in over USB. The
radio's own node list can drop entries after an unplug or reboot, so the log on
disk is the record of what was actually heard.

- Each packet is written and flushed as it arrives, so a suspend or a pulled
  cable loses nothing already heard.
- On connect it also saves a snapshot of every node the radio remembers.
- Files go to `~/.local/state/lora/YYYY-MM-DD.jsonl`.
- If the radio disconnects, the script exits and systemd restarts it.

## Install

Needs Python 3 and the `meshtastic` package (`pip install meshtastic`, or
`uv tool install meshtastic` and point the shebang at that tool's python).

```
cp lora-log ~/.local/bin/
cp lora-log.service ~/.config/systemd/user/
systemctl --user enable --now lora-log
```

Set `LORA_PORT` in the unit to the radio's `/dev/serial/by-id/...` path. Stop
the service before using the `meshtastic` CLI, because it holds the serial port.

## Reading the log

```
jq -r 'select(.type=="packet") | [.logged_at[11:19], .fromId, .decoded.portnum, .rxSnr, .rxRssi] | @tsv' \
  ~/.local/state/lora/$(date +%F).jsonl
```
