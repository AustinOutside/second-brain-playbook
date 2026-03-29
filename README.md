# The Claude Second Brain

Turn Claude into a 24/7 personal assistant with a dual-Mac setup, persistent memory, and automated workflows.

## What Is This?

A framework for running Claude Cowork (Anthropic's desktop AI) as an always-on "second brain" — one Mac stays home running scheduled tasks and long jobs, while the other travels with you. A background sync daemon keeps Claude's memory consistent across both machines through iCloud Drive.

**The result:** Claude remembers everything about you, your projects, your people, and your preferences — across every session, on every device. You can start a research task at home, head out, and review the results from a coffee shop.

## What's Inside

| File | What It Is |
|------|-----------|
| [**Playbook**](playbook.html) | The complete guide — architecture, workflows, 18 ready-to-use recipes, 14 power tips, plugin guide, troubleshooting |
| [**Getting Started**](getting-started.md) | Step-by-step setup instructions for the dual-Mac sync |
| [**Sync Script**](sync-script.md) | The daemon script that makes it all work |

## How It Works

```
┌─────────────────────┐         ┌─────────────────────┐
│  Mac 1 (Second Brain) │         │  Mac 2 (Daily Driver) │
│                     │         │                     │
│  - Always-on at home│         │  - Portable         │
│  - Scheduled tasks  │         │  - Interactive work  │
│  - Long research    │  iCloud │  - Content creation  │
│  - Overnight jobs   │◄──────►│  - Meetings & calls  │
│                     │  sync   │                     │
│  CLAUDE.md ──────────┼────────┼── CLAUDE.md          │
│  TASKS.md  ──────────┼────────┼── TASKS.md           │
│  memory/   ──────────┼────────┼── memory/            │
└─────────────────────┘  60s    └─────────────────────┘
```

A launchd daemon runs every 60 seconds on both Macs, syncing Claude's context files bidirectionally through iCloud Drive using MD5 hash-based change detection.

## Key Features

- **Persistent memory** — Claude remembers your people, projects, preferences, and decisions across every session
- **Cross-device sync** — Start work on one Mac, pick it up on the other
- **Scheduled tasks** — Morning briefings, job scanners, website monitoring — all automated
- **18 ready-to-use recipes** — Copy-paste prompts for research, content, email, trip planning, and more
- **Plugin-powered workflows** — Marketing, sales, data analysis, legal, finance, and more
- **Custom skills** — Turn repeating workflows into one-command shortcuts

## Requirements

- Two Macs with iCloud Drive (same Apple ID)
- [Claude Desktop app](https://claude.ai/download) installed on both
- macOS 13+ (Ventura or later)

## Quick Start

1. Read the [Getting Started](getting-started.md) guide
2. Set up the [Sync Script](sync-script.md) on both Macs
3. Open the [Playbook](playbook.html) for workflows, recipes, and tips

## Built By

[Austin Virts](https://austinvirts.com) — March 2026

---

*This playbook was itself built using the second brain setup it describes. Meta.*
