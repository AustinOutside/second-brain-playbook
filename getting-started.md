# Getting Started

A step-by-step guide to setting up your Claude Second Brain with dual-Mac sync.

**Time required:** ~30 minutes for the initial setup

**What you'll need:**
- Two Macs signed into the same Apple ID with iCloud Drive enabled
- Claude Desktop app installed on both ([download here](https://claude.ai/download))
- Basic comfort with Terminal (copy-paste level is fine)

---

## Phase 1: Prepare Both Macs (~5 min)

### 1.1 Enable iCloud Drive
On both Macs: System Settings > Apple ID > iCloud > iCloud Drive > make sure it's ON.

### 1.2 Add bash to Full Disk Access
The sync daemon needs permission to write to iCloud Drive. On **both** Macs:

1. System Settings > Privacy & Security > Full Disk Access
2. Click the + button
3. Press Cmd+Shift+G and type `/bin`
4. Select `bash` and click Open
5. Make sure the toggle is ON

### 1.3 Install Claude Desktop
Download and install from [claude.ai/download](https://claude.ai/download) on both Macs. Open it once to create the initial directories.

---

## Phase 2: Create the Sync Infrastructure (~5 min)

Run these commands on **both** Macs:

**Create the sync directory:**
```bash
mkdir -p ~/.claude-sync/state
```

**Create the iCloud sync folder** (only needed on one Mac — iCloud creates it on the other):
```bash
mkdir -p ~/Library/Mobile\ Documents/com~apple~CloudDocs/claude-sync
```

---

## Phase 3: Create the Sync Script (~5 min)

The easiest way: open Claude Cowork on Mac 1 and say:

> "I want to set up a dual-Mac second brain. Create a sync script at ~/.claude-sync/cowork-sync.sh that syncs CLAUDE.md, TASKS.md, and memory/ between my Cowork sessions and iCloud Drive. Use MD5 hash-based change detection, detect new sessions and seed from iCloud, and log all activity."

Claude will generate the full script customized for your system. Alternatively, see [sync-script.md](sync-script.md) for the reference implementation.

After Claude creates the script, make it executable:
```bash
chmod +x ~/.claude-sync/cowork-sync.sh
```

**Copy the script to Mac 2** via iCloud, AirDrop, or just repeat the process on Mac 2.

---

## Phase 4: Create the Launch Daemon (~5 min)

This makes the sync run automatically every 60 seconds. On **both** Macs, create the file:

```bash
cat > ~/Library/LaunchAgents/com.claude.cowork-sync.plist << 'PLIST'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.claude.cowork-sync</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>$HOME/.claude-sync/cowork-sync.sh</string>
    </array>
    <key>StartInterval</key>
    <integer>60</integer>
    <key>RunAtLoad</key>
    <true/>
    <key>StandardOutPath</key>
    <string>$HOME/.claude-sync/sync.log</string>
    <key>StandardErrorPath</key>
    <string>$HOME/.claude-sync/sync-stderr.log</string>
</dict>
</plist>
PLIST
```

> **Note:** Replace `$HOME` with your actual home directory path (e.g., `/Users/yourusername`) in the plist file — launchd doesn't expand environment variables.

**Load the daemon:**
```bash
launchctl load ~/Library/LaunchAgents/com.claude.cowork-sync.plist
```

---

## Phase 5: Configure the Always-On Mac (~5 min)

On the Mac that will stay home (your "second brain"):

### Prevent Sleep
System Settings > Battery > Options > **"Prevent automatic sleeping when display is off"** → ON

### Disable Auto-Install Updates
System Settings > General > Software Update > Automatic Updates > **"Install macOS updates"** → OFF

(Downloads stay on — updates will be ready when you choose to install them manually.)

### Physical Setup
- Keep the lid open
- Plug in to power
- Display sleep is fine (set to 5 min — the machine stays awake)

---

## Phase 6: Set Up Your CLAUDE.md (~10 min)

Open Claude Cowork on Mac 1 and say:

> "Help me set up my CLAUDE.md. I want sections for: About Me, Preferences, my work context, personal projects, people I work with, a glossary of terms, and a decision log."

Tell Claude about yourself, your projects, your contacts, and how you like things done. The more context you add, the better Claude becomes.

**Recommended structure:**
1. About Me
2. Preferences (tone, format, deliverable style)
3. Work (company, role, products)
4. Personal Projects
5. People (contacts with role/context)
6. Terms & Glossary
7. Capabilities (plugins installed)
8. Dual-MacBook Setup (sync info)
9. Scheduled Tasks
10. Decision Log
11. Lessons Learned

---

## Phase 7: Test the Sync (~5 min)

### On Mac 1:
```bash
# Check daemon is running
launchctl list | grep cowork-sync

# Check sync log
tail -10 ~/.claude-sync/sync.log
```

### Test cross-machine sync:
1. Open Cowork on Mac 1 and say: "Add a test note to CLAUDE.md: SYNC TEST"
2. Wait 60 seconds
3. Open Cowork on Mac 2 and ask: "Do you see a SYNC TEST note?"
4. If yes — you're syncing! Remove the test note.

### If it doesn't work:
- Check error log: `cat ~/.claude-sync/sync-stderr.log`
- Most common issue: bash needs Full Disk Access (Phase 1.2)
- Force sync: `bash ~/.claude-sync/cowork-sync.sh`

---

## Phase 8: Install Plugins (~5 min)

Install the same plugins on both Macs. In Cowork, browse available plugins or ask Claude: "What plugins are available for marketing?" (or sales, data, legal, etc.)

**Remember:** Plugins are per-machine. Install them on both Macs separately.

**Same for connectors** (Gmail, Calendar, Notion, etc.): authenticate on each Mac independently.

---

## Phase 9: Create Scheduled Tasks

On your always-on Mac, create your first scheduled task:

> "Create a scheduled task called 'morning-briefing' that runs daily at 7am. It should check my calendar for today's events, summarize any urgent emails, and list my top 3 priorities from TASKS.md."

See the [Playbook](playbook.html) for more scheduled task ideas.

---

## You're Done!

Your second brain is now running. Here's your daily workflow:

1. **Morning:** Review overnight outputs on Mac 1, kick off heavy work
2. **Midday:** Use Mac 2 on the go — same context, pick up where you left off
3. **Evening:** Save context, queue overnight work on Mac 1

For the full guide — workflows, recipes, tips, and troubleshooting — open the [Playbook](playbook.html).

---

## Quick Reference

```bash
# Daemon status
launchctl list | grep cowork-sync

# Recent sync activity
tail -20 ~/.claude-sync/sync.log

# Force sync
bash ~/.claude-sync/cowork-sync.sh

# Restart daemon
launchctl unload ~/Library/LaunchAgents/com.claude.cowork-sync.plist && \
launchctl load ~/Library/LaunchAgents/com.claude.cowork-sync.plist

# Check for errors
cat ~/.claude-sync/sync-stderr.log
```
