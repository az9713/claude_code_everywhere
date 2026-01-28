# tmux Tutorial for Absolute Beginners

## What You'll Learn

By the end of this tutorial, you'll understand:
- What tmux is and why it's incredibly useful
- How to create, detach, and reattach to sessions
- How to keep programs running even when you disconnect
- Essential tmux commands you'll use every day

---

## Part 1: What is tmux?

### The Simple Explanation

**tmux** stands for **T**erminal **Mu**ltiple**x**er.

In plain English: **tmux keeps your terminal sessions alive even when you close the window or disconnect.**

### The Problem tmux Solves

Imagine this scenario:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        THE PROBLEM: WITHOUT TMUX                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   SCENARIO: You're running a long process on a remote server                │
│                                                                              │
│   ┌─────────────────┐                         ┌─────────────────┐           │
│   │ Your Computer   │        SSH              │ Remote Server   │           │
│   │                 │ ═══════════════════════ │                 │           │
│   │ Running Claude  │       Connection        │ Claude Code     │           │
│   │ Code remotely   │                         │ doing work...   │           │
│   └─────────────────┘                         └─────────────────┘           │
│                                                                              │
│   THEN: Your WiFi drops for 5 seconds, or you close your laptop...         │
│                                                                              │
│   ┌─────────────────┐                         ┌─────────────────┐           │
│   │ Your Computer   │        💥               │ Remote Server   │           │
│   │                 │     Connection          │                 │           │
│   │  😱 Oh no!      │        LOST!            │  💀 Process     │           │
│   │                 │                         │     KILLED!     │           │
│   └─────────────────┘                         └─────────────────┘           │
│                                                                              │
│   Everything you were working on is GONE.                                   │
│   Claude Code stops mid-task. You have to start over.                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

Now with tmux:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        THE SOLUTION: WITH TMUX                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────────┐                         ┌─────────────────┐           │
│   │ Your Computer   │        SSH              │ Remote Server   │           │
│   │                 │ ═══════════════════════ │ ┌─────────────┐ │           │
│   │ Connected to    │       Connection        │ │   TMUX      │ │           │
│   │ tmux session    │                         │ │ ┌─────────┐ │ │           │
│   │                 │                         │ │ │ Claude  │ │ │           │
│   └─────────────────┘                         │ │ │ Code    │ │ │           │
│                                               │ │ └─────────┘ │ │           │
│                                               │ └─────────────┘ │           │
│   THEN: WiFi drops, you close laptop...      └─────────────────┘           │
│                                                                              │
│   ┌─────────────────┐                         ┌─────────────────┐           │
│   │ Your Computer   │        💥               │ Remote Server   │           │
│   │                 │     Connection          │ ┌─────────────┐ │           │
│   │  🤷 Oh well,    │        LOST!            │ │   TMUX      │ │           │
│   │  I'll reconnect │                         │ │ ┌─────────┐ │ │ ◄── Still │
│   │  later          │                         │ │ │ Claude  │ │ │    running!
│   └─────────────────┘                         │ │ │ Code    │ │ │           │
│                                               │ │ └─────────┘ │ │           │
│   LATER: You reconnect...                     │ └─────────────┘ │           │
│                                               └─────────────────┘           │
│   ┌─────────────────┐                         ┌─────────────────┐           │
│   │ Your Computer   │        SSH              │ Remote Server   │           │
│   │                 │ ═══════════════════════ │ ┌─────────────┐ │           │
│   │  😊 I'm back!   │    New Connection       │ │   TMUX      │ │           │
│   │  Right where    │                         │ │ ┌─────────┐ │ │           │
│   │  I left off!    │ ◄══════════════════════ │ │ │ Claude  │ │ │           │
│   │                 │    Reattach to same     │ │ │ Code    │ │ │           │
│   └─────────────────┘        session          │ │ └─────────┘ │ │           │
│                                               │ └─────────────┘ │           │
│                                               └─────────────────┘           │
│                                                                              │
│   Everything is exactly as you left it!                                     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Real-World Analogy

**Without tmux:** Like a phone call. When you hang up, the conversation ends.

**With tmux:** Like a group chat. You can close the app, and when you come back, all the messages are still there and the conversation continues.

---

## Part 2: Why Do You Need tmux?

### Reason 1: Sessions Survive Disconnection

```
REAL SCENARIO:
──────────────

You're on a train, using mobile data to work on your home PC via SSH.

WITHOUT TMUX:
│
├── Train goes through tunnel
├── Internet drops for 30 seconds
├── SSH connection dies
├── Your Claude Code session is GONE
├── You have to start over when internet returns
│
└── 😤 Frustrating!

WITH TMUX:
│
├── Train goes through tunnel
├── Internet drops for 30 seconds
├── SSH connection dies
├── But tmux keeps running on your PC!
├── When internet returns, you reconnect
├── tmux attach - you're back exactly where you were
│
└── 😊 No problem!
```

### Reason 2: Switch Devices Seamlessly

```
REAL SCENARIO:
──────────────

Morning: Working on your PC at your desk
│
├── Started a long Claude Code task
├── tmux is running
│
└── Need to leave for a meeting

Afternoon: On your phone with Termius
│
├── SSH to your PC
├── tmux attach
├── Continue exactly where you left off!
│
└── Finish the task on your phone

Evening: Back at your desk
│
├── tmux attach
├── See all the work done from your phone
│
└── Continue on PC

ALL THE SAME SESSION!
```

### Reason 3: Run Multiple Things at Once

```
ONE TMUX SESSION, MULTIPLE WINDOWS:
───────────────────────────────────

┌─────────────────────────────────────────────────────────────────┐
│  tmux session: "project"                                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Window 0: Claude Code                                         │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ Claude is helping you write code...                     │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   Window 1: Running tests                                       │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ npm test -- --watch                                      │   │
│   │ ✓ 47 tests passed                                       │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   Window 2: Server logs                                         │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ [INFO] Server running on port 3000                      │   │
│   │ [INFO] Request: GET /api/users                          │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   Switch between windows with Ctrl+B, then 0/1/2                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Part 3: Installing tmux

### On Ubuntu/Debian (including WSL)

Open your terminal and type:

```bash
sudo apt update
sudo apt install tmux
```

You'll be asked for your password. Type it (nothing shows as you type) and press Enter.

When asked "Do you want to continue? [Y/n]", type `y` and press Enter.

### On Mac

Using Homebrew:
```bash
brew install tmux
```

### On Other Linux Distributions

**Fedora/RHEL/CentOS:**
```bash
sudo dnf install tmux
```

**Arch Linux:**
```bash
sudo pacman -S tmux
```

### Verify Installation

Check that tmux is installed:

```bash
tmux -V
```

You should see something like:
```
tmux 3.3a
```

(The version number may be different)

---

## Part 4: Your First tmux Session

### Step 4.1: Starting tmux

In your terminal, type:

```bash
tmux
```

Press Enter.

**What you'll see:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│                                                                              │
│   Your normal terminal, but now...                                          │
│                                                                              │
│   username@computer:~$                                                       │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│[0] 0:bash*                                            "computer" 14:30 27-Jan│
└─────────────────────────────────────────────────────────────────────────────┘
                                    ▲
                                    │
                            This green bar at the bottom
                            means you're inside tmux!
```

### Step 4.2: Understanding the Status Bar

```
[0] 0:bash*                                            "computer" 14:30 27-Jan
 │   │  │                                                   │        │
 │   │  │                                                   │        └── Date/time
 │   │  │                                                   └── Your computer name
 │   │  └── The program running in this window (bash = your shell)
 │   └── Window number (0 is the first window)
 └── Session number/name (0 is the default)
```

### Step 4.3: Exiting tmux

To exit tmux completely (closes the session):

```bash
exit
```

The green bar disappears, and you're back to your normal terminal.

**But wait!** Exiting with `exit` actually CLOSES the session. That's usually not what you want. What you usually want is to DETACH...

---

## Part 5: The Magic - Detaching and Reattaching

### The Key Concept

- **Detach**: Disconnect from a session WITHOUT stopping it
- **Attach**: Reconnect to a running session

This is the core superpower of tmux!

### Step 5.1: The Prefix Key

tmux uses a special "prefix" key to know you want to send a command to tmux (not to the program running inside).

**The prefix is: `Ctrl + B`**

When you want to give tmux a command:
1. Press `Ctrl + B` (and release)
2. Then press the command key

It's like saying "Hey tmux, listen up!" before giving a command.

### Step 5.2: Detaching from a Session

Let's practice:

1. Start tmux:
   ```bash
   tmux
   ```

2. Run something so you can see it's working:
   ```bash
   top
   ```
   (This shows a live view of running processes)

3. Now DETACH:
   - Press `Ctrl + B`
   - Release both keys
   - Press `d`

4. You'll see:
   ```
   [detached (from session 0)]
   ```

You're back at your normal terminal. But `top` is STILL RUNNING inside tmux!

### Step 5.3: Reattaching to a Session

To reconnect:

```bash
tmux attach
```

**Amazing!** You're back, and `top` is still running!

Press `q` to quit `top`.

### Visual Explanation

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      DETACH AND REATTACH FLOW                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   STEP 1: You're in tmux, working                                           │
│   ────────────────────────────────                                          │
│                                                                              │
│   ┌─────────────────────────────────────┐                                   │
│   │         YOUR SCREEN                  │                                   │
│   │   ┌─────────────────────────────┐   │                                   │
│   │   │  TMUX SESSION               │   │                                   │
│   │   │  ┌─────────────────────┐    │   │                                   │
│   │   │  │ Claude Code running │    │   │                                   │
│   │   │  └─────────────────────┘    │   │                                   │
│   │   └─────────────────────────────┘   │                                   │
│   │   [claude] 0:bash                   │                                   │
│   └─────────────────────────────────────┘                                   │
│                                                                              │
│   STEP 2: You press Ctrl+B, then d (DETACH)                                 │
│   ─────────────────────────────────────────                                 │
│                                                                              │
│   ┌─────────────────────────────────────┐    ┌─────────────────────────┐    │
│   │         YOUR SCREEN                  │    │  STILL RUNNING!        │    │
│   │                                      │    │  (you just can't see)  │    │
│   │   [detached from session claude]    │    │  ┌─────────────────┐   │    │
│   │                                      │    │  │ Claude Code     │   │    │
│   │   username@computer:~$              │    │  │ still working!  │   │    │
│   │                                      │    │  └─────────────────┘   │    │
│   │   (back to normal terminal)         │    │                        │    │
│   └─────────────────────────────────────┘    └─────────────────────────┘    │
│                                                                              │
│   STEP 3: Later, you type: tmux attach (REATTACH)                           │
│   ───────────────────────────────────────────────                           │
│                                                                              │
│   ┌─────────────────────────────────────┐                                   │
│   │         YOUR SCREEN                  │                                   │
│   │   ┌─────────────────────────────┐   │                                   │
│   │   │  TMUX SESSION               │   │                                   │
│   │   │  ┌─────────────────────┐    │   │                                   │
│   │   │  │ Claude Code - still │    │   │                                   │
│   │   │  │ exactly where you   │    │   │                                   │
│   │   │  │ left it!            │    │   │                                   │
│   │   │  └─────────────────────┘    │   │                                   │
│   │   └─────────────────────────────┘   │                                   │
│   │   [claude] 0:bash                   │                                   │
│   └─────────────────────────────────────┘                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Part 6: Named Sessions

### Why Name Sessions?

Instead of sessions named "0", "1", etc., you can give them meaningful names:

```
tmux sessions:
├── claude    (for Claude Code)
├── project   (for a specific project)
└── server    (for running a server)
```

### Creating a Named Session

```bash
tmux new -s claude
```

This creates a new session named "claude".

The `-s` means "session name".

### Attaching to a Named Session

```bash
tmux attach -t claude
```

The `-t` means "target".

### Listing All Sessions

```bash
tmux ls
```

Example output:
```
claude: 1 windows (created Mon Jan 27 14:30:00 2026)
project: 2 windows (created Mon Jan 27 10:15:00 2026)
```

### The -d Flag for Multiple Devices

When you connect from another device, you want to "take over" the session:

```bash
tmux attach -t claude -d
```

The `-d` means "detach anyone else first".

This is important! Otherwise, you might have two devices fighting for the same session.

---

## Part 7: Essential tmux Commands

### Command Reference Card

All commands start with the prefix: `Ctrl + B`, then the command key.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         TMUX COMMAND REFERENCE                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   SESSION COMMANDS                                                           │
│   ────────────────                                                           │
│                                                                              │
│   Ctrl+B, then d     │  Detach from session (session keeps running)         │
│   Ctrl+B, then $     │  Rename current session                              │
│   Ctrl+B, then s     │  List and switch between sessions                    │
│                                                                              │
│   WINDOW COMMANDS (like browser tabs)                                        │
│   ────────────────────────────────────                                       │
│                                                                              │
│   Ctrl+B, then c     │  Create new window                                   │
│   Ctrl+B, then ,     │  Rename current window                               │
│   Ctrl+B, then n     │  Next window                                         │
│   Ctrl+B, then p     │  Previous window                                     │
│   Ctrl+B, then 0-9   │  Go to window number 0-9                             │
│   Ctrl+B, then &     │  Close current window                                │
│                                                                              │
│   PANE COMMANDS (split screen)                                               │
│   ────────────────────────────                                               │
│                                                                              │
│   Ctrl+B, then %     │  Split vertically (left/right)                       │
│   Ctrl+B, then "     │  Split horizontally (top/bottom)                     │
│   Ctrl+B, then arrow │  Move between panes                                  │
│   Ctrl+B, then x     │  Close current pane                                  │
│                                                                              │
│   OTHER                                                                      │
│   ─────                                                                      │
│                                                                              │
│   Ctrl+B, then ?     │  Show all keyboard shortcuts                         │
│   Ctrl+B, then :     │  Enter command mode                                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Terminal Commands (Outside tmux)

```bash
# Create new named session
tmux new -s sessionname

# List all sessions
tmux ls

# Attach to a session
tmux attach -t sessionname

# Attach and detach others
tmux attach -t sessionname -d

# Kill a session
tmux kill-session -t sessionname

# Kill ALL sessions
tmux kill-server
```

---

## Part 8: Practical Workflows

### Workflow 1: Daily Claude Code Session

**Morning:**
```bash
# Start your Claude Code session
tmux new -s claude

# Inside tmux, navigate to your project
cd ~/projects/my-app

# Start Claude Code
claude

# Work on your project...
```

**When you need to leave:**
```
Press Ctrl+B, then d
```

You see: `[detached (from session claude)]`

**Later, when you return:**
```bash
tmux attach -t claude -d
```

You're back exactly where you left off!

### Workflow 2: Multiple Projects

```bash
# Session for main project
tmux new -s project1
# (work on project1)
# Ctrl+B, d to detach

# Session for side project
tmux new -s project2
# (work on project2)
# Ctrl+B, d to detach

# List all sessions
tmux ls
# Output:
# project1: 1 windows (created ...)
# project2: 1 windows (created ...)

# Switch between them
tmux attach -t project1
# (work)
# Ctrl+B, d

tmux attach -t project2
# (work)
```

### Workflow 3: Connecting from Phone

On your phone (via Termius):

```bash
# SSH to your PC
# You land in Windows, so first:
wsl

# Now attach to your session
tmux attach -t claude -d

# Work on your phone!
# When done:
# Ctrl+B, d (swipe left in Termius for special keys)
```

---

## Part 9: Understanding the tmux Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           TMUX HIERARCHY                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   TMUX SERVER                                                                │
│   ───────────                                                               │
│   │                                                                         │
│   ├── SESSION: "claude"                                                     │
│   │   │                                                                     │
│   │   ├── WINDOW 0: "main"                                                 │
│   │   │   │                                                                │
│   │   │   └── PANE (where Claude Code runs)                                │
│   │   │                                                                     │
│   │   └── WINDOW 1: "tests"                                                │
│   │       │                                                                │
│   │       └── PANE (where tests run)                                       │
│   │                                                                         │
│   └── SESSION: "server"                                                     │
│       │                                                                     │
│       └── WINDOW 0: "logs"                                                 │
│           │                                                                │
│           ├── PANE 1 (server logs)                                         │
│           │                                                                │
│           └── PANE 2 (monitoring)                                          │
│                                                                              │
│   ANALOGY:                                                                   │
│   ────────                                                                   │
│   Sessions = Different projects/contexts                                    │
│   Windows  = Like browser tabs                                              │
│   Panes    = Split screen within a window                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Part 10: Scrolling and Copy Mode

### The Problem

When you're in tmux and try to scroll with your mouse or trackpad, it doesn't work the normal way.

### The Solution: Copy Mode

To scroll in tmux:

1. Press `Ctrl+B`, then `[`

2. You're now in "copy mode"

3. Use arrow keys or Page Up/Down to scroll

4. Press `q` to exit copy mode

### Visual Guide

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          SCROLLING IN TMUX                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   STEP 1: Enter copy mode                                                   │
│           Press: Ctrl+B, then [                                             │
│                                                                              │
│   STEP 2: Navigate                                                          │
│           ↑ ↓     = Scroll line by line                                    │
│           PgUp    = Scroll page up                                          │
│           PgDown  = Scroll page down                                        │
│           g       = Go to top                                               │
│           G       = Go to bottom                                            │
│                                                                              │
│   STEP 3: Exit copy mode                                                    │
│           Press: q                                                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Enable Mouse Support (Optional)

You can enable mouse scrolling by adding this to your config file.

Create the file `~/.tmux.conf`:

```bash
nano ~/.tmux.conf
```

Add this line:
```
set -g mouse on
```

Save and exit (Ctrl+O, Enter, Ctrl+X).

Reload tmux configuration:
```bash
tmux source-file ~/.tmux.conf
```

Now you can scroll with your mouse!

---

## Part 11: Troubleshooting

### Problem: "no sessions" Error

```bash
$ tmux attach
no sessions
```

**Cause:** There are no running sessions.

**Solution:** Create a new session:
```bash
tmux new -s mysession
```

### Problem: "sessions should be nested with care"

```bash
$ tmux
sessions should be nested with care, unset $TMUX to force
```

**Cause:** You're already inside a tmux session.

**Solution:** You don't need to run tmux inside tmux. You're already in one!

### Problem: Session Exists but Can't Attach

```bash
$ tmux attach -t claude
can't find session: claude
```

**Cause:** Session name is wrong, or it doesn't exist.

**Solution:** List sessions to see what exists:
```bash
tmux ls
```

### Problem: Special Keys Don't Work (on Phone)

**Cause:** Mobile keyboards don't have Ctrl easily accessible.

**Solution:** In Termius, swipe left on the keyboard to reveal special keys including Ctrl.

### Problem: Prefix Key Doesn't Work

**Symptom:** Pressing Ctrl+B does nothing.

**Possible causes:**
1. You're not in tmux (check for green status bar)
2. The program you're running is capturing the key
3. Your terminal app is intercepting the key

**Solutions:**
1. Make sure you see the green status bar at the bottom
2. Try pressing Ctrl+B twice (sends Ctrl+B to the program)

---

## Part 12: Quick Reference

### Starting and Stopping

| What | Command |
|------|---------|
| Start new session | `tmux` |
| Start named session | `tmux new -s name` |
| List sessions | `tmux ls` |
| Attach to session | `tmux attach -t name` |
| Attach and kick others | `tmux attach -t name -d` |
| Kill session | `tmux kill-session -t name` |
| Kill all sessions | `tmux kill-server` |

### Inside tmux (Prefix = Ctrl+B)

| What | Keys |
|------|------|
| Detach | Prefix, then `d` |
| Rename session | Prefix, then `$` |
| New window | Prefix, then `c` |
| Next window | Prefix, then `n` |
| Previous window | Prefix, then `p` |
| Go to window N | Prefix, then `0-9` |
| Split vertical | Prefix, then `%` |
| Split horizontal | Prefix, then `"` |
| Switch pane | Prefix, then arrow key |
| Scroll mode | Prefix, then `[` |
| Exit scroll mode | `q` |
| Help | Prefix, then `?` |

---

## Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                      TMUX IN A NUTSHELL                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   WHAT:  Terminal multiplexer - keeps sessions alive            │
│                                                                  │
│   WHY:   • Sessions survive disconnection                       │
│          • Switch between devices seamlessly                    │
│          • Run multiple things in one session                   │
│          • Never lose work to dropped connections               │
│                                                                  │
│   HOW:   tmux new -s name    (start session)                    │
│          Ctrl+B, d           (detach)                           │
│          tmux attach -t name (reattach)                         │
│                                                                  │
│   REMEMBER:                                                      │
│          • Green bar = you're in tmux                           │
│          • Detach ≠ Exit (detach keeps it running!)             │
│          • Use -d flag when attaching from another device       │
│          • Sessions persist until you kill them                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## What's Next?

Now that you understand tmux:
- **SSH**: The secure way to connect to remote computers
- **Tailscale**: Fixed addresses for your devices, anywhere
- **Termius**: A great mobile app for SSH + tmux

With tmux as your foundation, you can work from anywhere without fear of losing your session!
