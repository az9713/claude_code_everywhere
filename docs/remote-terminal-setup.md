# Remote Terminal Agent Access Setup Guide

## For Absolute Beginners - Every Step Explained

---

## What Are We Building?

Imagine you're working with Claude Code on your Windows laptop at home. You start a conversation, Claude is helping you build something, and then... you need to leave. Normally, closing your laptop means losing that entire session.

**This guide solves that problem.**

After following this guide, you will be able to:
1. Start a Claude Code session on your Windows PC
2. Walk away (close the laptop, turn off the monitor, whatever)
3. Pick up your iPhone, connect from anywhere in the world
4. Continue EXACTLY where you left off - same conversation, same context, everything

**Real example:** You're at your desk, Claude Code is halfway through refactoring your project. You need to catch a train. You grab your phone, connect while walking to the station, and keep working. Claude doesn't even know you switched devices.

---

## System Architecture Overview

Before we install anything, let's see the big picture of what we're building.

### The Complete Setup

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           YOUR WINDOWS PC                                    │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                              WINDOWS                                   │  │
│  │                                                                        │  │
│  │   ┌─────────────┐     ┌─────────────────────────────────────────┐     │  │
│  │   │  Tailscale  │     │            WSL (Linux)                  │     │  │
│  │   │   Service   │     │  ┌───────────────────────────────────┐  │     │  │
│  │   │             │     │  │             tmux                   │  │     │  │
│  │   │ Assigns IP: │     │  │  ┌─────────────────────────────┐  │  │     │  │
│  │   │ 100.x.x.x   │     │  │  │                             │  │  │     │  │
│  │   │             │     │  │  │       Claude Code           │  │  │     │  │
│  │   │ Encrypts    │     │  │  │                             │  │  │     │  │
│  │   │ all traffic │     │  │  │   (Your AI Assistant)       │  │  │     │  │
│  │   │             │     │  │  │                             │  │  │     │  │
│  │   └─────────────┘     │  │  └─────────────────────────────┘  │  │     │  │
│  │                       │  │   Keeps session alive even when   │  │     │  │
│  │   ┌─────────────┐     │  │   you disconnect!                 │  │     │  │
│  │   │ SSH Server  │◄────┼──┴───────────────────────────────────┘  │     │  │
│  │   │ (OpenSSH)   │     │                                         │     │  │
│  │   │ Port 22     │     └─────────────────────────────────────────┘     │  │
│  │   └─────────────┘                                                      │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
         ▲
         │
         │  Encrypted tunnel through the internet
         │  (Tailscale VPN - like a private subway)
         │
         ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              YOUR iPHONE                                     │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                                                                        │  │
│  │   ┌─────────────┐         ┌─────────────────────────────────────┐     │  │
│  │   │  Tailscale  │         │            Termius                   │     │  │
│  │   │    App      │────────►│         (SSH Client)                 │     │  │
│  │   │             │         │                                      │     │  │
│  │   │ Connects to │         │  Shows your PC's terminal on        │     │  │
│  │   │ same network│         │  your phone screen. You type        │     │  │
│  │   │ as your PC  │         │  commands, they run on your PC.     │     │  │
│  │   │             │         │                                      │     │  │
│  │   └─────────────┘         └─────────────────────────────────────┘     │  │
│  │                                                                        │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘

                    ┌─────────────────────────────────┐
                    │         SLACK (Optional)         │
                    │                                  │
                    │  Receives notifications when     │
                    │  Claude Code needs attention     │
                    │                                  │
                    └─────────────────────────────────┘
```

### How the Layers Stack Up

Think of it like a set of Russian nesting dolls - each layer lives inside the next:

```
┌──────────────────────────────────────────────────────────────────┐
│  LAYER 1: Windows PC (The physical computer)                     │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  LAYER 2: WSL - Ubuntu (Linux running inside Windows)      │  │
│  │  ┌──────────────────────────────────────────────────────┐  │  │
│  │  │  LAYER 3: tmux session (Keeps things running)        │  │  │
│  │  │  ┌────────────────────────────────────────────────┐  │  │  │
│  │  │  │  LAYER 4: Claude Code (Your AI assistant)      │  │  │  │
│  │  │  │                                                 │  │  │  │
│  │  │  │  This is where the magic happens!              │  │  │  │
│  │  │  │                                                 │  │  │  │
│  │  │  └────────────────────────────────────────────────┘  │  │  │
│  │  └──────────────────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Communication Flow Diagrams

### Flow 1: Starting a Session (On Your PC)

This is what happens when you first start working:

```
    YOU AT YOUR PC
         │
         │ 1. Open Ubuntu terminal
         ▼
    ┌─────────────┐
    │   Ubuntu    │
    │   (WSL)     │
    └──────┬──────┘
           │ 2. Type: tmux new -s claude
           ▼
    ┌─────────────┐
    │    tmux     │
    │  session    │
    │  "claude"   │
    └──────┬──────┘
           │ 3. Type: claude
           ▼
    ┌─────────────┐
    │   Claude    │
    │    Code     │  ◄── Now running and ready!
    │             │
    └─────────────┘
```

### Flow 2: Detaching (Leaving Your PC)

What happens when you walk away:

```
    YOU AT YOUR PC
    (in Claude Code)
           │
           │ 1. Press Ctrl+B, then D
           ▼
    ┌─────────────────────────────────────────────────────┐
    │                                                      │
    │   You see: [detached (from session claude)]         │
    │                                                      │
    └─────────────────────────────────────────────────────┘
           │
           │ 2. Close laptop, walk away, go to bed...
           ▼
    ┌─────────────────────────────────────────────────────┐
    │                                                      │
    │   Meanwhile, on your PC:                            │
    │                                                      │
    │   ┌───────────────────────────────────────────┐     │
    │   │  tmux session "claude" is STILL RUNNING   │     │
    │   │  ┌─────────────────────────────────────┐  │     │
    │   │  │  Claude Code is STILL ACTIVE        │  │     │
    │   │  │  (just waiting for you to return)   │  │     │
    │   │  └─────────────────────────────────────┘  │     │
    │   └───────────────────────────────────────────┘     │
    │                                                      │
    └─────────────────────────────────────────────────────┘
```

### Flow 3: Connecting from iPhone

The complete journey when you connect from your phone:

```
    YOU WITH YOUR iPHONE
    (anywhere in the world)
           │
           │ 1. Open Tailscale app (if not already on)
           ▼
    ┌─────────────┐
    │  Tailscale  │
    │    App      │──── Creates secure tunnel to your PC
    └──────┬──────┘
           │
           │ 2. Open Termius app
           ▼
    ┌─────────────┐         ┌─────────────┐
    │   Termius   │────────►│  Your PC's  │
    │  (iPhone)   │   SSH   │  Tailscale  │
    │             │ through │     IP      │
    │             │ tunnel  │ 100.x.x.x   │
    └──────┬──────┘         └──────┬──────┘
           │                       │
           │ 3. You see Windows    │
           │    command prompt     │
           ▼                       ▼
    ┌─────────────────────────────────────────────────────┐
    │  C:\Users\YourName>                                 │
    │                                                      │
    │  You type: wsl                                      │
    │                                                      │
    └──────────────────────────────────────────────────────┘
           │
           │ 4. Now you're in Linux
           ▼
    ┌─────────────────────────────────────────────────────┐
    │  username@COMPUTERNAME:~$                           │
    │                                                      │
    │  You type: tmux attach -t claude -d                 │
    │                                                      │
    └──────────────────────────────────────────────────────┘
           │
           │ 5. You're reconnected to Claude Code!
           ▼
    ┌─────────────────────────────────────────────────────┐
    │                                                      │
    │  ┌─────────────────────────────────────────────┐    │
    │  │                                              │    │
    │  │         Claude Code                         │    │
    │  │                                              │    │
    │  │   Exactly where you left off!               │    │
    │  │   Same conversation, same context           │    │
    │  │                                              │    │
    │  └─────────────────────────────────────────────┘    │
    │                                                      │
    └─────────────────────────────────────────────────────┘
```

### Flow 4: Complete Round-Trip (PC → Phone → PC)

Here's a day in the life:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│   MORNING: At your desk                                                     │
│   ═══════════════════                                                       │
│                                                                              │
│   ┌─────────┐  start   ┌─────────┐  start   ┌─────────┐                    │
│   │   You   │─────────►│  tmux   │─────────►│ Claude  │                    │
│   │  (PC)   │ session  │         │  claude  │  Code   │                    │
│   └─────────┘          └─────────┘          └─────────┘                    │
│                                                   │                         │
│   Work for a few hours...                        │                         │
│                                                   ▼                         │
│   ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ │
│                                                                              │
│   LUNCH: Need to leave                                                      │
│   ════════════════════                                                      │
│                                                                              │
│   ┌─────────┐ Ctrl+B,D ┌─────────┐  (keeps   ┌─────────┐                   │
│   │   You   │─────────►│  tmux   │  running) │ Claude  │                   │
│   │  (PC)   │ DETACH   │         │◄─────────►│  Code   │                   │
│   └─────────┘          └─────────┘           └─────────┘                   │
│        │                                          ▲                         │
│        │ Close laptop, leave                      │ Still alive!            │
│        ▼                                          │                         │
│   ┌─────────┐                                     │                         │
│   │  Gone!  │                                     │                         │
│   └─────────┘                                     │                         │
│                                                   │                         │
│   ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│
│                                                   │                         │
│   ON THE TRAIN: Connect from phone               │                         │
│   ════════════════════════════════               │                         │
│                                                   │                         │
│   ┌─────────┐ Tailscale ┌─────────┐             │                         │
│   │  You    │──────────►│  Your   │             │                         │
│   │(iPhone) │  tunnel   │   PC    │             │                         │
│   └────┬────┘           └────┬────┘             │                         │
│        │                     │                   │                         │
│        │ Termius SSH         │                   │                         │
│        │ connection          │                   │                         │
│        └─────────────────────┘                   │                         │
│                     │                            │                         │
│                     │ tmux attach -t claude -d   │                         │
│                     └────────────────────────────┘                         │
│                                                   │                         │
│   ┌─────────┐          ┌─────────┐          ┌────┴────┐                    │
│   │  You    │◄────────►│  tmux   │◄────────►│ Claude  │                    │
│   │(iPhone) │ attached │         │          │  Code   │                    │
│   └─────────┘          └─────────┘          └─────────┘                    │
│                                                                              │
│   Continue working on the train!                                            │
│                                                                              │
│   ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ │
│                                                                              │
│   EVENING: Back at your desk                                                │
│   ══════════════════════════                                                │
│                                                                              │
│   ┌─────────┐ tmux attach ┌─────────┐         ┌─────────┐                  │
│   │  You    │────────────►│  tmux   │◄───────►│ Claude  │                  │
│   │  (PC)   │ -t claude -d│         │         │  Code   │                  │
│   └─────────┘             └─────────┘         └─────────┘                  │
│        ▲                       │                                            │
│        │                       │                                            │
│        │              ┌────────┴────────┐                                   │
│        │              │ iPhone gets     │                                   │
│        │              │ disconnected    │                                   │
│        │              │ (the -d flag)   │                                   │
│        │              └─────────────────┘                                   │
│        │                                                                     │
│   Continue where you left off on the train!                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Flow 5: Slack Notifications (Optional)

When Claude needs your attention:

```
    ┌─────────────────────────────────────────────────────┐
    │              YOUR WINDOWS PC                         │
    │                                                      │
    │    ┌───────────────────────────────────────────┐    │
    │    │  Claude Code stops and waits for input    │    │
    │    └───────────────────┬───────────────────────┘    │
    │                        │                             │
    │                        │ Triggers hook script        │
    │                        ▼                             │
    │    ┌───────────────────────────────────────────┐    │
    │    │  ~/.claude/hooks/stop.sh                  │    │
    │    │                                            │    │
    │    │  Sends HTTP request to Slack webhook      │    │
    │    └───────────────────┬───────────────────────┘    │
    │                        │                             │
    └────────────────────────┼─────────────────────────────┘
                             │
                             │ HTTPS request
                             ▼
    ┌─────────────────────────────────────────────────────┐
    │                    INTERNET                          │
    └───────────────────────┬─────────────────────────────┘
                            │
                            ▼
    ┌─────────────────────────────────────────────────────┐
    │                   SLACK SERVERS                      │
    │                                                      │
    │    Receives webhook, posts to your channel          │
    └───────────────────────┬─────────────────────────────┘
                            │
                            │ Push notification
                            ▼
    ┌─────────────────────────────────────────────────────┐
    │                                                      │
    │    YOUR PHONE                                        │
    │    ┌───────────────────────────────────────────┐    │
    │    │  🔔 Slack                                  │    │
    │    │  #claude: Claude needs your attention!    │    │
    │    └───────────────────────────────────────────┘    │
    │                                                      │
    │    You open Termius and reconnect!                  │
    │                                                      │
    └─────────────────────────────────────────────────────┘
```

---

## Network Diagram

Here's how the network connection works:

```
                              THE INTERNET
    ┌──────────────────────────────────────────────────────────────┐
    │                                                               │
    │     Coffee Shop WiFi    Hotel WiFi    Mobile Data    Home    │
    │           │                │              │           │       │
    └───────────┼────────────────┼──────────────┼───────────┼───────┘
                │                │              │           │
                └────────────────┴──────────────┴───────────┘
                                      │
                    ┌─────────────────┴─────────────────┐
                    │         TAILSCALE NETWORK         │
                    │    (Your private encrypted VPN)   │
                    │                                   │
                    │   Only YOUR devices can connect   │
                    │   Everything is encrypted         │
                    │   Works through any firewall      │
                    │                                   │
                    └─────────────────┬─────────────────┘
                                      │
                 ┌────────────────────┼────────────────────┐
                 │                    │                    │
                 ▼                    ▼                    ▼
    ┌────────────────────┐ ┌────────────────────┐ ┌────────────────────┐
    │   Your Windows PC  │ │    Your iPhone     │ │   (Future devices) │
    │                    │ │                    │ │                    │
    │   IP: 100.64.1.10  │ │   IP: 100.64.1.20  │ │   iPad, MacBook,   │
    │                    │ │                    │ │   another PC...    │
    │   Runs Claude Code │ │   Runs Termius     │ │                    │
    └────────────────────┘ └────────────────────┘ └────────────────────┘

    ═══════════════════════════════════════════════════════════════════

    WHAT THIS MEANS:

    ✓ Your iPhone can ALWAYS reach your PC at 100.64.1.10
    ✓ Doesn't matter if you're at home, at a coffee shop, or overseas
    ✓ All traffic is encrypted end-to-end
    ✓ Your PC is NOT exposed to the public internet
    ✓ Only devices logged into YOUR Tailscale account can connect
```

---

## The Three Tools We'll Install

Before we dive in, let's understand what each tool does:

### 1. tmux (Terminal Multiplexer)
**What it is:** A program that keeps your terminal sessions alive in the background.

**Analogy:** Imagine you're reading a book. Normally, closing the book loses your place. tmux is like a magical bookmark that not only saves your place, but keeps the story playing even when the book is closed. When you open it again, you're exactly where you were.

**Why we need it:** Without tmux, closing your terminal kills Claude Code. With tmux, Claude Code keeps running even when you disconnect.

### 2. Tailscale (Secure VPN Mesh Network)
**What it is:** A service that creates a private, encrypted network between your devices.

**Analogy:** Imagine your devices are houses in different cities. Normally, to visit another house, you'd have to navigate public streets (the internet) where anyone could follow you. Tailscale builds a private underground tunnel directly connecting your houses. Only you have the key.

**Why we need it:** Your iPhone needs to "find" your Windows PC securely. Tailscale gives your PC a special address that only your devices can access.

### 3. Termius (Mobile SSH Client)
**What it is:** An app that lets your iPhone connect to your PC's terminal.

**Analogy:** It's like a remote control for your computer's command line, but it works over the internet.

**Why we need it:** It's how you'll actually type commands on your PC from your iPhone.

---

## Prerequisites Checklist

Before starting, make sure you have:

- [ ] A Windows 10 or 11 computer (this will be your "host" - the machine running Claude Code)
- [ ] An iPhone (this will be your "client" - how you connect remotely)
- [ ] An email address (for creating accounts)
- [ ] About 1-2 hours of time (it's detailed, but worth it)
- [ ] Administrator access to your Windows PC (you'll need to install software)

---

## Part 1: Installing WSL (Windows Subsystem for Linux)

### What is WSL?

WSL lets you run Linux inside Windows. We need it because tmux is a Linux program that doesn't run natively on Windows.

**Think of it as:** A Windows computer that has a Linux computer living inside it.

### Step 1.1: Open PowerShell as Administrator

1. Look at the bottom-left of your screen. You should see a search bar or a Windows icon.

2. Click on the search bar (or press the Windows key on your keyboard)

3. Type exactly: `PowerShell`

4. You'll see "Windows PowerShell" appear in the search results

5. **IMPORTANT:** Don't just click on it! We need to run it as Administrator. Here's how:
   - Right-click on "Windows PowerShell"
   - Click "Run as administrator"

6. A dialog box will appear asking "Do you want to allow this app to make changes to your device?"
   - Click "Yes"

7. A blue window will open. This is PowerShell. You should see something like:
   ```
   Windows PowerShell
   Copyright (C) Microsoft Corporation. All rights reserved.

   PS C:\Windows\System32>
   ```

   The blinking cursor after `>` is where you type commands.

### Step 1.2: Install WSL

1. In the PowerShell window, carefully type this command exactly as shown:
   ```
   wsl --install
   ```

2. Press the Enter key

3. You'll see text scrolling by as Windows downloads and installs WSL. This may take 5-15 minutes depending on your internet speed.

4. When it's done, you'll see a message saying installation was successful and asking you to restart.

5. **RESTART YOUR COMPUTER NOW**
   - Click the Windows icon (bottom-left)
   - Click the power icon
   - Click "Restart"

### Step 1.3: Complete Ubuntu Setup

After your computer restarts:

1. A window will automatically open titled "Ubuntu". If it doesn't:
   - Click the Windows search bar
   - Type `Ubuntu`
   - Click on the Ubuntu app

2. Wait while it says "Installing, this may take a few minutes..."

3. You'll be asked to create a username:
   ```
   Enter new UNIX username:
   ```

   Type a simple username (lowercase letters only, no spaces). For example: `simon`
   Press Enter.

4. You'll be asked to create a password:
   ```
   New password:
   ```

   **IMPORTANT:** When you type your password, nothing will appear on screen. This is normal! It's a security feature. The computer IS recording what you type, it just doesn't show it.

   Type a password you'll remember. Press Enter.

5. You'll be asked to confirm:
   ```
   Retype new password:
   ```

   Type the same password again. Press Enter.

6. You should see a success message and then a prompt that looks like:
   ```
   username@COMPUTERNAME:~$
   ```

   **Congratulations! You now have Linux running inside Windows!**

### Step 1.4: Verify WSL is Working

Let's make sure everything is working:

1. In the Ubuntu window, type:
   ```
   echo "Hello from Linux!"
   ```

2. Press Enter

3. You should see:
   ```
   Hello from Linux!
   ```

If you see that, WSL is working perfectly.

---

## Part 2: Installing tmux

### Step 2.1: Update Linux Packages

Before installing new software on Linux, we should update the package list (like refreshing an app store):

1. In your Ubuntu window, type:
   ```
   sudo apt update
   ```

2. Press Enter

3. You'll be asked for your password (the one you just created). Type it and press Enter. Remember: nothing appears as you type - that's normal.

4. You'll see lots of text scroll by as it downloads package information. Wait for it to finish (you'll see the prompt `username@COMPUTERNAME:~$` again).

### Step 2.2: Install tmux

1. Type:
   ```
   sudo apt install tmux
   ```

2. Press Enter

3. You'll see a question:
   ```
   Do you want to continue? [Y/n]
   ```

4. Type `y` and press Enter

5. Wait for the installation to complete.

### Step 2.3: Verify tmux is Installed

1. Type:
   ```
   tmux -V
   ```

   (That's a capital V)

2. Press Enter

3. You should see something like:
   ```
   tmux 3.3a
   ```

   (The exact version number may differ)

**tmux is now installed!**

---

## Part 3: Learning tmux Basics

Before we continue with other installations, let's learn how tmux works. This is important - you'll use these commands every day.

### Step 3.1: Your First tmux Session

1. In your Ubuntu window, type:
   ```
   tmux
   ```

2. Press Enter

3. Your screen will change! You'll notice:
   - The bottom of the screen now has a green bar
   - The green bar shows something like `[0] 0:bash`

   **You are now INSIDE a tmux session!**

### Step 3.2: Understanding the Green Bar

The green bar at the bottom tells you:
- `[0]` - The session name (default is a number)
- `0:bash` - Window 0 running the bash shell
- On the right: the date and time

### Step 3.3: The Most Important Concept - Detaching

Here's the magic of tmux. You can "detach" from a session (disconnect from it) while it keeps running.

**Let's practice:**

1. First, let's run a simple command that we can see is "doing something":
   ```
   top
   ```

   Press Enter. You'll see a constantly updating display of your computer's processes.

2. Now we'll detach. Here's how:
   - Press and HOLD the `Ctrl` key
   - While holding Ctrl, press the `b` key once
   - Release both keys
   - Now press the `d` key (just `d`, not Ctrl+d)

3. You should see:
   ```
   [detached (from session 0)]
   ```

   And you're back at the regular terminal prompt!

**But here's the magic:** That `top` command is STILL RUNNING inside the tmux session, even though you can't see it.

### Step 3.4: Reattaching to a Session

Let's reconnect to see that `top` is still running:

1. Type:
   ```
   tmux attach
   ```

2. Press Enter

3. **Amazing!** You're back, and `top` is still running, showing live updates!

4. Press `q` to quit `top` (you'll see the regular prompt inside tmux)

### Step 3.5: Creating Named Sessions

For real work, we give sessions meaningful names. Let's create a properly named session:

1. First, let's exit the current unnamed session. Type:
   ```
   exit
   ```
   Press Enter. This closes the session completely.

2. Now create a named session:
   ```
   tmux new -s claude
   ```

   This creates a new session named "claude"

3. You're now in a tmux session named "claude" (check the green bar - it says `[claude]`)

### Step 3.6: Practice the Full Workflow

Let's practice what you'll do every day:

1. You're in the "claude" session. Detach:
   - Press `Ctrl+b`, then `d`

2. You should see:
   ```
   [detached (from session claude)]
   ```

3. List all sessions:
   ```
   tmux ls
   ```

   You'll see:
   ```
   claude: 1 windows (created [date/time])
   ```

4. Reattach:
   ```
   tmux attach -t claude
   ```

   (The `-t claude` means "target the session named claude")

5. You're back in!

### Step 3.7: The Detach Flag for Multiple Devices

When connecting from another device (like your iPhone), you want to "take over" the session. Use `-d`:

```
tmux attach -t claude -d
```

The `-d` means "detach anyone else who's connected to this session first, then attach me"

This is important! When you connect from your iPhone, it kicks off your PC. When you get back to your PC, you kick off your iPhone. Only one device controls the session at a time.

### tmux Command Summary

| What you want to do | Command |
|---------------------|---------|
| Start a new named session | `tmux new -s NAME` |
| List all sessions | `tmux ls` |
| Attach to a session | `tmux attach -t NAME` |
| Attach and kick others off | `tmux attach -t NAME -d` |
| Detach (while in tmux) | Press `Ctrl+b`, then `d` |
| Kill a session | `tmux kill-session -t NAME` |

---

## Part 4: Installing Tailscale on Windows

Now we'll set up the secure network that connects your devices.

### Step 4.1: Download Tailscale

1. Open your regular web browser (Chrome, Edge, Firefox, etc.)

2. Go to: https://tailscale.com/download/windows

3. Click the big "Download Tailscale for Windows" button

4. A file will download. It's probably in your Downloads folder with a name like `tailscale-setup-x.xx.x.exe`

### Step 4.2: Install Tailscale

1. Find the downloaded file and double-click it

2. If Windows asks "Do you want to allow this app to make changes to your device?" click "Yes"

3. The installer will run. Click through any prompts (default options are fine)

4. When it finishes, Tailscale is installed!

### Step 4.3: Start and Log Into Tailscale

1. Look at the bottom-right of your screen in the system tray (where the clock is). You might need to click the little "^" arrow to see hidden icons.

2. Find the Tailscale icon - it looks like a series of connected dots

3. Click on it

4. Click "Log in"

5. Your browser will open to a Tailscale login page

6. Choose how to log in:
   - **Google** - Use your Google account
   - **Microsoft** - Use your Microsoft/Outlook account
   - **GitHub** - Use your GitHub account
   - Or create a Tailscale account with email

   **Pick whichever you prefer. Remember which one you choose - you'll use the same method on your iPhone!**

7. Complete the login process in your browser

8. You'll see a success message. You can close the browser tab.

### Step 4.4: Verify Tailscale is Working

1. Click the Tailscale icon in the system tray again

2. You should see your machine name listed and it should say "Connected"

3. You'll also see an IP address like `100.x.x.x` - this is your Tailscale IP address. **Write this down or take a screenshot! You'll need it later.**

### Step 4.5: Find Your Tailscale IP Address

If you need to find your IP again later:

1. Click the Tailscale icon in system tray

2. Your IP appears next to your machine name

OR

1. Open PowerShell (you don't need admin this time, just regular PowerShell)

2. Type:
   ```
   tailscale ip -4
   ```

3. Press Enter

4. You'll see an IP like: `100.64.x.x`

**This IP address is how your iPhone will find your PC!**

---

## Part 5: Setting Up SSH on Windows

SSH (Secure Shell) is how your iPhone will actually connect to your PC's command line. We need to enable it on Windows.

### Step 5.1: Install OpenSSH Server

1. Click the Windows Start button (bottom-left)

2. Click the gear icon ⚙️ to open Settings

3. Click "Apps" (or "Apps & features")

4. Click "Optional features" (in Windows 11, you might need to scroll down to find it)

5. Click "Add a feature" (or "View features" then "Add")

6. In the search box, type: `OpenSSH Server`

7. Check the box next to "OpenSSH Server"

8. Click "Install" (or "Next" then "Install")

9. Wait for it to install (might take a minute)

### Step 5.2: Start the SSH Service

Now we need to turn on the SSH service:

1. Open PowerShell as Administrator (remember: search for PowerShell, right-click, "Run as administrator")

2. Start the SSH service by typing:
   ```
   Start-Service sshd
   ```
   Press Enter.

3. Make SSH start automatically when Windows boots:
   ```
   Set-Service -Name sshd -StartupType Automatic
   ```
   Press Enter.

### Step 5.3: Verify SSH is Running

1. In the same PowerShell window, type:
   ```
   Get-Service sshd
   ```
   Press Enter.

2. You should see:
   ```
   Status   Name               DisplayName
   ------   ----               -----------
   Running  sshd               OpenSSH SSH Server
   ```

If it says "Running", you're good!

### Step 5.4: Find Your Windows Username

You'll need your Windows username to connect. To find it:

1. In PowerShell, type:
   ```
   $env:USERNAME
   ```
   Press Enter.

2. It will show your username (like `simon` or `JohnDoe`)

**Write this down! You'll need it for Termius.**

---

## Part 6: Installing Tailscale on iPhone

Now let's set up your iPhone to connect to your PC.

### Step 6.1: Download Tailscale

1. On your iPhone, open the App Store

2. Tap the Search tab (bottom right, magnifying glass icon)

3. Type "Tailscale" in the search bar

4. Find the app called "Tailscale" by Tailscale Inc. (it has a logo of connected dots)

5. Tap "Get" (or the cloud icon if you've downloaded it before)

6. Authenticate with Face ID, Touch ID, or your password

7. Wait for it to download and install

### Step 6.2: Set Up Tailscale

1. Open the Tailscale app

2. Tap "Get Started" or "Log in"

3. **IMPORTANT:** Log in with the SAME account you used on Windows!
   - If you used Google on Windows, use Google here
   - If you used Microsoft on Windows, use Microsoft here
   - Same account = same private network

4. You'll be asked to allow VPN configurations. Tap "Allow"

5. Enter your iPhone passcode if asked

6. Complete the login process

### Step 6.3: Verify Connection

1. In the Tailscale app, you should see a screen showing your devices

2. You should see:
   - Your iPhone (probably says "This device")
   - Your Windows PC (with its name)

3. Both should show as connected

4. Look at your Windows PC in the list - note its IP address (the `100.x.x.x` number). It should match what you wrote down earlier.

### Step 6.4: Enable Tailscale VPN

1. At the top of the Tailscale app, there's a toggle switch

2. Make sure it's turned ON (green/blue)

3. When it's on, your devices can communicate securely

**You can leave this on all the time. It uses minimal battery and only activates when connecting to your Tailscale devices.**

---

## Part 7: Installing Termius on iPhone

Termius is the app that lets you actually type commands on your PC from your iPhone.

### Step 7.1: Download Termius

1. Open the App Store on your iPhone

2. Search for "Termius"

3. Find the app called "Termius - Terminal & SSH" (blue icon with a ">" symbol)

4. Tap "Get" and install it

### Step 7.2: First Launch

1. Open Termius

2. You'll see a welcome screen. You can:
   - Create a free account (allows syncing between devices)
   - Or skip and use without an account

   Either is fine. A free account is convenient but optional.

3. If you create an account, complete the signup process

### Step 7.3: Add Your Windows PC as a Host

Now we'll tell Termius how to connect to your PC:

1. In Termius, tap the "Hosts" tab (bottom of screen)

2. Tap the "+" button (top right) to add a new host

3. Tap "New Host"

4. Fill in the fields:

   **Alias:**
   - This is just a nickname for your convenience
   - Type something like: `My Windows PC`

   **Hostname:**
   - This is your Tailscale IP address
   - Type the `100.x.x.x` number from earlier (e.g., `100.64.23.45`)

   **Use SSH:** Make sure this is enabled (toggle should be on)

   **Port:**
   - Leave as `22` (this is the default SSH port)

   **Username:**
   - Type your Windows username (what you found in Step 5.4)

   **Password:**
   - Tap on the password field
   - Type your Windows login password

5. Tap "Save" (top right)

### Step 7.4: Test the Connection

1. In the Hosts list, tap on "My Windows PC" (or whatever you named it)

2. Termius will attempt to connect

3. You might see a message about the host's authenticity:
   ```
   The authenticity of host '100.x.x.x' can't be established...
   Are you sure you want to continue connecting?
   ```

   Tap "Continue" or "Yes" - this is normal for first-time connections

4. If everything is set up correctly, you'll see a command prompt!

It might look like:
```
Microsoft Windows [Version 10.0.xxxxx]
(c) Microsoft Corporation. All rights reserved.

C:\Users\YourUsername>
```

**Congratulations! Your iPhone is now connected to your Windows PC!**

### Troubleshooting Connection Issues

**"Connection refused" or can't connect?**
- Make sure Tailscale is running on BOTH devices
- Make sure both devices are connected to the internet
- Verify the IP address is correct
- Make sure SSH is running on Windows (re-check Part 5)

**"Password authentication failed"?**
- Double-check you're using your Windows login password
- Make sure the username is correct (case-sensitive!)

---

## Part 8: Connecting to WSL and tmux from iPhone

When you connect via SSH, you land in Windows. But Claude Code runs in WSL (Linux). Here's how to get there.

### Step 8.1: Enter WSL

After connecting with Termius, you're in Windows command line. To enter Linux:

1. Type:
   ```
   wsl
   ```

2. Press Enter (tap the return key on your iPhone keyboard)

3. Your prompt will change from:
   ```
   C:\Users\YourName>
   ```
   To something like:
   ```
   username@COMPUTERNAME:~$
   ```

   You're now in Linux!

### Step 8.2: Attach to Your tmux Session

If you have a tmux session running (which you should from your PC):

1. Type:
   ```
   tmux attach -t claude -d
   ```

2. Press Enter

3. You're now in your Claude Code session!

### What if There's No Session?

If you see an error like "no sessions" or "can't find session 'claude'":

- You haven't started a session yet, OR
- The session was closed/killed

To start a new session:
```
tmux new -s claude
```

Then launch Claude Code:
```
claude
```

---

## Part 9: The Complete Daily Workflow

Here's what your typical day looks like:

### Morning - Starting Work on Your PC

1. Open Ubuntu (search for it in Windows)

2. Start or attach to your tmux session:
   ```
   tmux new -s claude
   ```
   Or if it already exists:
   ```
   tmux attach -t claude
   ```

3. Navigate to your project:
   ```
   cd ~/projects/my-app
   ```

4. Start Claude Code:
   ```
   claude
   ```

5. Work on your project...

### Need to Leave? Just Detach

1. Press `Ctrl+b`, then `d`

2. You see `[detached (from session claude)]`

3. You can now close Ubuntu, close your laptop, whatever - the session keeps running!

### On the Go - Continue from iPhone

1. Make sure Tailscale is on (open the app if unsure)

2. Open Termius

3. Tap on your Windows PC connection

4. Once connected, type:
   ```
   wsl
   ```
   Press Enter.

5. Attach to your session:
   ```
   tmux attach -t claude -d
   ```
   Press Enter.

6. You're back in Claude Code, exactly where you left off!

### Back at Your PC

1. Open Ubuntu (or any terminal)

2. Get your session back:
   ```
   tmux attach -t claude -d
   ```
   (This kicks your iPhone off the session)

3. Continue working!

---

## Part 10: Tips for Using Claude Code on iPhone

### Keyboard Tips

- The iOS keyboard works, but you might want to:
  - Rotate to landscape mode for more screen space
  - Use an external Bluetooth keyboard for longer sessions

### Termius-Specific Tips

- **Pinch to zoom** - Make text larger/smaller
- **Swipe left** - Show extra keys (Ctrl, Alt, Tab, etc.)
- **Tap and hold** - Access copy/paste options

### Sending Special Keys

In Termius, swipe left on the keyboard to reveal special keys:
- **Ctrl** - For key combinations
- **Esc** - Exit modes in vim/nano
- **Tab** - Autocomplete commands
- **Arrow keys** - Navigate

### For tmux Key Combinations

To press `Ctrl+b` (tmux prefix):
1. Tap and hold `Ctrl` on the extra keys bar
2. Tap `b`
3. Release `Ctrl`
4. Tap `d` (for detach) or whatever other key you need

---

## Part 11: Setting Up Slack Notifications (Optional)

Want to get a Slack message when Claude needs your attention? Set up a notification hook.

### Step 11.1: Create a Slack Webhook

1. Go to https://api.slack.com/apps

2. Click "Create New App"

3. Choose "From scratch"

4. Give it a name like "Claude Notifier"

5. Select your workspace

6. Click "Create App"

7. In the left sidebar, click "Incoming Webhooks"

8. Turn on "Activate Incoming Webhooks"

9. Click "Add New Webhook to Workspace"

10. Choose a channel where you want notifications (like #claude or #general)

11. Click "Allow"

12. Copy the Webhook URL that appears - it looks like:
    ```
    https://hooks.slack.com/services/XXXXX/YYYYY/ZZZZZ
    ```

### Step 11.2: Create the Notification Script

1. In your WSL terminal, create the hooks directory:
   ```
   mkdir -p ~/.claude/hooks
   ```

2. Create the script:
   ```
   nano ~/.claude/hooks/stop.sh
   ```

3. Paste this content (replace YOUR_WEBHOOK_URL with your actual URL):
   ```bash
   #!/bin/bash
   curl -X POST -H 'Content-type: application/json' \
     --data '{"text":"Claude needs your attention!"}' \
     https://hooks.slack.com/services/XXXXX/YYYYY/ZZZZZ
   ```

4. Press `Ctrl+O` to save, then `Enter`, then `Ctrl+X` to exit

5. Make the script executable:
   ```
   chmod +x ~/.claude/hooks/stop.sh
   ```

6. Configure Claude Code to use hooks (check Claude Code documentation for your version)

---

## Part 12: Security Best Practices

### Keep Your Devices Secure

1. **Use strong passwords** on both Windows and your iPhone

2. **Enable screen lock** on both devices

3. **Keep software updated** - Windows, WSL, Tailscale, and Termius

### What Tailscale Protects

- All traffic between your devices is encrypted
- Your PC isn't exposed to the public internet
- Only devices logged into your Tailscale account can connect

### What to Be Careful About

- **Don't share your Tailscale IP** with people you don't trust
- **Log out of Tailscale** on devices you no longer use
- **Don't run autonomous agents** on machines with sensitive data

---

## Troubleshooting Reference

### Problem: Can't connect from iPhone

**Check in this order:**
1. Is Tailscale connected on your iPhone? (Open app, check status)
2. Is Tailscale running on Windows? (Check system tray icon)
3. Is the IP address correct in Termius?
4. Is SSH running on Windows? Run `Get-Service sshd` in PowerShell
5. Is your Windows firewall blocking connections?

### Problem: Connected but tmux session not found

**Solutions:**
- List sessions: `tmux ls`
- Maybe you haven't created one yet: `tmux new -s claude`
- Maybe you typed the name wrong - session names are case-sensitive

### Problem: Slow/Laggy connection

**This is normal, especially on mobile data:**
- tmux handles disconnections gracefully
- Consider using WiFi when possible
- The session state is preserved even if you disconnect

### Problem: "Permission denied" in WSL

**Try:**
- Make sure you're using the correct password
- Password is case-sensitive!

### Problem: Claude Code not found

**Solutions:**
- Make sure Claude Code is installed in WSL
- Check if it's in your PATH: `which claude`
- Reinstall if necessary

---

## Quick Reference Card

Print this or save it on your phone:

```
# START SESSION (on PC)
wsl
tmux new -s claude
claude

# DETACH (keep running)
Ctrl+b, then d

# ATTACH FROM ANYWHERE
wsl
tmux attach -t claude -d

# LIST SESSIONS
tmux ls

# END SESSION
exit

# KILL SESSION
tmux kill-session -t claude
```

---

## Congratulations!

You now have:
- ✅ tmux keeping your sessions alive
- ✅ Tailscale connecting your devices securely
- ✅ Termius letting you connect from your iPhone
- ✅ Full Claude Code access from anywhere

Welcome to the world of mobile AI-assisted development!

---

## Sources

- [tmux Wiki](https://github.com/tmux/tmux/wiki)
- [tmux Getting Started](https://github.com/tmux/tmux/wiki/Getting-Started)
- [Tailscale Installation](https://tailscale.com/kb/1017/install)
- [Tailscale Windows Setup](https://tailscale.com/kb/1022/install-windows)
- [Termius Documentation](https://termius.com/documentation/what-is-termius)
- [WSL Installation Guide](https://learn.microsoft.com/en-us/windows/wsl/install)
- [Windows OpenSSH Setup](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse)
