# Synced Box – Writeup (Easy)

**Machine:** Synced  
**Difficulty:** Easy  
**Tech Used:** Nmap, Rsync, Linux Command Line  
**Objective:** Demonstrate network enumeration, identifying an exposed Rsync service, accessing an anonymously available module, and retrieving a flag.

**By:** Ce'Bria Haynes |  **Published:** 9/21/2026

---

## Overview

Synced is an easy-level HackTheBox machine focused on basic network enumeration and identifying an improperly configured Rsync service. The machine demonstrates how exposed services can provide access to sensitive files when anonymous access is enabled.

I began by performing a full TCP port scan using Nmap to identify the services running on the target. The scan revealed that only port `873` was open, which is commonly associated with the Rsync service.

After identifying Rsync, I used the `rsync` command to query the daemon directly and determine what modules were available. The output showed that anonymous access was enabled and that a module named `public` was accessible.

I then enumerated the contents of the `public` module and discovered a file named `flag.txt`. Since the file was accessible without authentication, I downloaded it directly using Rsync.

After downloading the file to my local machine, I used `cat` to read its contents and retrieve the flag.

---

## Enumeration

### Nmap Scan:

I started with a full TCP port scan using Nmap with service detection and default scripts:

```bash
nmap -sC -sV -p- 10.129.147.66
```

<img width="605" height="217" alt="nmap" src="https://github.com/user-attachments/assets/787634ee-d972-44c2-824a-6e7df159c0aa" />

The scan revealed that only one port was open:

```text
873/tcp open  rsync
```

Port 873 is commonly used by Rsync, a utility that allows files and directories to be synchronized between systems.

Since Rsync was the only exposed service, I focused my enumeration on the Rsync daemon.

## Rsync Enumeration

### Checking Available Modules:

I used the rsync command to connect to the daemon and list the available modules:

```bash
rsync --list-only rsync://10.129.147.66/
```

<img width="568" height="51" alt="rsync results" src="https://github.com/user-attachments/assets/08ae8d4f-4ecb-43f8-90a9-c9192af744a4" />


The output revealed an accessible module named:

```text
public
```

This was interesting because it indicated that the Rsync service was allowing access to files without requiring authentication.

Anonymous access to a file synchronization service can potentially expose sensitive files if the service has not been properly configured.

### Enumerating the Public Module

After discovering the public module, I checked its contents to determine what files were available:

```bash
rsync --list-only rsync://10.129.147.66/public/
```

<img width="486" height="51" alt="public module" src="https://github.com/user-attachments/assets/47887d4c-a5e6-460c-8657-2168ede74ef8" />

The output showed that a file named flag.txt was present.

```text
flag.txt
```

Since the file was accessible through the publicly exposed module, I did not need to exploit any additional vulnerability.

### Downloading the Flag

I downloaded the flag.txt file using Rsync:

```bash
rsync -av rsync://10.129.147.66/public/flag.txt ./
```
<img width="541" height="98" alt="flag download" src="https://github.com/user-attachments/assets/e6c0dbfd-a11c-4d83-8220-cac618da0a46" />

The -a option enables archive mode, while -v provides verbose output so I could see the transfer taking place.

After the command completed, the file was downloaded to my current working directory.

I verified that the file was present:

```bash
ls
```

The output showed:

```text
flag.txt
```

I then used cat to read the contents of the file:

```bash
cat flag.txt
```

The flag was displayed in the terminal.

<img width="1134" height="99" alt="flag" src="https://github.com/user-attachments/assets/c44610f1-6012-44d3-a6bb-992abdfe1191" />

## Conclusion

Synced was a straightforward machine that demonstrated the importance of properly configuring network services and checking for anonymous access.

The initial Nmap scan identified port 873 as the only open port on the target. Since port 873 is associated with Rsync, I used the Rsync client to enumerate the available modules on the server.

The enumeration showed that anonymous access was enabled and that a module named public was available. I then enumerated the contents of this module and discovered flag.txt.

Because the file was accessible without authentication, I downloaded it directly using the Rsync client:

```bash
rsync -av rsync://10.129.147.66/public/flag.txt ./
```

Finally, I used the cat command to read the downloaded file and retrieve the flag.

The main techniques demonstrated by this machine were:

- Full TCP port enumeration with Nmap

- Service and version enumeration

- Identifying Rsync on port 873

- Enumerating Rsync modules

- Identifying anonymous access

- Enumerating files within an exposed module

- Downloading files using Rsync

- Retrieving a flag from an exposed file

Overall, Synced demonstrates how a simple service misconfiguration can expose files to unauthorized users.
An Rsync daemon should be properly configured with appropriate access controls and authentication where sensitive data is involved.
In this case, anonymous access to the public module allowed the flag to be retrieved without requiring any further exploitation.

