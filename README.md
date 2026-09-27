# 🛡️ Blue Team Cheat Sheet

Quick-reference hardening/patching commands for a cybersecurity competition. No AI, no personal devices allowed on-site — this repo is the reference.

- [`windows.md`](windows.md) — Windows patching, user/firewall/service hardening, persistence checks, logging
- [`linux.md`](linux.md) — Linux patching, user/firewall/service hardening, persistence checks, logging

## ⚡ Order of Operations (both OSes)
1. Change all passwords
2. Patch the OS
3. Enable/lock down the firewall
4. Audit users & admin/sudo groups
5. Kill unauthorized services/processes
6. Check persistence mechanisms
7. Check open ports
8. Enable logging/auditing
