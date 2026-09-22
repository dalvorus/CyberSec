# Deep Dive: Environment, Users & Sensitive File Permissions

After performing initial quick checks, perform an in-depth audit of local user accounts, user history files, active sessions, and access permissions on critical system configuration files.

---

## 1. User Accounts & Password Policies

Analyze system users, interactive shell access, and UID/GID configurations.

```bash
# List accounts with interactive shell access (bash, sh, zsh)
grep -vE ':(/bin/false|/sbin/nologin|/bin/sync)$' /etc/passwd

# Check for accounts with UID 0 (root privileges)
awk -F: '($3 == 0) {print $1}' /etc/passwd

# Inspect last logged-in users and current active sessions
last
w
🔍 Baseline vs. Anomaly
Standard Output: Only root has UID 0. Interactive shells (/bin/bash) are assigned only to real users (e.g., UID >= 1000). System daemons (www-data, nobody, daemon) use /usr/sbin/nologin or /bin/false.

🔴 Anomalies to Spot:

Multiple users with UID 0 (backdoor accounts).

Service accounts (e.g., www-data or mysql) assigned a valid interactive shell like /bin/bash.

Unknown users in /etc/passwd created recently without standard system group memberships.

2. Sensitive File Permissions (/etc/passwd & /etc/shadow)
Inspect key authentication files for broken permission masks allowing unauthorized modification or reading.

Bash
# Check read/write permissions on critical password files
ls -la /etc/passwd /etc/shadow /etc/master.passwd 2>/dev/null

# Test if /etc/passwd or /etc/shadow are writable by current user
[ -w /etc/passwd ] && echo "[!] /etc/passwd IS WRITABLE"
[ -w /etc/shadow ] && echo "[!] /etc/shadow IS WRITABLE"
🔍 Baseline vs. Anomaly
Standard Output:

/etc/passwd: -rw-r--r-- 1 root root (Readable by all, writable only by root).

/etc/shadow: -rw-r----- 1 root shadow (Readable/writable only by root and shadow group).

🔴 Anomalies to Spot:

Writable /etc/passwd: Allows appending a custom root user with a known password hash directly (e.g., hacker:$1$saltsalt$...:0:0:root:/root:/bin/bash).

Readable /etc/shadow: Allows cracking root or service password hashes offline using john or hashcat.

3. Shell History & Sensitive Credentials
Search user home directories and system logs for plain-text credentials, tokens, or history files.

Bash
# Search for readable shell history files in /home and /root
ls -la /home/*/.bash_history /home/*/.python_history /root/.bash_history 2>/dev/null

# Grep for common password keywords in history files
grep -iE 'password|pass|pwd|ssh|key' /home/*/.bash_history 2>/dev/null

# Search for SSH keys (private/public)
find /home /root /var /opt -name "id_rsa" -o -name "id_ed25519" -o -name "known_hosts" 2>/dev/null
🔍 Baseline vs. Anomaly
Standard Output: Shell history files owned by their respective users, mode 600 (-rw-------), without hardcoded plain-text secrets or passwords passed as inline arguments.

🔴 Anomalies to Spot:

Readable private SSH keys (id_rsa) stored in world-readable locations or web roots (/var/www/html).

Plain-text passwords exposed in history logs (e.g., mysql -u root -pSuperSecret123, sshpass -p 'pass123' ssh user@host).

Unprotected API keys, bearer tokens, or cloud credentials in ~/.aws/credentials or ~/.config/.

4. Environment Variables & PATH Abuse
Examine environment variables for unsafe path declarations or exported credentials.

Bash
# Print current environment settings
env

# Check default PATH for non-privileged accounts and sudo context
echo $PATH
sudo env | grep PATH
🔍 Baseline vs. Anomaly
Standard Output: Clean system PATH pointing to standard binaries (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin).

🔴 Anomalies to Spot:

Relative paths or . (current directory) in $PATH.

Environment variables containing active database credentials (DB_PASSWORD), secret keys (SECRET_KEY), or sudo tokens.

env_keep directives in sudoers allowing dangerous variables like LD_PRELOAD or LD_LIBRARY_PATH to persist during elevated execution.
EOFcat << 'EOF' > 01-enumeration/02-env-and-users.md
# Deep Dive: Environment, Users & Sensitive File Permissions

After performing initial quick checks, perform an in-depth audit of local user accounts, user history files, active sessions, and access permissions on critical system configuration files.

---

## 1. User Accounts & Password Policies

Analyze system users, interactive shell access, and UID/GID configurations.

```bash
# List accounts with interactive shell access (bash, sh, zsh)
grep -vE ':(/bin/false|/sbin/nologin|/bin/sync)$' /etc/passwd

# Check for accounts with UID 0 (root privileges)
awk -F: '($3 == 0) {print $1}' /etc/passwd

# Inspect last logged-in users and current active sessions
last
w
🔍 Baseline vs. Anomaly
Standard Output: Only root has UID 0. Interactive shells (/bin/bash) are assigned only to real users (e.g., UID >= 1000). System daemons (www-data, nobody, daemon) use /usr/sbin/nologin or /bin/false.

🔴 Anomalies to Spot:

Multiple users with UID 0 (backdoor accounts).

Service accounts (e.g., www-data or mysql) assigned a valid interactive shell like /bin/bash.

Unknown users in /etc/passwd created recently without standard system group memberships.

2. Sensitive File Permissions (/etc/passwd & /etc/shadow)
Inspect key authentication files for broken permission masks allowing unauthorized modification or reading.

Bash
# Check read/write permissions on critical password files
ls -la /etc/passwd /etc/shadow /etc/master.passwd 2>/dev/null

# Test if /etc/passwd or /etc/shadow are writable by current user
[ -w /etc/passwd ] && echo "[!] /etc/passwd IS WRITABLE"
[ -w /etc/shadow ] && echo "[!] /etc/shadow IS WRITABLE"
🔍 Baseline vs. Anomaly
Standard Output:

/etc/passwd: -rw-r--r-- 1 root root (Readable by all, writable only by root).

/etc/shadow: -rw-r----- 1 root shadow (Readable/writable only by root and shadow group).

🔴 Anomalies to Spot:

Writable /etc/passwd: Allows appending a custom root user with a known password hash directly (e.g., hacker:$1$saltsalt$...:0:0:root:/root:/bin/bash).

Readable /etc/shadow: Allows cracking root or service password hashes offline using john or hashcat.

3. Shell History & Sensitive Credentials
Search user home directories and system logs for plain-text credentials, tokens, or history files.

Bash
# Search for readable shell history files in /home and /root
ls -la /home/*/.bash_history /home/*/.python_history /root/.bash_history 2>/dev/null

# Grep for common password keywords in history files
grep -iE 'password|pass|pwd|ssh|key' /home/*/.bash_history 2>/dev/null

# Search for SSH keys (private/public)
find /home /root /var /opt -name "id_rsa" -o -name "id_ed25519" -o -name "known_hosts" 2>/dev/null
🔍 Baseline vs. Anomaly
Standard Output: Shell history files owned by their respective users, mode 600 (-rw-------), without hardcoded plain-text secrets or passwords passed as inline arguments.

🔴 Anomalies to Spot:

Readable private SSH keys (id_rsa) stored in world-readable locations or web roots (/var/www/html).

Plain-text passwords exposed in history logs (e.g., mysql -u root -pSuperSecret123, sshpass -p 'pass123' ssh user@host).

Unprotected API keys, bearer tokens, or cloud credentials in ~/.aws/credentials or ~/.config/.

4. Environment Variables & PATH Abuse
Examine environment variables for unsafe path declarations or exported credentials.

Bash
# Print current environment settings
env

# Check default PATH for non-privileged accounts and sudo context
echo $PATH
sudo env | grep PATH
🔍 Baseline vs. Anomaly
Standard Output: Clean system PATH pointing to standard binaries (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin).

🔴 Anomalies to Spot:

Relative paths or . (current directory) in $PATH.

Environment variables containing active database credentials (DB_PASSWORD), secret keys (SECRET_KEY), or sudo tokens.

env_keep directives in sudoers allowing dangerous variables like LD_PRELOAD or LD_LIBRARY_PATH to persist during elevated execution.
