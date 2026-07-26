# Project 2: Linux Fundamentals + Self-Built Mini CTF — Write-Up

**Author:** Suraj Pratap Singh
**Date:** July 2026
**Environment:** Same Kali Linux / Metasploitable2 lab from Project 1
**Tools used:** chmod, usermod, newgrp, ps, kill, crontab, /proc filesystem

## Objective

Practice core Linux fundamentals — file permissions, users & groups, processes, and cron jobs — then apply that knowledge by building and solving a self-made mini CTF with one flag per concept.

## Part 1: Linux Fundamentals

### File Permissions
- Explored the `ls -l` permission string (`-rwxr--r--` format): file type, owner/group/other read-write-execute bits
- Used `chmod 600` (numeric mode) and `chmod +x` (symbolic mode) to restrict and grant access
- Learned that execute permission is separate from read/write — a text file isn't runnable until explicitly marked executable

### Users & Groups
- Inspected identity with `whoami` and `id` — UID, primary GID, and secondary group memberships
- Confirmed that `sudo` access is just group membership (being in the `sudo` group), not a special flag
- Created a new user (`testuser`) with `adduser`, confirmed it had no sudo access by default
- Granted sudo access with `usermod -aG sudo testuser` and confirmed the change took effect

### Processes
- Used `ps aux` to view all running processes and `ps aux | grep <name>` to filter
- Used `pstree` to visualize parent-child process relationships
- Started a background process with `sleep 300 &`, found its PID, and terminated it with `kill <PID>`

### Cron Jobs
- Created a scheduled task with `crontab -e` running every minute
- Verified execution by checking the output file it generated over time
- Removed the job cleanly with `crontab -r`

## Part 2: Self-Built Mini CTF

Built and solved four flags, each testing one fundamental:

1. **Permissions flag** — a file created with `chmod 000` (no access, even for the owner). Solved by restoring access with `chmod 600`.
2. **Users & groups flag** — a file owned by `root:secretgroup` with group-read permission. Solved by adding myself to the group (`usermod -aG`) and refreshing group membership in-session with `newgrp`.
3. **Process flag** — a flag hidden in a background process's environment variable. Solved by finding the process PID via `ps aux` and reading `/proc/<PID>/environ`.
4. **Cron flag** — a flag embedded in a scheduled cron command. Solved instantly via `crontab -l` (no need to wait for execution), and confirmed again via the actual scheduled output file.

## Key Takeaways

- Permissions control access independently of ownership — even a file's owner can be locked out.
- `sudo` privilege is fundamentally just group membership; there's no hidden magic to it.
- Environment variables passed to a process are visible to anyone who can read `/proc/<PID>/environ` — a real-world credential leak risk if secrets are set this way instead of using proper secrets management.
- Cron entries are usually readable directly via `crontab -l` — a "hidden" cron-based flag or backdoor isn't actually hidden from anyone who checks.
- Cron is a classic persistence mechanism attackers use to survive reboots on a compromised machine.

## Next Steps

- Project 3: Web fundamentals — inspecting Metasploitable2's web services with browser dev tools / Burp Suite
- Project 4: Cryptography practice set (Python + CyberChef)
