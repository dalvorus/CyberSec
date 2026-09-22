# 60-Second Manual Enumeration Checklist (Quick Wins)

Always perform these manual checks prior to running automated tools like LinPEAS. Manual enumeration prevents unnecessary log noise, avoids blind spots, and teaches you to spot system anomalies instantly.

---

## 1. User Privileges & Sudo Rights

Check current user context, group memberships, and passwordless sudo capabilities.

```bash
# Display effective UID, GID, and group memberships
id

# List allowed (and forbidden) commands for the invoking user
sudo -l
🔍 Baseline vs. Anomaly
Standard Output: uid=1000(user) gid=1000(user) groups=1000(user),27(sudo). For sudo -l, a standard user usually sees (ALL : ALL) ALL (requires password) or Matching Defaults entries.

🔴 Anomalies to Spot:

Membership in dangerous groups: docker, lxd, lxc, disk, shadow, adm.

sudo -l showing specific binaries with NOPASSWD: (e.g., (root) NOPASSWD: /usr/bin/vim or /usr/bin/find). Check these instantly on GTFOBins.

2. System Environment & OS Fingerprinting
Identify kernel release, OS distribution, and environment variables for PATH manipulation.

Bash
# Operating system release and kernel details
uname -a
cat /etc/os-release 2>/dev/null || cat /etc/issue

# Environment variables (Check PATH for writeable directories)
env
🔍 Baseline vs. Anomaly
Standard Output: PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin. Kernel version matching standard release notes (e.g., Linux 5.x/6.x).

🔴 Anomalies to Spot:

Outdated kernel versions vulnerable to known exploits (e.g., Dirty COW, OverlayFS CVEs).

Current working directory . or writable user directories (like /tmp or /home/user) placed at the beginning of $PATH.

Hardcoded tokens, API keys, or database credentials exposed inside env output.

3. SUID / SGID Executables
Locate binaries with the SUID bit set that execute with elevated root privileges.

Bash
# Find all SUID binaries, suppressing permission errors
find / -perm -4000 -type f 2>/dev/null
🔍 Baseline vs. Anomaly
Standard Output: Standard core system utilities (~20-30 binaries): /usr/bin/sudo, /usr/bin/passwd, /usr/bin/su, /usr/bin/chfn, /usr/bin/newgrp, /bin/ping, /bin/mount.

🔴 Anomalies to Spot:

Any non-standard binary with SUID set (e.g., python3, bash, find, nmap, cp, vim).

Custom binaries located in non-standard directories like /tmp, /opt, /var/www, or /home/*.

4. Active Processes & Local Services
Identify processes running as root and internal services bound to 127.0.0.1.

Bash
# Inspect processes running under root context
ps aux | grep root

# Display listening TCP/UDP ports and associated processes
ss -tulpn
🔍 Baseline vs. Anomaly
Standard Output: Kernel threads [kworker/...], /sbin/init, /usr/sbin/sshd, /usr/sbin/cron. Listening ports limited to port 22 (SSH) or standard public web services.

🔴 Anomalies to Spot:

Root executing custom scripts or binaries from writable paths (e.g., root /bin/bash /opt/scripts/backup.sh).

Internal services listening on 127.0.0.1 (e.g., MySQL on 3306, Redis on 6379, local web apps on 8080 or 3000). These bypass external firewalls and are prime LPE targets.

5. Scheduled Tasks (Cron Jobs)
Inspect cron configurations for scripts executed by root with insecure permissions.

Bash
# System-wide crontab and daily/hourly jobs
cat /etc/crontab /etc/cron.d/* 2>/dev/null

# User crontabs
ls -la /var/spool/cron/crontabs/ 2>/dev/null
🔍 Baseline vs. Anomaly
Standard Output: Default crontabs containing only system maintenance scripts executing standard cron.daily, cron.hourly, etc.

🔴 Anomalies to Spot:

Active cron jobs pointing to custom shell/python scripts in /tmp, /opt, or /var/tmp.

Root cron jobs executing binaries using wildcards * (vulnerable to wildcard injection attacks).

Cron scripts executed by root that are writable by standard users (ls -la /path/to/script.sh).

6. System Services & Timers Inspection
Check custom systemd service definitions, running unit files, and systemd timers.

Bash
# List active systemd services and unit files
systemctl list-units --type=service --state=running 2>/dev/null

# Find systemd timers (modern cron alternative running as root)
systemctl list-timers --all 2>/dev/null

# Look for non-standard or user-writable service files
find /etc/systemd/system/ /lib/systemd/system/ -writable -type f 2>/dev/null
🔍 Baseline vs. Anomaly
Standard Output: Clean systemd timers for logrotate.timer, fstrim.timer, apt-daily.timer. Service unit files owned strictly by root:root with -rw-r--r-- permissions.

🔴 Anomalies to Spot:

Writable unit files (.service) where a non-root user can edit the ExecStart= parameter.

Custom timers executing unprivileged scripts with root rights.
