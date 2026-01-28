# SSH Tutorial for Absolute Beginners

## What You'll Learn

By the end of this tutorial, you'll understand:
- What SSH is and why it exists
- How SSH keeps your connections secure
- How to use SSH to connect to other computers
- Common SSH commands and what they do

---

## Part 1: What is SSH?

### The Simple Explanation

**SSH** stands for **S**ecure **Sh**ell.

Think of it as a **secure telephone line for computers**. Just like you can call someone on the phone and have a conversation, SSH lets you "call" another computer and type commands on it as if you were sitting right in front of it.

### The Problem SSH Solves

Imagine you have a computer at home, but you're at a coffee shop. You need to run a program on your home computer. How do you do it?

**Without SSH:**
```
You (at coffee shop)                Your Computer (at home)
      │                                      │
      │   "Hey, run this command!"           │
      │ ────────────────────────────────────►│
      │                                      │
      │        Anyone can read this!         │
      │        ┌──────────────────┐         │
      │        │ 👀 Hackers       │         │
      │        │ 👀 Your ISP      │         │
      │        │ 👀 Anyone        │         │
      │        └──────────────────┘         │
      │                                      │
```

**With SSH:**
```
You (at coffee shop)                Your Computer (at home)
      │                                      │
      │   🔒 Encrypted message 🔒            │
      │ ════════════════════════════════════►│
      │                                      │
      │   Looks like random garbage to       │
      │   anyone watching!                   │
      │        ┌──────────────────┐         │
      │        │ 😕 Hackers see:  │         │
      │        │ "x7#@kL9$mN..."  │         │
      │        └──────────────────┘         │
      │                                      │
```

### Real-World Analogy

Imagine you need to send a secret message across a crowded room. You have two options:

1. **Shouting across the room** (no SSH): Everyone hears your message
2. **Using walkie-talkies with encryption** (SSH): Only the person with the matching walkie-talkie can understand

SSH is like those encrypted walkie-talkies for computers.

---

## Part 2: Why Do You Need SSH?

### Use Case 1: Remote Access

You want to access your home computer from anywhere in the world.

```
┌─────────────────┐                    ┌─────────────────┐
│  You on Phone   │                    │  Your Home PC   │
│  or Laptop      │                    │                 │
│                 │      SSH           │  Running your   │
│  At a hotel     │ ──────────────────►│  programs,      │
│  in Tokyo       │    Connection      │  files, etc.    │
│                 │                    │                 │
└─────────────────┘                    └─────────────────┘

Result: You can type commands on your home PC
        as if you were sitting in front of it!
```

### Use Case 2: Secure File Transfer

You need to send sensitive files between computers.

```
┌─────────────────┐                    ┌─────────────────┐
│  Your Laptop    │                    │  Company Server │
│                 │                    │                 │
│  secret_        │      SFTP          │                 │
│  document.pdf   │ ──────────────────►│  Now has the    │
│                 │  (SSH File         │  file securely  │
│                 │   Transfer)        │                 │
└─────────────────┘                    └─────────────────┘
```

### Use Case 3: Secure Tunneling

You want to access a service that's only available on a private network.

```
                        FIREWALL
                           ║
┌───────────┐             ║            ┌───────────────┐
│  You      │             ║            │ Internal      │
│  (outside)│   SSH       ║   ════►    │ Website       │
│           │───tunnel────╬────────────│ (normally     │
│           │             ║            │  blocked)     │
└───────────┘             ║            └───────────────┘
                          ║
              SSH "drills through" the firewall safely
```

---

## Part 3: How SSH Works (Simplified)

### The Handshake Process

When you connect via SSH, here's what happens:

```
┌──────────────────────────────────────────────────────────────────┐
│                    THE SSH HANDSHAKE                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│   STEP 1: "Hello!"                                               │
│   ─────────────────                                              │
│   Your Computer                          Remote Computer         │
│        │                                        │                │
│        │──── "Hi, I want to connect!" ─────────►│                │
│        │                                        │                │
│        │◄─── "OK, here's my ID card" ──────────│                │
│        │     (server's public key)              │                │
│                                                                   │
│   STEP 2: Verify Identity                                        │
│   ───────────────────────                                        │
│        │                                        │                │
│        │ You verify the ID is correct           │                │
│        │ (first time: you say "yes, trust it")  │                │
│        │ (later: computer remembers it)         │                │
│                                                                   │
│   STEP 3: Create Secret Code                                     │
│   ──────────────────────────                                     │
│        │                                        │                │
│        │◄───────────────────────────────────────│                │
│        │   Both computers create a secret       │                │
│        │   code that only they know             │                │
│        │───────────────────────────────────────►│                │
│                                                                   │
│   STEP 4: Encrypted Communication                                │
│   ───────────────────────────────                                │
│        │                                        │                │
│        │════ Everything from now on ═══════════►│                │
│        │     is encrypted with the              │                │
│        │◄════ secret code ═════════════════════│                │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### What "Keys" Are

SSH uses something called "keys" - think of them like a special lock and key system:

```
┌─────────────────────────────────────────────────────────────────┐
│                     SSH KEY PAIRS                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   PUBLIC KEY                         PRIVATE KEY                 │
│   ══════════                         ═══════════                 │
│                                                                  │
│   ┌───────────────────┐             ┌───────────────────┐       │
│   │                   │             │                   │       │
│   │   🔓 Like a       │             │   🔑 Like the     │       │
│   │   padlock you     │             │   only key that   │       │
│   │   can give to     │             │   opens that      │       │
│   │   anyone          │             │   padlock         │       │
│   │                   │             │                   │       │
│   └───────────────────┘             └───────────────────┘       │
│                                                                  │
│   • You can share this freely       • NEVER share this!        │
│   • Lives on the remote computer    • Lives on YOUR computer   │
│   • Anyone can see it               • Must stay secret         │
│                                                                  │
│   HOW THEY WORK TOGETHER:                                       │
│   ───────────────────────                                       │
│                                                                  │
│   Remote Computer:        Your Computer:                        │
│   "I'll only let in       "I have the key                      │
│   someone who has         that matches that                     │
│   the key to this         padlock!"                             │
│   padlock"                                                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Part 4: SSH Components

### What You Need to Use SSH

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   YOUR COMPUTER                      REMOTE COMPUTER            │
│   (the one you're sitting at)        (the one you want to       │
│                                       connect to)               │
│   ┌─────────────────────┐           ┌─────────────────────┐    │
│   │                     │           │                     │    │
│   │   SSH CLIENT        │           │   SSH SERVER        │    │
│   │   ───────────       │           │   ──────────        │    │
│   │                     │           │                     │    │
│   │   The program that  │   ═══►    │   The program that  │    │
│   │   makes the call    │           │   answers the call  │    │
│   │                     │           │                     │    │
│   │   (Already on Mac/  │           │   (Needs to be      │    │
│   │    Linux. Install   │           │    installed and    │    │
│   │    on Windows)      │           │    running)         │    │
│   │                     │           │                     │    │
│   └─────────────────────┘           └─────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The SSH Client

This is what YOU use to connect:

| Platform | SSH Client Status |
|----------|-------------------|
| Mac | Built-in (just open Terminal) |
| Linux | Built-in (just open Terminal) |
| Windows 10/11 | Built-in (open PowerShell or Command Prompt) |
| iPhone/iPad | Need an app like Termius |
| Android | Need an app like Termius or JuiceSSH |

### The SSH Server

This runs on the computer you want to connect TO:

| Platform | How to Enable |
|----------|---------------|
| Mac | System Preferences → Sharing → Remote Login |
| Linux | Usually installed, run `sudo systemctl start sshd` |
| Windows 10/11 | Install via Optional Features (we'll cover this) |

---

## Part 5: Using SSH - Step by Step

### Step 5.1: Opening a Terminal

**On Windows:**
1. Press the Windows key
2. Type "PowerShell" or "Command Prompt"
3. Click to open

**On Mac:**
1. Press Cmd + Space
2. Type "Terminal"
3. Press Enter

**On Linux:**
1. Press Ctrl + Alt + T (usually)
2. Or search for "Terminal" in your applications

### Step 5.2: Basic SSH Command

The basic command structure is:

```
ssh username@hostname
```

Let's break this down:

```
    ssh           username    @    hostname
     │               │        │       │
     │               │        │       └── The address of the remote computer
     │               │        │           (IP address or domain name)
     │               │        │
     │               │        └── Separator (literally the @ symbol)
     │               │
     │               └── Your username on the remote computer
     │
     └── The SSH command itself
```

**Real Examples:**

```bash
# Connect to a computer at IP address 192.168.1.100 as user "john"
ssh john@192.168.1.100

# Connect to a server at example.com as user "admin"
ssh admin@example.com

# Connect to a Tailscale device as user "simon"
ssh simon@100.64.23.45
```

### Step 5.3: Your First Connection

When you connect to a new computer for the first time, you'll see this:

```
$ ssh john@192.168.1.100
The authenticity of host '192.168.1.100' can't be established.
ED25519 key fingerprint is SHA256:xYz123AbC456...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

**What this means:**
- Your computer hasn't seen this remote computer before
- It's showing you the remote computer's "fingerprint" (ID)
- It's asking: "Do you trust this computer?"

**What to do:**
- If you're connecting to YOUR OWN computer, type `yes` and press Enter
- If you're unsure, verify the fingerprint through another channel first

After typing `yes`:

```
Warning: Permanently added '192.168.1.100' (ED25519) to the list of known hosts.
john@192.168.1.100's password:
```

Now type your password (nothing will appear as you type - that's normal!) and press Enter.

```
Welcome to Ubuntu 22.04 LTS
Last login: Mon Jan 27 10:30:00 2026

john@remote-computer:~$
```

**You're in!** You're now typing commands on the remote computer.

### Step 5.4: Exiting SSH

When you're done, type:

```bash
exit
```

Or press `Ctrl + D`

You'll see:

```
Connection to 192.168.1.100 closed.
```

And you're back on your own computer.

---

## Part 6: Setting Up SSH on Windows (Server Side)

If you want OTHER devices to connect TO your Windows PC, you need to set up the SSH server.

### Step 6.1: Install OpenSSH Server

1. **Open Settings**
   - Click the Start button
   - Click the gear icon (Settings)

2. **Navigate to Optional Features**
   - Click "Apps"
   - Click "Optional features"

3. **Add OpenSSH Server**
   - Click "Add a feature" (or "View features")
   - In the search box, type: `OpenSSH Server`
   - Check the box next to "OpenSSH Server"
   - Click "Install"

4. **Wait for installation**
   - This may take a minute or two

### Step 6.2: Start the SSH Service

1. **Open PowerShell as Administrator**
   - Search for "PowerShell"
   - Right-click "Windows PowerShell"
   - Click "Run as administrator"
   - Click "Yes" when asked

2. **Start the SSH service**
   ```powershell
   Start-Service sshd
   ```

3. **Make it start automatically when Windows boots**
   ```powershell
   Set-Service -Name sshd -StartupType Automatic
   ```

4. **Verify it's running**
   ```powershell
   Get-Service sshd
   ```

   You should see:
   ```
   Status   Name               DisplayName
   ------   ----               -----------
   Running  sshd               OpenSSH SSH Server
   ```

### Step 6.3: Test It

From another computer (or your phone with Termius), try:

```bash
ssh your_windows_username@your_ip_address
```

---

## Part 7: Common SSH Commands

### Connecting

```bash
# Basic connection
ssh username@hostname

# Connect on a different port (default is 22)
ssh -p 2222 username@hostname

# Connect with a specific key file
ssh -i ~/.ssh/my_key username@hostname
```

### Copying Files

SSH includes a tool called `scp` (secure copy):

```bash
# Copy file FROM remote TO local
scp username@hostname:/path/to/remote/file.txt /path/to/local/

# Copy file FROM local TO remote
scp /path/to/local/file.txt username@hostname:/path/to/remote/

# Copy entire folder
scp -r username@hostname:/path/to/folder /local/destination/
```

### Running a Single Command

You don't have to start a full session. Run one command and exit:

```bash
# Check disk space on remote computer
ssh username@hostname "df -h"

# See what's running
ssh username@hostname "ps aux"

# Check the date/time
ssh username@hostname "date"
```

---

## Part 8: SSH Security Tips

### DO:

```
✓ Use strong passwords (or better: SSH keys)
✓ Keep SSH software updated
✓ Verify host fingerprints when connecting to new servers
✓ Use a firewall to limit who can reach your SSH port
✓ Consider using a non-standard port (not 22)
```

### DON'T:

```
✗ Share your private SSH keys
✗ Use simple passwords like "password123"
✗ Ignore warnings about changed host keys
✗ Leave SSH servers open to the entire internet without protection
✗ Run SSH as root if not necessary
```

### The "Host Key Changed" Warning

If you see this, **STOP and think**:

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
```

**This could mean:**
1. The remote computer was reinstalled (harmless)
2. You're connecting to a different computer than before (might be an attack)
3. Someone is intercepting your connection (dangerous!)

**What to do:**
- If you know the remote computer was reinstalled: it's probably fine
- If you're not sure: contact the server administrator
- If this is unexpected: **DO NOT CONTINUE** - investigate first

---

## Part 9: Understanding SSH Ports

### What is a Port?

Think of an IP address as a building's street address. Ports are like apartment numbers:

```
IP Address: 192.168.1.100     ←── The building
Port: 22                      ←── Apartment #22 (where SSH lives)

Other common "apartments":
- Port 80: Web servers (HTTP)
- Port 443: Secure web servers (HTTPS)
- Port 22: SSH (that's us!)
- Port 21: FTP (file transfer)
```

### Why Port 22?

SSH uses port 22 by default. It's like saying "the SSH service lives in apartment 22."

When you type `ssh user@host`, your computer automatically goes to port 22.

To use a different port:
```bash
ssh -p 2222 user@host
```

---

## Part 10: Troubleshooting

### Problem: "Connection refused"

```
ssh: connect to host 192.168.1.100 port 22: Connection refused
```

**Causes:**
- SSH server isn't running on the remote computer
- Firewall is blocking port 22
- Wrong IP address

**Solutions:**
1. Make sure the SSH server is running on the remote computer
2. Check firewall settings
3. Verify the IP address is correct

### Problem: "Permission denied"

```
Permission denied (publickey,password).
```

**Causes:**
- Wrong username
- Wrong password
- SSH key authentication required but you don't have the key

**Solutions:**
1. Double-check your username (it's case-sensitive!)
2. Make sure you're typing the password correctly
3. If using keys, make sure you have the right private key

### Problem: "Network is unreachable"

```
ssh: connect to host 192.168.1.100 port 22: Network is unreachable
```

**Causes:**
- You're not connected to the internet/network
- The remote computer is on a different network
- Routing issue

**Solutions:**
1. Check your internet connection
2. Make sure you're on the same network (or using VPN like Tailscale)
3. Verify the IP address is reachable

### Problem: "Connection timed out"

```
ssh: connect to host 192.168.1.100 port 22: Connection timed out
```

**Causes:**
- Remote computer is off or unreachable
- Firewall dropping packets silently
- Network issue between you and the remote computer

**Solutions:**
1. Make sure the remote computer is on
2. Check firewall settings on both ends
3. Try pinging the remote computer first: `ping 192.168.1.100`

---

## Part 11: Quick Reference

### Commands Cheat Sheet

| What you want to do | Command |
|---------------------|---------|
| Connect to remote computer | `ssh user@host` |
| Connect on different port | `ssh -p PORT user@host` |
| Exit SSH session | `exit` or Ctrl+D |
| Copy file to remote | `scp file.txt user@host:/path/` |
| Copy file from remote | `scp user@host:/path/file.txt ./` |
| Run single command | `ssh user@host "command"` |

### Important Files

| File | Purpose |
|------|---------|
| `~/.ssh/known_hosts` | Stores fingerprints of servers you've connected to |
| `~/.ssh/id_rsa` | Your private key (NEVER share!) |
| `~/.ssh/id_rsa.pub` | Your public key (safe to share) |
| `~/.ssh/authorized_keys` | Public keys allowed to connect to this computer |
| `~/.ssh/config` | Your personal SSH configuration |

---

## Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                        SSH IN A NUTSHELL                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   WHAT:  Secure Shell - encrypted remote computer access        │
│                                                                  │
│   WHY:   • Access computers remotely                            │
│          • Everything is encrypted (secure)                     │
│          • Transfer files safely                                │
│          • Run commands on remote machines                      │
│                                                                  │
│   HOW:   ssh username@hostname                                  │
│                                                                  │
│   COMPONENTS:                                                    │
│          • SSH Client: You use this to connect                  │
│          • SSH Server: Runs on the computer you connect to      │
│          • Keys: Special lock/key pairs for authentication      │
│                                                                  │
│   REMEMBER:                                                      │
│          • Default port is 22                                   │
│          • Password doesn't show when typing (that's normal!)   │
│          • Type "exit" to disconnect                            │
│          • Never share your private key!                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## What's Next?

Now that you understand SSH, you're ready to learn about:
- **Tailscale**: Makes SSH easier by giving your devices fixed addresses
- **tmux**: Keeps your SSH sessions alive even when you disconnect
- **Termius**: A beautiful SSH client for your phone

Each of these builds on SSH to make remote access even more powerful!
