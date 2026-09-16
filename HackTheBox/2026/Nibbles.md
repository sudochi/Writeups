# Nibbles – Writeup (Easy)

**Machine:** Nibbles  
**Difficulty:** Easy  
**Tech Used:** Nmap, FFUF, Nibbleblog, Metasploit, Reverse Shell, Linux privilege escalation, Sudo  
**Objective:** Demonstrate web enumeration, exploitation of a vulnerable CMS, obtaining a user shell, and privilege escalation to root.

---

## Overview

Nibbles is an easy-level HackTheBox machine focused on web enumeration, CMS exploitation, and Linux privilege escalation. The initial Nmap scan revealed two open ports: SSH on port 22 and HTTP on port 80. After adding the target IP to `/etc/hosts`, I navigated to the website running on port 80.

The webpage was very basic and only displayed "Hello world." After viewing the page source, I discovered a comment pointing to a `/nibbleblog/` directory. Navigating to this directory revealed a blog running the Nibbleblog CMS.

Further enumeration revealed the Nibbleblog administrative interface at `/admin.php`. After researching the default Nibbleblog credentials, I was able to log into the admin dashboard using `admin:nibbles`.

I then identified the installed Nibbleblog version as 4.0.3. This version contains an authenticated remote file upload vulnerability, identified as CVE-2015-6967. I used the corresponding Metasploit module to exploit the vulnerability and obtain a reverse shell as the `nibbler` user.

After retrieving the user flag, I checked the sudo permissions available to the `nibbler` user. I discovered that `nibbler` could execute `monitor.sh` as root. Since I had permission to modify the script, I was able to change it so that it read the root flag when executed with elevated privileges. This allowed me to retrieve the root flag and complete the machine.

---

## Enumeration

### Nmap Scan:

I started with a full TCP port scan using Nmap with service detection and default scripts:

```bash
nmap -sC -sV -p- 10.129.57.17 -oA nibbles_scan
```

The scan showed that two ports were open:

```bash
22/tcp open  SSH
80/tcp open  HTTP
```

Since port 80 was running an HTTP service, I added the target IP to my /etc/hosts file and navigated to the target in a web browser.

The webpage itself was very basic and only displayed:

```bash
Hello world
```

Since the webpage did not reveal anything useful, I viewed the page source.

While inspecting the source code, I found the following comment:

```bash
<!-- /nibbleblog/ directory. Nothing interesting here! -->
```

This revealed a potentially interesting directory, so I navigated to /nibbleblog.

The directory contained a blog site with no posts. The site was running the Nibbleblog CMS.

### Directory Enumeration:

While viewing the page source, I also noticed a JavaScript file located at:

```bash
/nibbleblog/admin/js/jquery/jquery.js
```

This suggested that an administrative directory existed under /nibbleblog/admin.

I navigated to /nibbleblog/admin and found several directories:

```bash
ajax
boot
controllers
js
kernel
templates
views
```

I then performed additional directory enumeration using FFUF:

```bash
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://nibbles.htb/nibbleblog/FUZZ -mc 200 -fs 42 -c -v
```

FFUF discovered several interesting files:

```bash
/index.php
/README
/admin.php
```

The /admin.php file was particularly interesting because it provided access to the Nibbleblog administrative login page.

## Initial Foothold
### Nibbleblog Admin Login

I navigated to /nibbleblog/admin.php and was presented with the Nibbleblog login page.

After researching the default credentials for Nibbleblog, I found that the default username and password were:

```bash
Username: admin
Password: nibbles
```

I successfully logged in using these credentials and gained access to the Nibbleblog administrative dashboard.

### Identifying the Nibbleblog Version

After gaining access to the administrative dashboard, I checked the update page at:

```bash
/nibbleblog/update.php
```

The installed version was:

```bash
Nibbleblog 4.0.3
```

This version of Nibbleblog contains an authenticated remote file upload vulnerability identified as CVE-2015-6967.

The vulnerability allows an authenticated user to upload a malicious file and achieve arbitrary PHP code execution on the target.

### Exploiting CVE-2015-6967

I used the following Metasploit module to exploit the vulnerability:

```bash
exploit/multi/http/nibbleblog_file_upload
```

The module successfully exploited the authenticated file upload vulnerability and provided a reverse shell on the target.

Once the connection was received, I upgraded the shell to make it more interactive.

I then confirmed the current user using:

```bash
whoami
```

The result showed:

```bash
nibbler
```

This confirmed that I had obtained command execution on the target as the `nibbler` user.

### User Flag

With access to the target as nibbler, I navigated to the user's home directory:

```bash
cd /home/nibbler
```

The user.txt file was present in the home directory. I retrieved the flag using:

```bash
cat user.txt
```

This successfully provided the user flag.

## Privilege Escalation

After obtaining the user flag, I checked the sudo permissions available to the nibbler user:

```bash
sudo -l
```

The output showed that the nibbler user could execute monitor.sh as root.

I navigated to the directory containing the script:

```bash
cd /home/nibbler/personal/stuff
```

The directory contained the monitor.sh script.

Since I had permission to modify the script, this presented an opportunity to execute commands with root privileges.

I modified monitor.sh to contain:

```bash
echo "cat /root/root.txt" > monitor.sh
```

I then executed the script using the sudo permissions identified earlier.

Because monitor.sh was executed as root, the command inside the script also ran with root privileges. This caused the contents of /root/root.txt to be displayed.

The root flag was successfully retrieved.

## Conclusion

Nibbles demonstrates how multiple simple security weaknesses can be chained together to obtain complete system compromise.

The initial Nmap scan identified SSH and HTTP as the only exposed services. Since port 80 was running an HTTP service, I investigated the website and found a comment in the page source pointing toward the /nibbleblog/ directory.

The directory contained a Nibbleblog CMS installation. Further enumeration revealed the administrative interface at /admin.php. Researching the default Nibbleblog credentials allowed me to successfully authenticate to the administrative dashboard using the default admin:nibbles credentials.

After gaining access to the CMS, I identified the installed version as Nibbleblog 4.0.3. This version is vulnerable to an authenticated remote file upload vulnerability identified as CVE-2015-6967. Using the appropriate Metasploit module, I exploited the vulnerability and obtained a reverse shell as the nibbler user.

From the initial foothold, I navigated to /home/nibbler and retrieved the user flag. I then checked the user's sudo permissions using sudo -l and discovered that nibbler could execute monitor.sh as root.

Because I had permission to modify the script, I changed its contents so that it would read /root/root.txt. Executing the script through sudo caused the command to run with root privileges, allowing me to retrieve the root flag.

The main techniques demonstrated by this machine were:

- Full TCP port enumeration with Nmap
- Web enumeration
- Inspecting webpage source code
- Directory and file enumeration with FFUF
- Identifying the Nibbleblog CMS
- Exploiting CVE-2015-6967
- Using Metasploit to obtain a reverse shell
- Linux filesystem enumeration
- Sudo privilege escalation
- Exploiting a writable script executed by root
- Obtaining the root flag

Overall, Nibbles is a good demonstration of how default credentials, a vulnerable CMS, and an insecure sudo configuration can be chained together to progress from an exposed web application to complete root-level access.
