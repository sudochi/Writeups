# Nibbles – Writeup (Easy)

**Machine:** Nibbles  
**Difficulty:** Easy  
**Tech Used:** Nmap, FFUF, Nibbleblog, Metasploit, Reverse Shell, Linux privilege escalation, Sudo  
**Objective:** Demonstrate web enumeration, exploitation of a vulnerable CMS, obtaining a user shell, and privilege escalation to root.


**By:** Ce'Bria Haynes |
**Published:** 9/8/2026

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
<img width="608" height="332" alt="nmap" src="https://github.com/user-attachments/assets/73a612bf-dfe5-481b-a80a-dbd42bc36722" />

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
<img width="740" height="353" alt="webpage" src="https://github.com/user-attachments/assets/34caf232-cc9c-4b8f-9d2f-1b2caa162eed" />

Since the webpage did not reveal anything useful, I viewed the page source.

While inspecting the source code, I found the following comment:

```bash
<!-- /nibbleblog/ directory. Nothing interesting here! -->
```
<img width="580" height="317" alt="view source" src="https://github.com/user-attachments/assets/73ebd8ea-3bdd-4916-adf7-276c6eacd22a" />

This revealed a potentially interesting directory, so I navigated to /nibbleblog.

The directory contained a blog site with no posts. The site was running the Nibbleblog CMS.

<img width="900" height="401" alt="blog" src="https://github.com/user-attachments/assets/a2144030-c544-45f4-81f5-8e5738018c2a" />

### Directory Enumeration:

While viewing the page source, I also noticed a JavaScript file located at:

```bash
/nibbleblog/admin/js/jquery/jquery.js
```

<img width="416" height="13" alt="admin directory src" src="https://github.com/user-attachments/assets/07b62e75-7277-45c4-b4a6-7a763ec244d3" />

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

<img width="482" height="317" alt="nibbleblog-admin-directories" src="https://github.com/user-attachments/assets/9bf9dd7d-5a07-40aa-9c35-f978d2a28184" />

I then performed additional directory enumeration using FFUF:

```bash
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://nibbles.htb/nibbleblog/FUZZ -mc 200 -fs 42 -c -v
```
<img width="605" height="283" alt="fuff" src="https://github.com/user-attachments/assets/13b1981d-d8d2-4f4e-ad19-014fb8674254" />

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

<img width="901" height="409" alt="admin login" src="https://github.com/user-attachments/assets/3b957446-6886-465e-aef3-25eccf327fd1" />

After researching the default credentials for Nibbleblog, I found that the default username and password were:

```bash
Username: admin
Password: nibbles
```

I successfully logged in using these credentials and gained access to the Nibbleblog administrative dashboard.

<img width="898" height="347" alt="admin dashboard" src="https://github.com/user-attachments/assets/39e3b462-4c46-4b22-8223-22c87232e132" />

### Identifying the Nibbleblog Version

After gaining access to the administrative dashboard, I checked the update page at:

```bash
/nibbleblog/update.php
```

<img width="629" height="247" alt="nibbeblog version" src="https://github.com/user-attachments/assets/0ab57e96-5654-40ca-9cd6-afc1e22142f7" />

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

<img width="614" height="181" alt="metasploit shell" src="https://github.com/user-attachments/assets/57eed879-6627-497d-a26f-bc4a4a081d22" />

Once the connection was received, I upgraded the shell to make it more interactive.

I then confirmed the current user using:

```bash
whoami
```

The result showed:

```bash
nibbler
```
<img width="608" height="82" alt="whoami nibbler" src="https://github.com/user-attachments/assets/a2c2102e-aa51-423a-a0e4-d1a53fc3d4f5" />

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

<img width="722" height="102" alt="user flag" src="https://github.com/user-attachments/assets/95c8bcf8-69fd-4d0a-a257-d602d44154ca" />

This successfully provided the user flag.

## Privilege Escalation

After obtaining the user flag, I checked the sudo permissions available to the nibbler user:

```bash
sudo -l
```

<img width="614" height="147" alt="sudo l" src="https://github.com/user-attachments/assets/d99d63d3-5ae6-41a5-97f9-20518ebcbec7" />

The output showed that the nibbler user could execute monitor.sh as root.

I navigated to the directory containing the script:

```bash
cd /home/nibbler/personal/stuff
```

Since I had permission to modify the script, this presented an opportunity to execute commands with root privileges.

I modified monitor.sh to contain:

```bash
echo "cat /root/root.txt" > monitor.sh
```

I then executed the script using the sudo permissions identified earlier.

Because monitor.sh was executed as root, the command inside the script also ran with root privileges. This caused the contents of /root/root.txt to be displayed.

<img width="1224" height="197" alt="root flag" src="https://github.com/user-attachments/assets/89a1007f-00ef-47b3-ae9c-4ea99aab0733" />

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
