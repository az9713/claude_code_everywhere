# Tailscale Tutorial for Absolute Beginners

## What You'll Learn

By the end of this tutorial, you'll understand:
- What Tailscale is and the problem it solves
- Why Tailscale makes remote access so much easier
- How to install and set up Tailscale on your devices
- How Tailscale keeps your connections secure

---

## Part 1: What is Tailscale?

### The Simple Explanation

**Tailscale** creates a **private network** that connects all your devices together, no matter where they are in the world.

Think of it as building **secret tunnels** between your devices that only you can use.

### The Problem Tailscale Solves

#### The Old Way (Without Tailscale)

Imagine you have a computer at home and you want to connect to it from a coffee shop:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    THE PROBLEM: CONNECTING WITHOUT TAILSCALE                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   YOU AT COFFEE SHOP                           YOUR HOME COMPUTER           │
│   ┌─────────────────┐                         ┌─────────────────┐           │
│   │   📱 Laptop     │                         │   🖥️ Desktop    │           │
│   │                 │                         │                 │           │
│   │ "I want to      │         ???             │   Behind        │           │
│   │  connect to     │ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│   router        │           │
│   │  my home PC"    │                         │   and firewall  │           │
│   └─────────────────┘                         └─────────────────┘           │
│                                                        │                     │
│   PROBLEMS:                                           │                     │
│   ─────────                                           ▼                     │
│                                              ┌─────────────────┐            │
│   ❌ Your home PC's address changes          │     ROUTER      │            │
│      (Dynamic IP)                            │   "Who are you? │            │
│                                              │    Go away!"    │            │
│   ❌ Your router blocks incoming             └─────────────────┘            │
│      connections (firewall)                                                  │
│                                                                              │
│   ❌ You'd need to configure "port                                          │
│      forwarding" (complicated!)                                              │
│                                                                              │
│   ❌ Your IP is exposed to the                                              │
│      whole internet (security risk)                                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### The Tailscale Way

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    THE SOLUTION: WITH TAILSCALE                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   YOU AT COFFEE SHOP                           YOUR HOME COMPUTER           │
│   ┌─────────────────┐                         ┌─────────────────┐           │
│   │   📱 Laptop     │                         │   🖥️ Desktop    │           │
│   │                 │                         │                 │           │
│   │ Tailscale       │      ENCRYPTED          │   Tailscale     │           │
│   │ IP: 100.64.1.20 │ ════════════════════════│   IP: 100.64.1.10│          │
│   │                 │       TUNNEL            │                 │           │
│   │ Just type:      │   (Through coffee       │   Always        │           │
│   │ ssh simon@      │    shop WiFi,           │   reachable at  │           │
│   │ 100.64.1.10     │    through router,      │   100.64.1.10   │           │
│   │                 │    through home         │                 │           │
│   └─────────────────┘    router - doesn't     └─────────────────┘           │
│                          matter!)                                            │
│                                                                              │
│   ✅ Fixed IP address (100.64.x.x) that never changes                       │
│   ✅ Works through any firewall automatically                               │
│   ✅ No port forwarding needed                                              │
│   ✅ Encrypted end-to-end                                                   │
│   ✅ Only YOUR devices can connect                                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Real-World Analogy

**Without Tailscale:** You live in a gated community. To visit your own house, you need to convince the security guard (router/firewall) to let you in every time. The guard changes, the rules change, and sometimes they just won't let you in.

**With Tailscale:** You have a secret underground tunnel that goes directly from wherever you are to your house. No guards, no gates - just walk through your private tunnel.

---

## Part 2: Why Do You Need Tailscale?

### Reason 1: Your Home IP Address Changes

Most internet connections have a "dynamic IP" - your address changes regularly.

```
WITHOUT TAILSCALE:
──────────────────

Monday:    Your home IP is 73.45.123.89
Tuesday:   Your home IP is 73.45.167.22  ← It changed!
Wednesday: Your home IP is 73.45.201.45  ← Changed again!

"Wait, what's my home IP today??" 😫

WITH TAILSCALE:
───────────────

Monday:    Your Tailscale IP is 100.64.1.10
Tuesday:   Your Tailscale IP is 100.64.1.10  ← Same!
Wednesday: Your Tailscale IP is 100.64.1.10  ← Always the same!

"My home PC is always at 100.64.1.10" 😊
```

### Reason 2: Firewalls and Routers Block You

Your router is designed to block incoming connections (for security). This makes it hard to connect FROM outside TO your home.

```
NORMAL INTERNET:
────────────────

   Your phone                    Your router              Your PC
      │                              │                       │
      │──── "Hey, let me in!" ──────►│                       │
      │                              │                       │
      │◄─── "Who are you? NO!" ──────│                       │
      │                              │                       │
      ✗ Connection blocked!

TAILSCALE:
──────────

   Your phone                                            Your PC
      │                                                     │
      │   Both devices logged into Tailscale, so they      │
      │   recognize each other like old friends            │
      │                                                     │
      │═══════════════ Direct connection! ═════════════════│
      │                                                     │
      ✓ Just works!
```

### Reason 3: Security

Your devices are NOT exposed to the public internet:

```
REGULAR PORT FORWARDING:
────────────────────────

  ┌─────────────────────────────────────────────────────────────┐
  │                      THE INTERNET                            │
  │                                                              │
  │    👤 You        👤 Hackers       👤 Bots                   │
  │      │              │                │                       │
  │      └──────────────┴────────────────┘                       │
  │                     │                                        │
  │              ▼      ▼      ▼                                │
  │         ┌────────────────────┐                              │
  │         │    YOUR ROUTER     │                              │
  │         │  (port 22 open)    │◄─── Anyone can try to       │
  │         └────────────────────┘     connect and attack!      │
  │                     │                                        │
  │                     ▼                                        │
  │              YOUR COMPUTER                                   │
  │                                                              │
  └─────────────────────────────────────────────────────────────┘

TAILSCALE:
──────────

  ┌─────────────────────────────────────────────────────────────┐
  │                      THE INTERNET                            │
  │                                                              │
  │    👤 You        👤 Hackers       👤 Bots                   │
  │      │              │                │                       │
  │      │              │                │                       │
  │      │              ✗ "Can't find    ✗ "Nothing              │
  │      │                 any open         to attack!"          │
  │      │                 ports!"                               │
  │      │                                                       │
  │      └───────► TAILSCALE NETWORK ◄───── Only your           │
  │                (encrypted tunnel)        devices!            │
  │                        │                                     │
  │                        ▼                                     │
  │                 YOUR COMPUTER                                │
  │                 (invisible to internet)                      │
  │                                                              │
  └─────────────────────────────────────────────────────────────┘
```

### Reason 4: It Just Works Everywhere

Tailscale works through:
- Home WiFi
- Coffee shop WiFi
- Hotel WiFi
- Mobile data (4G/5G)
- Corporate firewalls
- Anywhere with internet!

No configuration needed for each network.

---

## Part 3: How Tailscale Works (Simplified)

### The Tailscale Network (Tailnet)

When you install Tailscale on your devices and log in with the same account, they all join your personal "Tailnet":

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           YOUR TAILNET                                       │
│                   (Your private network in the cloud)                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│      ┌─────────────────┐                     ┌─────────────────┐            │
│      │ Windows Desktop │                     │     MacBook     │            │
│      │ 100.64.1.10     │                     │   100.64.1.15   │            │
│      └────────┬────────┘                     └────────┬────────┘            │
│               │                                       │                      │
│               │         ┌─────────────────┐          │                      │
│               └─────────│   TAILSCALE     │──────────┘                      │
│                         │   SERVERS       │                                  │
│               ┌─────────│  (Coordination  │──────────┐                      │
│               │         │   Only)         │          │                      │
│               │         └─────────────────┘          │                      │
│               │                                       │                      │
│      ┌────────┴────────┐                     ┌───────┴─────────┐            │
│      │     iPhone      │                     │  Work Laptop    │            │
│      │   100.64.1.20   │                     │   100.64.1.25   │            │
│      └─────────────────┘                     └─────────────────┘            │
│                                                                              │
│   All devices can talk to each other using their 100.x.x.x addresses!       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### What Tailscale Servers Do (and Don't Do)

```
TAILSCALE SERVERS:                    YOUR ACTUAL DATA:
──────────────────                    ─────────────────

✓ Help devices find each other        ✓ Goes DIRECTLY between your devices
✓ Coordinate connections               ✗ Does NOT go through Tailscale servers
✓ Handle authentication                ✓ Encrypted end-to-end
                                       ✓ Tailscale cannot see your data!

Think of it like a phone directory:
- Tailscale knows your phone number (IP)
- But doesn't listen to your calls (data)
```

### WireGuard: The Secret Sauce

Tailscale uses a technology called **WireGuard** for encryption:

```
┌─────────────────────────────────────────────────────────────────┐
│                         WIREGUARD                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   • Modern, fast VPN protocol                                   │
│   • Very secure encryption                                      │
│   • Low battery usage on phones                                 │
│   • Handles network changes gracefully                          │
│                                                                  │
│   Your data travels through a WireGuard tunnel:                 │
│                                                                  │
│   ┌──────────┐                              ┌──────────┐        │
│   │ Device A │ ══════════════════════════ │ Device B │        │
│   └──────────┘    Encrypted Tunnel          └──────────┘        │
│                   (WireGuard)                                    │
│                                                                  │
│   Even if someone intercepts the tunnel traffic,                │
│   they only see random encrypted gibberish.                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Part 4: Installing Tailscale

### On Windows

#### Step 4.1: Download Tailscale

1. Open your web browser

2. Go to: **https://tailscale.com/download/windows**

3. Click the big **"Download Tailscale for Windows"** button

4. A file downloads (something like `tailscale-setup-1.XX.X.exe`)

#### Step 4.2: Install Tailscale

1. Find the downloaded file (probably in your Downloads folder)

2. **Double-click** the file to run it

3. If Windows asks "Do you want to allow this app to make changes to your device?":
   - Click **"Yes"**

4. The installer runs. Click through any prompts (defaults are fine)

5. When it finishes, Tailscale is installed!

#### Step 4.3: Log In to Tailscale

1. Look at the bottom-right of your screen in the **system tray** (near the clock)

2. You might need to click the small **"^"** arrow to see hidden icons

3. Find and click the **Tailscale icon** (looks like connected dots)

4. Click **"Log in"**

5. Your browser opens to the Tailscale login page

6. Choose how to log in:
   ```
   ┌─────────────────────────────────────────────────────────────┐
   │                    SIGN IN TO TAILSCALE                      │
   ├─────────────────────────────────────────────────────────────┤
   │                                                              │
   │   Choose your identity provider:                            │
   │                                                              │
   │   ┌────────────────────┐                                    │
   │   │      Google        │  ◄── Use your Google account       │
   │   └────────────────────┘                                    │
   │                                                              │
   │   ┌────────────────────┐                                    │
   │   │     Microsoft      │  ◄── Use your Microsoft account    │
   │   └────────────────────┘                                    │
   │                                                              │
   │   ┌────────────────────┐                                    │
   │   │      GitHub        │  ◄── Use your GitHub account       │
   │   └────────────────────┘                                    │
   │                                                              │
   │   Pick whichever you prefer.                                │
   │   REMEMBER YOUR CHOICE - you'll use the same one on         │
   │   all your other devices!                                   │
   │                                                              │
   └─────────────────────────────────────────────────────────────┘
   ```

7. Complete the login process in your browser

8. You'll see a success message. Close the browser tab.

#### Step 4.4: Verify It's Working

1. Click the Tailscale icon in the system tray

2. You should see:
   - Your machine name
   - "Connected" status
   - An IP address like `100.64.X.X`

3. **Write down this IP address!** This is how other devices will find this computer.

### On iPhone

#### Step 4.5: Download Tailscale

1. Open the **App Store** on your iPhone

2. Tap the **Search** tab (magnifying glass, bottom right)

3. Type **"Tailscale"**

4. Find the app called **"Tailscale"** by Tailscale Inc.
   - Logo looks like connected dots

5. Tap **"Get"** to download

6. Authenticate with Face ID, Touch ID, or password if asked

#### Step 4.6: Set Up Tailscale on iPhone

1. Open the **Tailscale** app

2. Tap **"Get Started"** or **"Log in"**

3. **CRITICAL:** Log in with the **SAME ACCOUNT** you used on Windows!
   - Same Google account
   - Same Microsoft account
   - Same GitHub account

   If you use a different account, your devices won't see each other!

4. You'll be asked to allow VPN configurations:
   ```
   ┌─────────────────────────────────────────────────────────────┐
   │           "Tailscale" Would Like to Add                      │
   │              VPN Configurations                              │
   │                                                              │
   │   All network activity on this iPhone may be filtered       │
   │   or monitored when using VPN.                              │
   │                                                              │
   │              ┌──────────┐  ┌──────────┐                     │
   │              │  Don't   │  │  Allow   │                     │
   │              │  Allow   │  │          │                     │
   │              └──────────┘  └──────────┘                     │
   │                                ▲                             │
   │                                │                             │
   │                          TAP THIS!                           │
   │                                                              │
   └─────────────────────────────────────────────────────────────┘
   ```

   Tap **"Allow"**

5. Enter your iPhone passcode if asked

6. Complete the login process

#### Step 4.7: Verify Connection

1. In the Tailscale app, you should see a list of your devices

2. You should see:
   - Your iPhone (marked as "This device")
   - Your Windows PC (the one you set up earlier)

3. Both should show as **connected**

4. Note the Windows PC's IP address - it should be `100.64.X.X`

#### Step 4.8: Enable the VPN

1. At the top of the Tailscale app, there's a toggle switch

2. Make sure it's **ON** (usually shows as green or blue)

3. When it's on, you'll see a VPN icon in your iPhone's status bar

---

## Part 5: Finding Your Tailscale IP

### Method 1: From the Tailscale App

**On Windows:**
1. Click the Tailscale icon in the system tray
2. Your IP is shown next to your machine name

**On iPhone:**
1. Open the Tailscale app
2. Look at your device in the list
3. The IP is shown under the device name

### Method 2: From Command Line

**On Windows (PowerShell):**
```powershell
tailscale ip -4
```

**On Mac/Linux:**
```bash
tailscale ip -4
```

This returns just the IP address, like:
```
100.64.1.10
```

### Method 3: From the Web Console

1. Go to: **https://login.tailscale.com/admin/machines**

2. Log in if needed

3. You'll see all your devices with their IPs:

```
┌─────────────────────────────────────────────────────────────────┐
│                    TAILSCALE ADMIN CONSOLE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   MACHINES                                                       │
│   ────────                                                       │
│                                                                  │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │ 🖥️  windows-desktop        100.64.1.10        Connected  │  │
│   └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │ 📱  simons-iphone          100.64.1.20        Connected  │  │
│   └──────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Part 6: Using Tailscale

### Connecting Between Devices

Once Tailscale is set up on multiple devices, you can use the Tailscale IP addresses just like they were on the same local network:

```bash
# SSH from your iPhone to your Windows PC
ssh simon@100.64.1.10

# Works from anywhere in the world!
# - At home
# - At a coffee shop
# - In another country
# - On mobile data
```

### The Magic: It Just Works

No matter where your devices are, as long as both have:
1. Internet connection
2. Tailscale running
3. Logged into the same account

They can connect directly!

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│   BEFORE (without Tailscale):                                               │
│   ────────────────────────────                                              │
│                                                                              │
│   "I need to SSH to my home PC..."                                          │
│   "What's my home IP? Let me check a dynamic DNS service..."                │
│   "Wait, did my router's port forwarding break again?"                      │
│   "This hotel WiFi is blocking port 22!"                                    │
│   "Ugh, I'll try later..."                                                  │
│                                                                              │
│   AFTER (with Tailscale):                                                   │
│   ───────────────────────                                                   │
│                                                                              │
│   "I need to SSH to my home PC..."                                          │
│   $ ssh simon@100.64.1.10                                                   │
│   [Connected!]                                                               │
│   "Done."                                                                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Part 7: Tailscale Magic DNS (Optional)

### What is Magic DNS?

Instead of remembering IP addresses, Magic DNS lets you use names:

```
WITHOUT Magic DNS:
──────────────────
ssh simon@100.64.1.10

WITH Magic DNS:
───────────────
ssh simon@windows-desktop
```

### Enabling Magic DNS

1. Go to: **https://login.tailscale.com/admin/dns**

2. Find "Magic DNS" section

3. Toggle it **ON**

4. Now you can use device names instead of IPs!

### How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Your command:                                                 │
│   ssh simon@windows-desktop                                     │
│                                                                  │
│        │                                                        │
│        ▼                                                        │
│                                                                  │
│   Tailscale: "windows-desktop? That's 100.64.1.10!"            │
│                                                                  │
│        │                                                        │
│        ▼                                                        │
│                                                                  │
│   Connection to 100.64.1.10                                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Part 8: Tailscale SSH (Advanced)

### What is Tailscale SSH?

Tailscale can handle SSH authentication for you, so you don't need passwords or SSH keys!

### How It Works

```
TRADITIONAL SSH:
────────────────
1. Set up SSH server
2. Configure firewall
3. Manage SSH keys or passwords
4. Remember which key goes with which server

TAILSCALE SSH:
──────────────
1. Enable Tailscale SSH
2. Done! Authentication happens through Tailscale
```

### Enabling Tailscale SSH

On the machine you want to SSH INTO:

```bash
tailscale set --ssh
```

Now other Tailscale devices can SSH in without password prompts!

---

## Part 9: Security Deep Dive

### What Tailscale Protects

```
┌─────────────────────────────────────────────────────────────────┐
│                    TAILSCALE SECURITY                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ✓ ENCRYPTION                                                  │
│     All traffic between devices is encrypted with WireGuard    │
│     Even on public WiFi, your data is safe                     │
│                                                                  │
│   ✓ AUTHENTICATION                                              │
│     Only devices logged into YOUR account can connect           │
│     No random internet strangers can see your devices          │
│                                                                  │
│   ✓ NO PUBLIC EXPOSURE                                          │
│     Your devices don't have open ports on the internet         │
│     Hackers can't scan and find your machines                  │
│                                                                  │
│   ✓ ZERO TRUST                                                  │
│     Each device authenticates separately                        │
│     Compromising one doesn't automatically compromise others    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### What You Should Still Do

```
✓ Use strong passwords on your devices
✓ Keep your devices and Tailscale updated
✓ Log out of Tailscale on devices you no longer use
✓ Review your connected devices periodically
✓ Don't share your Tailscale account credentials
```

---

## Part 10: Troubleshooting

### Problem: Devices Can't See Each Other

**Check:**
1. Both devices have Tailscale installed and running
2. Both devices are logged into the SAME account
3. Both devices have internet connection

**Solutions:**
- Toggle Tailscale off and on
- Check the Tailscale admin console for both devices
- Make sure you used the same login method (Google, Microsoft, etc.)

### Problem: Connection is Slow

**Possible Causes:**
- One device has poor internet
- They're using relayed connection instead of direct

**Solutions:**
- Check internet speed on both devices
- Tailscale usually establishes direct connections, but sometimes uses relays
- Wait a minute for direct connection to establish

### Problem: "Not Connected" Status

**On Windows:**
1. Click Tailscale icon in system tray
2. If it says "Not connected," click to reconnect
3. You may need to log in again

**On iPhone:**
1. Open Tailscale app
2. Toggle the connection on
3. Make sure VPN is enabled

### Problem: Can't Find Tailscale Icon

**On Windows:**
1. Click the "^" arrow in system tray to see hidden icons
2. If not there, search for "Tailscale" in Start menu
3. Run it to start the program

---

## Part 11: Quick Reference

### Important URLs

| What | URL |
|------|-----|
| Download Tailscale | https://tailscale.com/download |
| Admin Console | https://login.tailscale.com/admin/machines |
| DNS Settings | https://login.tailscale.com/admin/dns |

### Common Commands

```bash
# Check your Tailscale IP
tailscale ip -4

# Check connection status
tailscale status

# Enable Tailscale SSH server
tailscale set --ssh

# Ping another device
tailscale ping device-name

# Log out
tailscale logout
```

### Understanding the IP Addresses

```
Tailscale IP: 100.64.X.X
              ─────┬─────
                   │
     This is a special range reserved for
     Tailscale. These IPs only work within
     your Tailnet.

Real IP: Whatever your ISP assigns
         This changes, Tailscale IP doesn't!
```

---

## Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                    TAILSCALE IN A NUTSHELL                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   WHAT:  Creates a private network connecting your devices      │
│                                                                  │
│   WHY:   • Fixed IP addresses (100.64.x.x)                      │
│          • Works through any firewall                           │
│          • No port forwarding needed                            │
│          • Everything encrypted                                 │
│          • Only your devices can connect                        │
│                                                                  │
│   HOW:   1. Install Tailscale on each device                    │
│          2. Log in with the SAME account                        │
│          3. Use the 100.64.x.x IP to connect!                   │
│                                                                  │
│   REMEMBER:                                                      │
│          • Same account = same network                          │
│          • Different account = can't see each other             │
│          • Works anywhere with internet                         │
│          • Check admin console to see all devices               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## What's Next?

Now that you have Tailscale set up:
- **SSH**: Connect securely to your devices using the Tailscale IPs
- **tmux**: Keep your sessions alive even when you disconnect
- **Termius**: Connect from your phone using a nice mobile app

Tailscale is the foundation that makes everything else easy!
