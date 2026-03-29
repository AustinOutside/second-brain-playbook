# Sync Script Reference

The sync daemon is the engine that makes the dual-Mac setup work. Here's how it operates and how to customize it.

## What It Does

Every 60 seconds, the script:

1. **Finds the newest Cowork session** by scanning `~/Library/Application Support/Claude/local-agent-mode-sessions/`
2. **Detects new sessions** — if the session directory changed since last run, treats iCloud as source of truth and seeds the new session
3. **Computes MD5 hashes** of both the local (Cowork) and iCloud versions of each file
4. **Compares against last-known hashes** stored in `~/.claude-sync/state/`
5. **Syncs the changed version** — if local changed, pushes to iCloud; if iCloud changed, pulls to local
6. **Logs everything** to `~/.claude-sync/sync.log`

## Key Design Decisions

### Why MD5 Hashes (Not Timestamps)?
iCloud Drive doesn't preserve file modification timestamps between machines. Two Macs will see different `mtime` values for the same file. MD5 hashes are content-based and always reliable.

### Why Session Detection?
Cowork creates a new directory (`local_<uuid>`) for each conversation. When a new session starts, it has a blank default CLAUDE.md. Without session detection, the daemon would push this blank file to iCloud, wiping out your real data. The script detects new sessions and seeds them from iCloud instead.

### Why iCloud (Not Dropbox/Git/etc.)?
- Already built into macOS — no extra software
- Handles large files and binary content
- Both Macs must share an Apple ID anyway for other reasons
- No API keys, tokens, or accounts to manage

## File Structure

```
~/.claude-sync/
├── cowork-sync.sh          # The sync script
├── sync.log                # Activity log
├── sync-stderr.log         # Error log
└── state/                  # Hash state files
    ├── claude-md-local.hash
    ├── claude-md-icloud.hash
    ├── tasks-md-local.hash
    ├── tasks-md-icloud.hash
    └── last-session         # Tracks current session path

~/Library/Mobile Documents/com~apple~CloudDocs/claude-sync/
├── CLAUDE.md               # Canonical copy
├── TASKS.md                # Canonical copy
├── memory/                 # Canonical copy
└── resources/              # Playbooks, checklists (optional)
```

## Generating Your Script

The easiest way to create the sync script is to ask Claude directly:

> "Create a bash sync script at ~/.claude-sync/cowork-sync.sh that:
> 1. Finds the newest Cowork session directory under ~/Library/Application Support/Claude/local-agent-mode-sessions/
> 2. Syncs CLAUDE.md, TASKS.md, and memory/ between that session and ~/Library/Mobile Documents/com~apple~CloudDocs/claude-sync/
> 3. Uses MD5 hashes for change detection (not timestamps)
> 4. Detects new sessions and seeds them from iCloud
> 5. Logs all activity to ~/.claude-sync/sync.log
> 6. Handles the memory/ directory recursively"

Claude will generate the full script tailored to your system.

## Core Functions

If you want to understand or customize the script, here are the key functions:

### Hash Function
```bash
file_hash() {
    md5 -q "$1" 2>/dev/null || echo "none"
}
```

### Session Detection
```bash
SESSIONS_BASE=~/Library/Application\ Support/Claude/local-agent-mode-sessions
LATEST_LOCAL_DIR=$(find "$SESSIONS_BASE" -type d -name "local_*" -maxdepth 3 2>/dev/null | sort | tail -1)

LAST_SESSION=$(cat "$STATE_DIR/last-session" 2>/dev/null || echo "")
if [ -n "$LATEST_LOCAL_DIR" ] && [ "$LATEST_LOCAL_DIR" != "$LAST_SESSION" ]; then
    # New session detected — seed from iCloud
    log "SESSION: New session detected, resetting sync state"
    rm -f "$STATE_DIR"/*.hash
    echo "$LATEST_LOCAL_DIR" > "$STATE_DIR/last-session"
fi
```

### Sync Logic
```bash
sync_file() {
    local filename="$1"
    local local_path="$LATEST_LOCAL_DIR/.claude/$filename"
    local icloud_path="$ICLOUD_DIR/$filename"
    local local_hash_file="$STATE_DIR/${filename//\//-}-local.hash"
    local icloud_hash_file="$STATE_DIR/${filename//\//-}-icloud.hash"

    local current_local_hash=$(file_hash "$local_path")
    local current_icloud_hash=$(file_hash "$icloud_path")
    local last_local_hash=$(cat "$local_hash_file" 2>/dev/null || echo "none")
    local last_icloud_hash=$(cat "$icloud_hash_file" 2>/dev/null || echo "none")

    if [ "$current_local_hash" != "$last_local_hash" ] && [ "$current_local_hash" != "none" ]; then
        # Local changed — push to iCloud
        cp "$local_path" "$icloud_path"
        log "SYNC: $filename Cowork → iCloud (local changed)"
    elif [ "$current_icloud_hash" != "$last_icloud_hash" ] && [ "$current_icloud_hash" != "none" ]; then
        # iCloud changed — pull to local
        cp "$icloud_path" "$local_path"
        log "SYNC: $filename iCloud → Cowork (iCloud changed)"
    fi

    # Update stored hashes
    file_hash "$local_path" > "$local_hash_file"
    file_hash "$icloud_path" > "$icloud_hash_file"
}
```

## The Launch Daemon

The plist file at `~/Library/LaunchAgents/com.claude.cowork-sync.plist` tells macOS to run the script every 60 seconds:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.claude.cowork-sync</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/Users/YOURUSERNAME/.claude-sync/cowork-sync.sh</string>
    </array>
    <key>StartInterval</key>
    <integer>60</integer>
    <key>RunAtLoad</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/Users/YOURUSERNAME/.claude-sync/sync.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/YOURUSERNAME/.claude-sync/sync-stderr.log</string>
</dict>
</plist>
```

> Replace `YOURUSERNAME` with your actual macOS username.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| "Operation not permitted" errors | Add `/bin/bash` to Full Disk Access |
| Daemon not running | `launchctl load ~/Library/LaunchAgents/com.claude.cowork-sync.plist` |
| New session has blank context | Wait 60s or force sync: `bash ~/.claude-sync/cowork-sync.sh` |
| Both machines overwriting each other | Verify both run v3 script: `head -3 ~/.claude-sync/cowork-sync.sh` |
| Reset everything | `rm -rf ~/.claude-sync/state/ && launchctl unload ... && launchctl load ...` |
