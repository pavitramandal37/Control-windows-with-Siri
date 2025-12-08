# Siri Laptop Automation
## Complete Setup Guide
**Control Your Windows Laptop with Voice Commands**

*December 2025*
*Created by: Pavitra Mandal*

---

## Table of Contents

- [Introduction](#introduction)
- [How It Works](#how-it-works)
- [Prerequisites](#prerequisites)
- [Part 1: Windows SSH Server Setup](#part-1-windows-ssh-server-setup)
- [Part 2: iPhone Setup with Termius](#part-2-iphone-setup-with-termius)
- [Part 3: Creating Siri Shortcuts](#part-3-creating-siri-shortcuts)
- [Part 4: Additional Useful Shortcuts](#part-4-additional-useful-shortcuts)
- [Troubleshooting](#troubleshooting)
- [Security Considerations](#security-considerations)
- [Frequently Asked Questions](#frequently-asked-questions)
- [Conclusion](#conclusion)

---

## Introduction

This guide will help you set up voice-controlled automation for your Windows laptop using Siri on your iPhone. Once configured, you'll be able to:

✓ Shut down your laptop by saying "Hey Siri, shut down laptop"
✓ Hibernate your laptop by saying "Hey Siri, hibernate laptop"
✓ Lock, sleep, restart, and more using custom voice commands
✓ Control your laptop from anywhere on your home WiFi network

---

## How It Works

The system uses SSH (Secure Shell) to enable secure remote communication:

1. You speak a command to Siri on your iPhone
2. Siri triggers a shortcut that sends an SSH command over your WiFi
3. Your laptop's SSH server receives and authenticates the command
4. The command executes (e.g., shutdown, hibernate, lock)
5. Your laptop performs the requested action

---

## Prerequisites

Before you begin, ensure you have:

- Windows 10/11 laptop connected to WiFi
- iPhone with Siri enabled
- Both devices connected to the same WiFi network
- Administrator access on your Windows laptop
- Termius app installed on iPhone (free from App Store)
- 30-45 minutes for complete setup

---

## Part 1: Windows SSH Server Setup

### Step 1: Remove Old SSH Installation (If Any)

**Open PowerShell as Administrator:**
- Press `Windows Key + X`
- Select "Windows PowerShell (Admin)" or "Terminal (Admin)"

**Run these commands:**

```powershell
Remove-Item C:\ProgramData\ssh\sshd_config
Remove-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

**⚠️ Important:** You will see "RestartNeeded: True". **Restart your laptop** before continuing.

---

### Step 2: Create SSH User Account

After restarting, **open PowerShell as Administrator** again.

**Create a dedicated SSH user** (recommended for security):
- Username: `sshuser` (you can keep your own username)
- Password: `YourPassword123` (you can choose your own password)

```powershell
net user sshuser YourPassword123 /add
```

> **Note:** Replace `YourPassword123` with a password of your choice. Remember this password - you'll need it later.

**Verify the user was created:**

```powershell
net user sshuser
```

**Give the user shutdown permissions:**

```powershell
net localgroup "Remote Management Users" sshuser /add
net localgroup "Administrators" sshuser /add
```

---

### Step 3: Configure SSH Server

**Create the SSH configuration file:**

```powershell
notepad C:\ProgramData\ssh\sshd_config
```

A blank Notepad window will open. **Copy and paste this configuration:**

```
# SSH Server Configuration
Port 22
PasswordAuthentication yes
PubkeyAuthentication yes
PermitRootLogin no
AllowUsers sshuser

# Subsystem for file transfer
Subsystem sftp sftp-server.exe
```

**✓ Save the file:** Press `Ctrl+S`, make sure "Save as type" is set to **"All Files (*.*)"**, then click Save. Close Notepad.

---

### Step 4: Set File Permissions

Run these commands in PowerShell (as Administrator):

```powershell
icacls C:\ProgramData\ssh\sshd_config /inheritance:r
icacls C:\ProgramData\ssh\sshd_config /grant "BUILTIN\Administrators:F"
icacls C:\ProgramData\ssh\sshd_config /grant "NT AUTHORITY\SYSTEM:F"
```

**Expected output:** "Successfully processed 1 files"

---

### Step 5: Disable Administrator Key Requirement

Run this command:

```powershell
Rename-Item C:\ProgramData\ssh\administrators_authorized_keys C:\ProgramData\ssh\administrators_authorized_keys.disabled -ErrorAction SilentlyContinue
```

---

### Step 6: Start SSH Service

Start the SSH server and configure it to start automatically:

```powershell
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'
```

---

### Step 7: Configure Windows Firewall

Allow SSH through the firewall:

```powershell
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22 -ErrorAction SilentlyContinue
```

---

### Step 8: Get Your Laptop's Hostname

Find your laptop's network name:

```powershell
hostname
```

**Write down the output** (e.g., "DESKTOP-XXXXXX" or "MyLaptop"). You'll need this for iPhone setup.

---

### Step 9: Test SSH Locally

**Open a NEW PowerShell window** (regular, NOT as Administrator).

**Test the SSH connection:**

```powershell
ssh sshuser@localhost
```

**What to expect:**
- **First time:** It will ask about authenticity → Type `yes` and press Enter
- Enter your password when prompted
- You should see a prompt like: `sshuser@DESKTOP-XXXXXX C:\Users\sshuser>`

**Test a shutdown command** (will schedule shutdown in 10 seconds):

```cmd
shutdown /s /t 10
```

**Immediately cancel it:**

```cmd
shutdown /a
```

**✓ Success:** If both commands work without "Access Denied" errors, your SSH server is configured correctly!

---

## Part 2: iPhone Setup with Termius

### Step 10: Install Termius App

On your iPhone:
1. Open the **App Store**
2. Search for **"Termius"**
3. Download and install the free version
4. Open Termius

---

### Step 11: Add Your Laptop as a Host

In Termius app:

1. Tap **"Connections"** at the bottom
2. Tap **"Add a host"** (blue computer chip icon)
3. Fill in the details:
   - **Alias:** My Laptop (or any name you like)
   - **Address:** YourHostname.local (e.g., `DESKTOP-XXXXXX.local`)
   - **Port:** 22
   - **Username:** sshuser
4. Tap **"Save"**

---

### Step 12: Test Connection from iPhone

1. Tap on **"My Laptop"** to connect
2. **First connection:** Tap "Yes" or "Continue" when asked about authenticity
3. Enter your password when prompted
4. You should see a command prompt on your iPhone screen

**Test a command** - type:

```cmd
shutdown /s /t 10
```

**Then cancel it:**

```cmd
shutdown /a
```

**✓ Success:** If this works, you're ready for Siri shortcuts!

---

## Part 3: Creating Siri Shortcuts

### Step 13: Create Shutdown Shortcut

On your iPhone, open the **Shortcuts** app

1. Tap the **"+"** button (top right corner)
2. Tap **"Add Action"**
3. In the search bar, type: **"Run Script Over SSH"**
4. Tap on **"Run Script Over SSH"** to add it
5. Configure the action by tapping each field:
   - **Host:** YourHostname.local (e.g., `DESKTOP-XXXXXX.local`)
   - **Port:** 22
   - **User:** sshuser
   - **Authentication:** Password
   - **Password:** YourPassword123 (the password you set earlier)
   - **Script:** `shutdown /s /t 0`
6. Tap the shortcut name at the top (says "New Shortcut")
7. Rename it to: **"Shut Down Laptop"**
8. Tap **"Done"**

---

### Step 14: Create Hibernate Shortcut

Repeat the same steps as above, but with these changes:

- **Script:** `shutdown /h` (instead of `/s /t 0`)
- **Name:** "Hibernate Laptop"

---

### Step 15: Test Your Siri Shortcuts

Say to Siri: **"Hey Siri, shut down laptop"**
- Your laptop should begin shutting down!

Say to Siri: **"Hey Siri, hibernate laptop"**
- Your laptop should hibernate!

---

## Part 4: Additional Useful Shortcuts

Once you have the basic shortcuts working, you can create many more! Here are some popular options:

### Lock Laptop
- **Command:** `rundll32.exe user32.dll,LockWorkStation`
- **Say:** "Hey Siri, lock laptop"
- **Use case:** Quickly lock your laptop when stepping away

### Sleep Laptop
- **Command:** `rundll32.exe powrprof.dll,SetSuspendState 0,1,0`
- **Say:** "Hey Siri, sleep laptop"
- **Use case:** Put laptop to sleep to save battery

### Restart Laptop
- **Command:** `shutdown /r /t 0`
- **Say:** "Hey Siri, restart laptop"
- **Use case:** Quick restart for troubleshooting

### Check Battery Status
- **Command:** `WMIC Path Win32_Battery Get EstimatedChargeRemaining`
- **Say:** "Hey Siri, check laptop battery"
- **Use case:** Know when to charge your laptop remotely

---

## Troubleshooting

### "Connection could not be established" Error

**Possible causes and solutions:**

- **Both devices not on same WiFi:** Verify iPhone and laptop are connected to the same network
- **Laptop IP changed:** Check current IP with `ipconfig` and update Termius
- **SSH service not running:** Open PowerShell as Admin and run `Start-Service sshd`
- **Firewall blocking:** Re-run the firewall command from Step 7

---

### "Permission Denied" Error

- **Wrong password:** Double-check you're using the correct password
- **Wrong username:** Verify you're using `sshuser` not your personal account
- **SSH config issue:** Re-run Step 3 to recreate the config file

---

### "Access Denied" When Running Commands

**User doesn't have admin rights:** Re-run this command:

```powershell
net localgroup "Administrators" sshuser /add
```

Then restart SSH service:

```powershell
Restart-Service sshd
```

---

### Hostname Not Resolving (.local doesn't work)

**Use IP address instead:**
- Find your laptop's IP with `ipconfig | findstr IPv4`
- **Update Termius and shortcuts:** Replace `hostname.local` with the IP address (e.g., `192.168.1.100`)

---

## Security Considerations

### Password Security

- Use a **strong password** for the sshuser account
- Mix uppercase, lowercase, numbers, and symbols
- Make it at least 12 characters long
- Never share your password

### Network Security

- **Only works on your home WiFi:** SSH connection only works when both devices are on the same network
- **Secure your WiFi:** Use WPA3 or WPA2 encryption with a strong password
- **Don't expose port 22 to the internet:** Never configure port forwarding for SSH on your router

### Advanced Security (Optional)

For maximum security, consider:

- SSH key authentication instead of passwords
- Changing the default SSH port (22) to a custom port
- Setting up fail2ban to block brute force attempts

---

## Frequently Asked Questions

### Q: Can I turn ON my laptop with Siri?

**A:** Not with this SSH method. SSH requires the laptop to already be running. However, you can use Wake-on-LAN (WOL) to wake a sleeping or hibernating laptop. This requires additional setup and works best with an Ethernet connection.

---

### Q: Will this work from outside my home?

**A:** No, not by default. This setup only works when your iPhone and laptop are on the same WiFi network. To control from outside, you would need to set up a VPN or configure port forwarding (not recommended for security reasons).

---

### Q: What happens if my laptop's IP address changes?

**A:** If using `hostname.local`, this shouldn't be an issue. If using an IP address, you'll need to update it in Termius and your Siri shortcuts. To prevent this, set a static IP on your laptop or configure DHCP reservation in your router.

---

### Q: Is this secure?

**A:** Yes, when properly configured. SSH is an industry-standard secure protocol. As long as you use a strong password, keep SSH limited to your local network, and don't expose it to the internet, this setup is secure for home use.

---

### Q: Can I use this with Android instead of iPhone?

**A:** Yes! Install Termius on Android and use Google Assistant instead of Siri. The setup process is similar, but you'll create Google Assistant routines instead of Siri shortcuts.

---

### Q: Will this drain my laptop's battery?

**A:** The SSH server uses minimal resources when idle. Battery impact is negligible - less than 1% per hour.

---

### Q: Can multiple people control the laptop?

**A:** Yes! Anyone on your WiFi network with the sshuser password can set up shortcuts on their iPhone or install Termius. You can also create multiple SSH users with different passwords for different people.

---

## Conclusion

**Congratulations!** You've successfully set up voice-controlled automation for your Windows laptop using Siri. You can now:

✓ Shut down your laptop with "Hey Siri, shut down laptop"
✓ Hibernate with "Hey Siri, hibernate laptop"
✓ Create unlimited custom shortcuts for other commands
✓ Control your laptop from anywhere on your home WiFi

### Next Steps:

- Explore creating more shortcuts from Part 4
- Customize command names to your preference
- Share this guide with friends who want the same setup
- Consider setting up Wake-on-LAN for even more control

**⚠️ Important:** Keep your passwords secure, only use this on trusted networks, and never expose SSH to the internet!

**Enjoy your automated laptop control!**

---

*Created with ❤️ by Pavitra Mandal*
*December 2025*
