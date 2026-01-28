# Documentation Index

## Remote Terminal Access - Access Claude Code From Anywhere

This documentation teaches you how to access terminal-based AI coding agents (Claude Code, Codex CLI, Gemini CLI) from any device, anywhere in the world.

---

## Quick Start

**New here?** Start with the main guide, then dive into individual tutorials as needed.

| Your Goal | Start Here |
|-----------|------------|
| Set up everything from scratch | [Complete Setup Guide](#complete-setup-guide) |
| Understand a specific tool | [Individual Tutorials](#individual-tutorials) |
| Quick reference while working | [Quick Reference Card](#quick-reference-card) |

---

## Complete Setup Guide

### [Remote Terminal Setup Guide](remote-terminal-setup.md)

The comprehensive, step-by-step guide that walks you through the entire setup process.

**What's Inside:**
- System architecture diagrams
- Communication flow visualizations
- Prerequisites checklist
- Complete installation steps for all components
- Daily workflow examples
- Troubleshooting guide

**Time to complete:** 1-2 hours

---

## Individual Tutorials

Learn each component in depth. Perfect for understanding the "what, why, and how" of each tool.

### [SSH Tutorial](tutorial-ssh.md)

**Secure Shell - The Foundation of Remote Access**

```
Difficulty: Beginner
Reading time: 20 minutes
```

Learn how SSH creates secure, encrypted connections between computers.

**Topics covered:**
- What SSH is and why it exists
- How encryption keeps your connection safe
- Setting up SSH server on Windows
- Common SSH commands
- Security best practices
- Troubleshooting connection issues

---

### [Tailscale Tutorial](tutorial-tailscale.md)

**Your Private Network, Anywhere**

```
Difficulty: Beginner
Reading time: 20 minutes
```

Learn how Tailscale creates a secure private network connecting all your devices.

**Topics covered:**
- What Tailscale is and the problems it solves
- How Tailscale works (WireGuard, Tailnet)
- Installation on Windows and iPhone
- Finding your Tailscale IP address
- Magic DNS for easy device names
- Security features

---

### [tmux Tutorial](tutorial-tmux.md)

**Terminal Sessions That Never Die**

```
Difficulty: Beginner
Reading time: 25 minutes
```

Learn how tmux keeps your terminal sessions alive even when you disconnect.

**Topics covered:**
- What tmux is and why it's essential
- Creating and naming sessions
- Detaching and reattaching (the core superpower)
- The prefix key (Ctrl+B) concept
- Windows and panes
- Scrolling and copy mode
- Complete command reference

---

### [Termius Tutorial](tutorial-termius.md)

**Your Computer's Terminal, On Your Phone**

```
Difficulty: Beginner
Reading time: 20 minutes
```

Learn how to use Termius to control your computer from your iPhone or Android.

**Topics covered:**
- What Termius is and why to use it
- Installation and setup
- Adding hosts (your computers)
- Using the mobile keyboard (Ctrl, Tab, arrows)
- Gestures and mobile tips
- Connecting to WSL and tmux
- Troubleshooting

---

## How the Pieces Fit Together

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        THE COMPLETE PICTURE                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   YOUR PHONE                           YOUR COMPUTER                         │
│   ──────────                           ─────────────                         │
│                                                                              │
│   ┌───────────────┐                   ┌───────────────────────────────────┐ │
│   │   Termius     │                   │            Windows                 │ │
│   │   (SSH app)   │                   │  ┌───────────────────────────────┐│ │
│   │               │                   │  │      WSL (Linux)              ││ │
│   └───────┬───────┘                   │  │  ┌───────────────────────────┐││ │
│           │                           │  │  │         tmux              │││ │
│           │                           │  │  │  ┌───────────────────────┐│││ │
│   ┌───────┴───────┐                   │  │  │  │     Claude Code      ││││ │
│   │   Tailscale   │═══════════════════│  │  │  │                      ││││ │
│   │   (VPN)       │  Secure Tunnel    │  │  │  └───────────────────────┘│││ │
│   └───────────────┘                   │  │  └───────────────────────────┘││ │
│                                       │  └───────────────────────────────┘│ │
│                                       │           Tailscale + SSH         │ │
│                                       └───────────────────────────────────┘ │
│                                                                              │
│   READING ORDER:                                                            │
│   1. SSH         - Understand secure connections                           │
│   2. Tailscale   - Set up your private network                            │
│   3. tmux        - Keep sessions alive                                     │
│   4. Termius     - Connect from your phone                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Quick Reference Card

Print this or save it on your phone for daily use:

```
═══════════════════════════════════════════════════════════════════
                    DAILY COMMANDS CHEAT SHEET
═══════════════════════════════════════════════════════════════════

ON YOUR PC - Starting a Session:
────────────────────────────────
wsl                              # Enter Linux
tmux new -s claude               # Start new tmux session
claude                           # Start Claude Code

ON YOUR PC - Leaving:
─────────────────────
Ctrl+B, then d                   # Detach (keeps running!)

FROM YOUR PHONE - Reconnecting:
───────────────────────────────
1. Open Termius
2. Tap your host
3. Type: wsl
4. Type: tmux attach -t claude -d

BACK ON YOUR PC:
────────────────
wsl                              # Enter Linux
tmux attach -t claude -d         # Take over session

TMUX ESSENTIALS:
────────────────
tmux ls                          # List sessions
tmux new -s NAME                 # New named session
tmux attach -t NAME -d           # Attach (kick others)
tmux kill-session -t NAME        # End session
Ctrl+B, d                        # Detach
Ctrl+B, [                        # Scroll mode (q to exit)

TAILSCALE:
──────────
tailscale ip -4                  # Show your IP
tailscale status                 # Check connection

═══════════════════════════════════════════════════════════════════
```

---

## Learning Path

### For Complete Beginners

Follow this order:

```
Week 1: Foundations
├── Read: SSH Tutorial
├── Practice: SSH to your own computer locally
└── Understand: How secure connections work

Week 1-2: Networking
├── Read: Tailscale Tutorial
├── Install: Tailscale on PC and phone
└── Test: Ping between devices

Week 2: Persistence
├── Read: tmux Tutorial
├── Practice: Create, detach, reattach sessions
└── Master: The Ctrl+B prefix

Week 2-3: Mobile Access
├── Read: Termius Tutorial
├── Setup: Add your PC as a host
└── Connect: Full workflow from phone

Week 3+: Daily Use
├── Use: The complete setup daily
├── Reference: Quick reference card
└── Troubleshoot: As issues arise
```

### For Those Who Just Want It Working

1. Follow [Remote Terminal Setup Guide](remote-terminal-setup.md) start to finish
2. Use the quick reference card for daily work
3. Read individual tutorials when you need deeper understanding

---

## Additional Resources

### Reference Documents

| File | Description |
|------|-------------|
| [sources.txt](sources.txt) | Original source URLs and references |
| [gemini3_summary.txt](gemini3_summary.txt) | AI-generated summary notes |
| [README_transcript.txt](README_transcript.txt) | Original project transcript |

### External Links

**Official Documentation:**
- [tmux Wiki](https://github.com/tmux/tmux/wiki)
- [Tailscale Docs](https://tailscale.com/kb/)
- [Termius Help](https://support.termius.com/)
- [OpenSSH Manual](https://www.openssh.com/manual.html)

**Helpful Tutorials:**
- [tmux Cheat Sheet](https://tmuxcheatsheet.com/)
- [SSH Tutorial for Beginners](https://www.ssh.com/academy/ssh)

---

## Troubleshooting Quick Links

| Problem | See |
|---------|-----|
| Can't connect via SSH | [SSH Tutorial - Troubleshooting](tutorial-ssh.md#part-10-troubleshooting) |
| Tailscale devices can't see each other | [Tailscale Tutorial - Troubleshooting](tutorial-tailscale.md#part-10-troubleshooting) |
| tmux session not found | [tmux Tutorial - Troubleshooting](tutorial-tmux.md#part-11-troubleshooting) |
| Termius connection issues | [Termius Tutorial - Troubleshooting](tutorial-termius.md#part-11-troubleshooting) |
| General setup issues | [Setup Guide - Troubleshooting](remote-terminal-setup.md#troubleshooting-reference) |

---

## Document Information

| Document | Last Updated |
|----------|--------------|
| This Index | January 2026 |
| Remote Terminal Setup Guide | January 2026 |
| SSH Tutorial | January 2026 |
| Tailscale Tutorial | January 2026 |
| tmux Tutorial | January 2026 |
| Termius Tutorial | January 2026 |

---

## Contributing

Found an error or want to improve these docs? The source files are in the `docs/` directory.

---

## Acknowledgements

This project was inspired by the YouTube video:

**["Keep Using Your Local Claude Code Session from Any Device | Huge Unlock!"](https://www.youtube.com/watch?v=5dTjU2BPPis)**

All documentation and code in this repository were generated by **Claude Code** powered by **Claude Opus 4.5** (Anthropic).

---

*Happy remote coding!*
