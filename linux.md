# 🐧 Linux — Hardening & Patching Cheat Sheet

## ⚡ Order of Operations
1. Change all passwords
2. Patch the OS
3. Enable/lock down the firewall
4. Audit users & sudoers
5. Kill unauthorized services/processes
6. Check persistence (cron, rc.local)
7. Check open ports
8. Enable logging/auditing

---

## 1. Patching / Updates
```bash
# Debian/Ubuntu
sudo apt update && sudo apt upgrade -y
sudo apt full-upgrade -y

# RHEL/CentOS/Fedora
sudo yum update -y
# or
sudo dnf update -y
```

## 2. User & Account Hardening
```bash
cat /etc/passwd

# Users with UID 0 — should ONLY be root
awk -F: '$3 == 0 {print $1}' /etc/passwd

sudo passwd -l sketchyuser        # lock account
sudo userdel -r sketchyuser       # delete account

sudo cat /etc/sudoers
sudo cat /etc/sudoers.d/*

sudo deluser sketchyuser sudo        # Debian
sudo gpasswd -d sketchyuser wheel    # RHEL

sudo chage -M 30 -m 1 username
sudo passwd -e username

# Accounts with empty passwords
sudo awk -F: '($2 == "") {print $1}' /etc/shadow
```

## 3. Firewall
```bash
# UFW (Debian/Ubuntu)
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw status verbose

# firewalld (RHEL/CentOS)
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo firewall-cmd --set-default-zone=drop
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload

# iptables (raw)
sudo iptables -L -n -v
sudo iptables -P INPUT DROP
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

## 4. Services & Processes
```bash
systemctl list-units --type=service --state=running

sudo systemctl stop telnet
sudo systemctl disable telnet
sudo systemctl stop rsh
sudo systemctl disable rsh

sudo ss -tulnp
sudo netstat -tulnp

ps aux --sort=-%cpu | head -20
sudo kill -9 <PID>
```

## 5. SSH Hardening
```bash
sudo nano /etc/ssh/sshd_config
# Recommended:
#   PermitRootLogin no
#   PasswordAuthentication no   (if using keys)
#   Protocol 2
#   MaxAuthTries 3

sudo systemctl restart sshd
```

## 6. Cron & Persistence Checks
```bash
sudo crontab -l -u root
ls -la /etc/cron.d/ /etc/cron.daily/ /etc/cron.hourly/
cat /etc/crontab
systemctl list-unit-files --type=service | grep enabled
cat /etc/rc.local
```

## 7. File Permissions & SUID Checks
```bash
find / -perm -4000 -type f 2>/dev/null   # SUID
find / -perm -2000 -type f 2>/dev/null   # SGID
find / -xdev -type f -perm -0002 2>/dev/null   # world-writable
```

## 8. Logging / Auditing
```bash
sudo grep "Failed password" /var/log/auth.log     # Debian
sudo grep "Failed password" /var/log/secure        # RHEL
sudo tail -f /var/log/auth.log
```

---

## 📋 Quick Reference
| Step | Command |
|------|---------|
| Change password | `passwd username` |
| Patch OS | `apt update && apt upgrade -y` |
| Enable firewall | `ufw enable` |
| Audit root-level users | `awk -F: '$3==0' /etc/passwd` |
| Kill unknown service | `systemctl stop <svc>` |
| Check persistence | crontab + rc.local |
| Check open ports | `ss -tulnp` |
