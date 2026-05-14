# TryHackMe — CTF Walkthrough

**Difficulty:** Easy

**Category:** Enumeration / SMB / Privilege Escalation

**Date:** 2026-05

## Summary

Gained initial access by enumerating an exposed SMB share that leaked
credentials, then escalated privileges to root via a sudo
misconfiguration.

## 1. Reconnaissance

Identified the target host on the network and scanned it for open
services:

```
sudo nmap <target-ip> -sS -sV -T4
```

**Findings:** [list the open ports/services nmap returned — e.g.
22/SSH, 80/HTTP, 139 + 445/SMB]

Checked the web server in a browser to look for an obvious entry point:

```
firefox http://<target-ip>/
```

**Result:** [what you found here — or "nothing immediately exploitable,
so I pivoted to SMB"]

## 2. SMB Enumeration

With SMB exposed, I enumerated users and shares using enum4linux:

```
enum4linux -U <target-ip>      # enumerate users
enum4linux -S <target-ip>      # enumerate shares
```

This revealed a readable share. I connected to it and pulled down the
available files:

```
smbclient //<target-ip>/share$
smb> ls
smb> get deets.txt
smb> get todolist.txt
smb> exit
```

## 3. Initial Access

Reviewing the downloaded files locally:

```
cat todolist.txt
cat deets.txt
```

**Key finding:** [explain what these files contained — e.g. deets.txt
held a plaintext password — and which user it belonged to]

I cross-referenced this against the local users identified earlier:

```
enum4linux <target-ip> | grep "Local User"
```

I first tried logging in as `nobody`, which had no valid shell, then
authenticated successfully over SSH as the intended user:

```
ssh nobody@<target-ip>     # no valid shell
ssh togie@<target-ip>      # success
```

## 4. Privilege Escalation

After landing a shell, I checked the current user's sudo permissions:

```
sudo -l
```

**Finding:** [what sudo -l returned — this is the critical step, e.g.
"the user was permitted to run su as root with no password"]

I used that misconfiguration to escalate to root:

```
sudo su
```

## 5. Proof

```
cd /root
ls
cat root.txt
```

Root access confirmed. (Flag value omitted intentionally.)

## Lessons Learned

- Exposed, readable SMB shares are a frequent initial foothold —
  always enumerate them early.
- Storing credentials in plaintext files (like `deets.txt`) is a
  common real-world misconfiguration and shows why proper secrets
  management matters.
- Overly permissive `sudo` rules turn a low-privilege shell into full
  root access — least privilege should always be enforced.
- [add one genuine takeaway in your own words]

## Tools Used

`nmap` · `enum4linux` · `smbclient` · `ssh`

## Disclaimer

This walkthrough was completed in a legal lab/CTF environment for educational purposes only.
