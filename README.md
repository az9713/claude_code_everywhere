# Claude Code Everywhere

Access Claude Code (and other terminal-based AI agents) from any device, anywhere in the world.

```
┌─────────────────┐                              ┌─────────────────┐
│    Your PC      │                              │   Your Phone    │
│                 │      Secure Connection       │                 │
│  ┌───────────┐  │◄════════════════════════════►│  ┌───────────┐  │
│  │  Claude   │  │       (Tailscale VPN)        │  │  Termius  │  │
│  │   Code    │  │                              │  │   App     │  │
│  └───────────┘  │                              │  └───────────┘  │
│                 │                              │                 │
└─────────────────┘                              └─────────────────┘

Start on your PC → Leave → Continue on your phone → Return to PC
              Same session. Same context. No interruption.
```

## The Problem

You're working with Claude Code on a complex task. You need to leave your desk. Normally, you'd lose your entire session - the context, the conversation, everything.

## The Solution

This guide shows you how to set up a system where:

- **Sessions persist** - Close your laptop, your session keeps running
- **Access anywhere** - Connect from your phone, tablet, or another computer
- **Seamlessly switch** - Pick up exactly where you left off on any device
- **Stay secure** - Everything is encrypted end-to-end

## How It Works

```
┌──────────────────────────────────────────────────────────────────┐
│  Your Windows PC                                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  WSL (Linux)                                                │  │
│  │  ┌──────────────────────────────────────────────────────┐  │  │
│  │  │  tmux (keeps sessions alive)                         │  │  │
│  │  │  ┌────────────────────────────────────────────────┐  │  │  │
│  │  │  │  Claude Code (your AI assistant)               │  │  │  │
│  │  │  └────────────────────────────────────────────────┘  │  │  │
│  │  └──────────────────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────────┘  │
│                              ▲                                    │
│                              │ SSH                                │
│                              │                                    │
│  ┌───────────────────────────┴────────────────────────────────┐  │
│  │  Tailscale (secure private network)                        │  │
│  └────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
                               ║
                               ║ Encrypted tunnel
                               ║ (works through any firewall)
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│  Your iPhone                                                      │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  Termius (SSH client) + Tailscale                          │  │
│  └────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

## Documentation

### [📚 Full Documentation](docs/index.md)

Start here for the complete guide and all tutorials.

### Quick Links

| Guide | Description |
|-------|-------------|
| [Complete Setup Guide](docs/remote-terminal-setup.md) | Step-by-step setup with diagrams |
| [SSH Tutorial](docs/tutorial-ssh.md) | Understanding secure connections |
| [Tailscale Tutorial](docs/tutorial-tailscale.md) | Setting up your private network |
| [tmux Tutorial](docs/tutorial-tmux.md) | Keeping sessions alive |
| [Termius Tutorial](docs/tutorial-termius.md) | Connecting from your phone |

## Quick Start

### On Your PC

```bash
# Enter Linux (WSL)
wsl

# Start a tmux session
tmux new -s claude

# Launch Claude Code
claude

# Work on your project...

# When leaving: Press Ctrl+B, then d (detaches, keeps running)
```

### From Your Phone

1. Open Termius
2. Connect to your PC (via Tailscale IP)
3. Type: `wsl`
4. Type: `tmux attach -t claude -d`
5. You're back in your session!

## Requirements

- Windows 10/11 PC (with WSL)
- iPhone or Android phone
- Free accounts: Tailscale, Termius (optional)

## What You'll Install

| Tool | Purpose | Cost |
|------|---------|------|
| **WSL** | Run Linux on Windows | Free (built into Windows) |
| **tmux** | Keep sessions alive | Free |
| **Tailscale** | Secure private network | Free tier available |
| **Termius** | Mobile SSH client | Free tier available |

## Use Cases

- **Commuting** - Continue coding on the train
- **Meetings** - Step away, come back to same session
- **Travel** - Full access from hotel WiFi
- **Multi-device** - Seamlessly switch between devices
- **Long tasks** - Let Claude work while you're away, check progress from phone

## Security

- **End-to-end encryption** via Tailscale (WireGuard)
- **No public exposure** - Your PC isn't visible on the internet
- **Private network** - Only your devices can connect
- **SSH authentication** - Password or key-based

## Acknowledgements

This project was inspired by the YouTube video:

**["Keep Using Your Local Claude Code Session from Any Device | Huge Unlock!"](https://www.youtube.com/watch?v=5dTjU2BPPis)**

## Generated By

All documentation in this repository was generated by **Claude Code** powered by **Claude Opus 4.5** (Anthropic).

---

## License

MIT License - Feel free to use, modify, and share.

---

*Access your AI coding assistant from anywhere. Never lose a session again.*
