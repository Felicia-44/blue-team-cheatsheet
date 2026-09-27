# 🪟 Windows — Hardening & Patching Cheat Sheet

## ⚡ Order of Operations
1. Change all passwords
2. Patch the OS
3. Enable/lock down the firewall
4. Audit users & admin groups
5. Kill unauthorized services/processes
6. Check persistence (scheduled tasks, registry)
7. Check open ports
8. Enable logging/auditing

---

## 1. Patching / Updates
```powershell
# Check current update status
Get-HotFix | Sort-Object InstalledOn -Descending

# Install all available updates (if PSWindowsUpdate module available)
Install-Module PSWindowsUpdate -Force
Import-Module PSWindowsUpdate
Get-WindowsUpdate
Install-WindowsUpdate -AcceptAll -AutoReboot

# Built-in, no internet-dependent module
wuauclt /detectnow /updatenow
usoclient StartScan
```

## 2. User & Account Hardening
```powershell
# List all local users
Get-LocalUser

# Disable / remove a suspicious account
Disable-LocalUser -Name "sketchyuser"
Remove-LocalUser -Name "sketchyuser"

# Strong password policy
net accounts /minpwlen:14 /maxpwage:30 /minpwage:1 /uniquepw:5

# Audit administrators group
Get-LocalGroupMember -Group "Administrators"
Remove-LocalGroupMember -Group "Administrators" -Member "sketchyuser"

# Disable Guest account
net user guest /active:no

# Lockout policy
net accounts /lockoutthreshold:5 /lockoutduration:30
```

## 3. Firewall
```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True
Set-NetFirewallProfile -DefaultInboundAction Block
Get-NetFirewallRule | Where-Object {$_.Enabled -eq "True"}

# Block a known bad port (example: reverse shell on 4444)
New-NetFirewallRule -DisplayName "Block4444" -Direction Inbound -LocalPort 4444 -Protocol TCP -Action Block
```

## 4. Services & Processes
```powershell
Get-Service | Where-Object {$_.Status -eq "Running"}

# Disable risky services
Stop-Service -Name "TlntSvr" -Force
Set-Service -Name "TlntSvr" -StartupType Disabled
Stop-Service -Name "RemoteRegistry" -Force
Set-Service -Name "RemoteRegistry" -StartupType Disabled

Get-Process | Sort-Object CPU -Descending
Stop-Process -Name "suspicious.exe" -Force
```

## 5. Network / Ports
```powershell
netstat -ano | findstr LISTEN
Get-Process -Id <PID>
Get-NetTCPConnection -State Established
```

## 6. Scheduled Tasks (persistence)
```powershell
Get-ScheduledTask | Where-Object {$_.State -eq "Ready"}
Get-ScheduledTask | Format-Table TaskName, TaskPath, State
Unregister-ScheduledTask -TaskName "backdoor_task" -Confirm:$false
```

## 7. Registry Autoruns (persistence)
```powershell
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
```

## 8. Logging / Auditing
```powershell
auditpol /set /category:"Logon/Logoff" /success:enable /failure:enable
Get-WinEvent -LogName Security -MaxEvents 50 | Where-Object {$_.Id -eq 4625}
```

---

## 📋 Quick Reference
| Step | Command |
|------|---------|
| Change password | `net user <user> *` |
| Patch OS | `Install-WindowsUpdate -AcceptAll` |
| Enable firewall | `Set-NetFirewallProfile -Enabled True` |
| Audit admins | `Get-LocalGroupMember -Group Administrators` |
| Kill unknown service | `Stop-Service -Name <svc>` |
| Check persistence | Scheduled Tasks + Registry Run keys |
| Check open ports | `netstat -ano` |
