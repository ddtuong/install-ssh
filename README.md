# 🔐 SSH Server on Ubuntu

This guide explains how to install and configure an **OpenSSH Server** on Ubuntu, verify the SSH service, open port `22`, configure UFW, find the Ubuntu VM's IP address, and connect to it remotely from Windows.

---

## 1. What is SSH?

**SSH (Secure Shell)** is a network protocol that allows you to securely access and control a remote machine.

Example:

```text
Windows PC
    │
    │ SSH
    │ Port 22
    ▼
Ubuntu VM
    │
    └── SSH Server
        └── sshd
```

Instead of opening a terminal directly inside the Ubuntu VM, you can connect from Windows:

```bash
ssh username@ip_address
```

Example:

```bash
ssh tuong@192.168.1.100
```

---

# 2. Install OpenSSH Server

Update the package list:

```bash
sudo apt update
```

Install the SSH server:

```bash
sudo apt install openssh-server -y
```

The package provides the SSH daemon:

```text
sshd
```

---

# 3. Check SSH Service Status

Check whether SSH is running:

```bash
sudo systemctl status ssh
```

You should see something similar to:

```text
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded
     Active: active (running)
```

The important part is:

```text
Active: active (running)
```

This means the SSH server is currently running.

---

# 4. Start SSH Server

If SSH is not running:

```bash
sudo systemctl start ssh
```

Check again:

```bash
sudo systemctl status ssh
```

---

# 5. Enable SSH at Boot

To automatically start SSH whenever Ubuntu boots:

```bash
sudo systemctl enable ssh
```

You can combine **enable + start**:

```bash
sudo systemctl enable --now ssh
```

Verify:

```bash
sudo systemctl status ssh
```

---

# 6. Check SSH Port

By default, SSH uses:

```text
Port 22
```

Check whether Ubuntu is listening on port `22`:

```bash
sudo ss -tlnp | grep :22
```

Example:

```text
LISTEN 0 128 0.0.0.0:22 0.0.0.0:*
LISTEN 0 128    [::]:22    [::]:*
```

This means SSH is listening on port `22`.

### Understanding the command

```bash
ss -tlnp
```

| Option | Meaning                      |
| ------ | ---------------------------- |
| `-t`   | TCP                          |
| `-l`   | Listening sockets            |
| `-n`   | Show numeric addresses/ports |
| `-p`   | Show process information     |

---

# 7. Check SSH Configuration

The main SSH server configuration file is:

```text
/etc/ssh/sshd_config
```

View it:

```bash
sudo nano /etc/ssh/sshd_config
```

The default SSH port is:

```text
Port 22
```

You can check the effective configuration with:

```bash
sudo sshd -T | grep port
```

Expected:

```text
port 22
```

> **Important:** Do not change the SSH port unless you understand the consequences. If you change it, remember to update UFW and your SSH client command.

For example, if SSH uses port `2222`:

```bash
ssh -p 2222 username@ip_address
```

---

# 8. Configure UFW

**UFW (Uncomplicated Firewall)** is Ubuntu's firewall management tool.

Check its status:

```bash
sudo ufw status
```

Example:

```text
Status: inactive
```

or:

```text
Status: active
```

---

## 8.1 Allow SSH

The easiest way to allow SSH:

```bash
sudo ufw allow ssh
```

This normally creates a rule for:

```text
TCP port 22
```

You can also explicitly specify:

```bash
sudo ufw allow 22/tcp
```

---

## 8.2 Enable UFW

If UFW is currently inactive:

```bash
sudo ufw enable
```

Then check:

```bash
sudo ufw status
```

Example:

```text
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
```

### Recommended sequence

Before enabling UFW over a remote SSH connection:

```bash
sudo ufw allow ssh
```

Then:

```bash
sudo ufw enable
```

This helps avoid accidentally blocking your SSH connection.

---

# 9. Find Ubuntu IP Address

To find the IP address of the Ubuntu machine:

```bash
hostname -I
```

Example:

```text
192.168.1.100
```

You can also use:

```bash
ip addr
```

Look for an interface such as:

```text
ens33
eth0
```

with an address similar to:

```text
inet 192.168.1.100/24
```

---

# 10. Test SSH Locally

Before connecting from Windows, test SSH inside Ubuntu itself:

```bash
ssh localhost
```

or:

```bash
ssh username@localhost
```

Example:

```bash
ssh tuong@localhost
```

If successful, you should receive a prompt similar to:

```text
tuong@tuong-VMware-Virtual-Platform:~$
```

Exit the SSH session:

```bash
exit
```

---

# 11. Connect from Windows

Open **PowerShell** on Windows.

Use:

```powershell
ssh username@ubuntu_ip
```

Example:

```powershell
ssh tuong@192.168.1.100
```

The first time you connect, you may see:

```text
The authenticity of host '192.168.1.100' can't be established.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Enter:

```text
yes
```

Then enter the Ubuntu user's password.

---

# 12. SSH Connection Flow

The complete connection looks like:

```text
┌───────────────────────┐
│    Windows 11         │
│                       │
│ PowerShell            │
│ ssh tuong@192.168...  │
└───────────┬───────────┘
            │
            │ TCP :22
            ▼
┌───────────────────────┐
│    VMware Network     │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│     Ubuntu VM         │
│                       │
│      UFW              │
│       │               │
│       ▼               │
│    TCP :22            │
│       │               │
│       ▼               │
│      sshd             │
└───────────────────────┘
```

For the connection to work, all of these must be correct:

```text
SSH server running
        +
Port 22 listening
        +
UFW allows port 22
        +
VM has reachable IP
        +
VMware networking allows connection
```

---

# 13. VMware Network Mode

If Windows cannot connect to the Ubuntu VM, check the VMware network configuration.

Common VMware network modes:

| Mode      | Description                                          |
| --------- | ---------------------------------------------------- |
| NAT       | VM accesses the network through the host             |
| Bridged   | VM appears as another device on the physical network |
| Host-only | VM communicates with the host/private network        |

## NAT

Typical architecture:

```text
Internet
   │
   ▼
Windows Host
   │
   │ NAT
   ▼
Ubuntu VM
```

The VM may have an IP such as:

```text
192.168.xxx.xxx
```

Depending on the VMware NAT configuration, direct inbound connections from Windows may require appropriate networking/port forwarding.

## Bridged

```text
Router
 ├── Windows
 └── Ubuntu VM
```

The Ubuntu VM gets its own address on the same network.

For example:

```text
Windows: 192.168.1.10
Ubuntu:  192.168.1.100
```

Then Windows can typically connect directly:

```powershell
ssh tuong@192.168.1.100
```

---

# 14. Troubleshooting

## Check SSH service

```bash
sudo systemctl status ssh
```

If stopped:

```bash
sudo systemctl start ssh
```

---

## Check port 22

```bash
sudo ss -tlnp | grep :22
```

No output usually means nothing is listening on port `22`.

---

## Check UFW

```bash
sudo ufw status
```

You should have a rule such as:

```text
22/tcp ALLOW
```

If not:

```bash
sudo ufw allow 22/tcp
```

---

## Check SSH process

```bash
ps aux | grep sshd
```

You should see the SSH daemon:

```text
/usr/sbin/sshd
```

---

## Check IP

```bash
hostname -I
```

Make sure you are using the correct Ubuntu VM IP from Windows.

---

## Test connectivity from Windows

In PowerShell:

```powershell
ping 192.168.1.100
```

Then:

```powershell
ssh tuong@192.168.1.100
```

If `ping` works but SSH does not, investigate:

```text
sshd
Port 22
UFW
```

If `ping` does not work, investigate:

```text
VMware networking
IP address
Network adapter
```

---

# 15. Useful SSH Commands

### Connect

```bash
ssh username@ip_address
```

### Connect using a specific port

```bash
ssh -p 2222 username@ip_address
```

### Exit SSH

```bash
exit
```

or:

```text
Ctrl + D
```

### Copy files to Ubuntu

From Windows PowerShell:

```powershell
scp file.txt tuong@192.168.1.100:/home/tuong/
```

### Copy a directory

```powershell
scp -r my_project tuong@192.168.1.100:/home/tuong/
```

---

# 16. Essential Commands Cheat Sheet

| Task            | Command                              |
| --------------- | ------------------------------------ |
| Update packages | `sudo apt update`                    |
| Install SSH     | `sudo apt install openssh-server -y` |
| Start SSH       | `sudo systemctl start ssh`           |
| Stop SSH        | `sudo systemctl stop ssh`            |
| Restart SSH     | `sudo systemctl restart ssh`         |
| Enable at boot  | `sudo systemctl enable ssh`          |
| Enable + start  | `sudo systemctl enable --now ssh`    |
| Check status    | `sudo systemctl status ssh`          |
| Check port      | `sudo ss -tlnp \| grep :22`          |
| Check IP        | `hostname -I`                        |
| UFW status      | `sudo ufw status`                    |
| Allow SSH       | `sudo ufw allow ssh`                 |
| Allow port 22   | `sudo ufw allow 22/tcp`              |
| Enable UFW      | `sudo ufw enable`                    |
| Local SSH test  | `ssh localhost`                      |
| Remote SSH      | `ssh user@ip`                        |
| Exit SSH        | `exit`                               |

---

# 17. Complete Setup — Quick Version

If you just want the essential setup:

```bash
# 1. Update packages
sudo apt update

# 2. Install OpenSSH Server
sudo apt install openssh-server -y

# 3. Enable and start SSH
sudo systemctl enable --now ssh

# 4. Check SSH status
sudo systemctl status ssh

# 5. Check port 22
sudo ss -tlnp | grep :22

# 6. Allow SSH through UFW
sudo ufw allow ssh

# 7. Enable firewall
sudo ufw enable

# 8. Check firewall
sudo ufw status

# 9. Find Ubuntu IP
hostname -I
```

Then from Windows:

```powershell
ssh tuong@<UBUNTU_IP>
```

Example:

```powershell
ssh tuong@192.168.1.100
```

---

# 18. Mental Model

Remember the SSH setup as four layers:

```text
              SSH Connection
                    │
                    ▼
          ┌──────────────────┐
          │ VMware Networking│
          └────────┬─────────┘
                   │
                   ▼
          ┌──────────────────┐
          │       UFW        │
          │   Allow :22/tcp  │
          └────────┬─────────┘
                   │
                   ▼
          ┌──────────────────┐
          │      Port 22     │
          │    LISTENING     │
          └────────┬─────────┘
                   │
                   ▼
          ┌──────────────────┐
          │      sshd        │
          │  SSH Server      │
          └──────────────────┘
```

When SSH does not work, check from bottom to top:

```text
1. Is sshd running?
        ↓
2. Is port 22 listening?
        ↓
3. Does UFW allow port 22?
        ↓
4. Does Ubuntu have the correct IP?
        ↓
5. Can Windows reach the VM?
        ↓
6. Is VMware networking configured correctly?
```

---

## 19. Security Notes

For a development VM, the default configuration is usually sufficient.

For production servers, additional security measures should be considered:

* SSH key authentication
* Disable password authentication where appropriate
* Disable direct root login
* Restrict SSH access by source IP where possible
* Use a firewall
* Keep OpenSSH and Ubuntu updated
* Monitor authentication logs
* Avoid exposing SSH unnecessarily to the public Internet

SSH logs can be inspected with:

```bash
sudo journalctl -u ssh
```

or:

```bash
sudo tail -f /var/log/auth.log
```

---

# 20. Final Checklist

Before using SSH from Windows, verify:

```text
[ ] openssh-server installed
[ ] ssh.service is active
[ ] SSH starts automatically
[ ] Port 22 is listening
[ ] UFW allows SSH
[ ] Ubuntu has a reachable IP address
[ ] VMware network is configured correctly
[ ] Windows can reach the Ubuntu VM
```

The basic command flow is:

```bash
sudo apt update
sudo apt install openssh-server -y

sudo systemctl enable --now ssh

sudo systemctl status ssh

sudo ss -tlnp | grep :22

sudo ufw allow ssh
sudo ufw enable
sudo ufw status

hostname -I
```

Then from Windows:

```powershell
ssh <username>@<ubuntu-ip>
```

Example:

```powershell
ssh tuong@192.168.1.100
```

**Result:** You can now manage the Ubuntu VM remotely from Windows through a secure SSH connection.
