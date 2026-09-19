# 60-Second Manual Enumeration Checklist (Quick Wins)

Always perform these manual checks prior to running automated tools like LinPEAS. Manual enumeration prevents unnecessary log noise, avoids blind spots, and identifies high-probability LPE vectors instantly.

---

## 1. User Privileges & Sudo Rights

Check current user context, groups, and passwordless sudo capabilities.

```bash
# Display effective UID, GID, and group memberships
id

# List allowed (and forbidden) commands for the invoking user
sudo -l

Target Groups to Watch For: docker, lxd, disk, shadow, adm, libvirt.

2. System Environment & OS Fingerprinting
Identify kernel release, OS distribution, and environment variables for PATH manipulation.

Bash
# Operating system release and kernel details
uname -a
cat /etc/os-release 2>/dev/null || cat /etc/issue

# Environment variables (Check PATH for writeable directories or sensitive tokens)
env
3. SUID / SGID Executables
Locate binaries with the SUID bit set that execute with root privileges.

Bash
# Find all SUID binaries, suppressing permission errors
find / -perm -4000 -type f 2>/dev/null
Analysis: Compare discovered binaries against GTFOBins or look for unusual custom binaries in /tmp, /opt, or /var/www/.

4. Active Processes & Local Services
Identify processes running as root and internal services bound to 127.0.0.1.

Bash
# Inspect processes running under root context
ps aux | grep root

# Display listening TCP/UDP ports and associated processes
ss -tulpn
5. Scheduled Tasks (Cron Jobs)
Inspect cron configurations for scripts executed by root with insecure file permissions.

Bash
# System-wide crontab and daily/hourly jobs
cat /etc/crontab /etc/cron.d/* 2>/dev/null

# User crontabs
ls -la /var/spool/cron/crontabs/ 2>/dev/null
