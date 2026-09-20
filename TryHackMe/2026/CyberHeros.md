# CyberHeros – Writeup (Easy)

**Machine:** CyberHeros  
**Difficulty:** Easy  
**Tech Used:** Web Enumeration, Page Source Analysis, JavaScript, Credential Discovery  
**Objective:** Demonstrate web enumeration, source-code analysis, discovery of exposed credentials, and authentication to retrieve the flag.

**By:** Ce'Bria Haynes |  **Published:** 9/20/2026

---

## Overview

CyberHeros is an easy-level room focused on basic web enumeration and analyzing client-side code for sensitive information. The initial website presented a platform for hackers, bug hunters, and developers who find bugs in websites legally.

I began by navigating to the target website at `http://10.65.132.6`. After exploring the site, I discovered a login page. Rather than attempting to guess the login credentials, I inspected the page source to see if there was any useful information exposed within the site's code.

While reviewing the source code, I discovered a JavaScript snippet containing what appeared to be a username and password. The username was provided directly in the script, while the password had been reversed.

After reversing the password, I was able to use the discovered credentials to successfully log in. The next page displayed the flag, completing the room.

---

## Enumeration

### Website Enumeration:

I started by navigating to the target IP address:

```text
http://10.65.132.6
```

The website was called CyberHeros and appeared to be designed for hackers, bug hunters, and developers. The website described itself as a place where users could find bugs in websites legally.

<img width="1920" height="958" alt="webpage" src="https://github.com/user-attachments/assets/dd66629c-345b-401a-a0d1-b2d3ee87e1c9" />

I explored the website and discovered a login page.

<img width="901" height="434" alt="login page" src="https://github.com/user-attachments/assets/3ca68c1f-855c-4dab-b8d6-c5cc65ba126d" />

The login page required a username and password. Instead of attempting to brute-force the login, I decided to inspect the page source for information that could reveal how the authentication process worked.

### Source Code Analysis:

I viewed the source code of the login page and searched through the JavaScript for anything related to the login functionality.

I found the following JavaScript:

```text
if (a.value=="h3ck3rBoi" & b.value==RevereString("54321@terceSrepuS"))
```
<img width="644" height="263" alt="source code script" src="https://github.com/user-attachments/assets/c79a0a32-c8ba-4914-9acc-5096c788d25a" />

This immediately stood out because the username was directly visible in the script:

```text
h3ck3rBoi
```

The password was also present, but it had been reversed:

```text
54321@terceSrepuS
```

I reversed the string to obtain the actual password:

```text
SuperSecret@12345
```

This gave me the following credentials:

```text
Username: h3ck3rBoi
Password: SuperSecret@12345
```

The credentials being stored in client-side JavaScript was the main security issue.
Since the JavaScript is sent to the user's browser, it can be viewed and analyzed.

## Initial Access

After discovering the username and reversing the password, I returned to the CyberHeros login page and entered the credentials.
I was able to login successfully, confirming that the information found in the JavaScript was valid.
After logging in, I was redirected to another page.

## Flag:

The authenticated page displayed the flag.

<img width="1773" height="708" alt="flag" src="https://github.com/user-attachments/assets/48a4aa44-f669-4aad-a30a-69e6f7d81c81" />

Since the flag was directly accessible after authentication, no additional exploitation or privilege escalation was required.

## Conclusion

CyberHeros demonstrates how sensitive information exposed in client-side JavaScript can lead to unauthorized access to a web application.

The initial enumeration revealed a website running at http://10.65.132.6, which contained a login page. Instead of attempting to guess the credentials, I inspected the page source and discovered JavaScript containing hardcoded authentication information.

The username h3ck3rBoi was directly visible in the JavaScript. The password was stored in reverse as 54321@terceSrepuS. After reversing the string, I obtained SuperSecret@12345.

Using these credentials, I successfully authenticated to the application and was redirected to a page containing the flag.

The main techniques demonstrated by this machine were:

- Web application enumeration

- Viewing page source

- Analyzing JavaScript

- Identifying hardcoded credentials

- Reversing an obfuscated password

- Using discovered credentials to authenticate

Overall, CyberHeros is a simple example of why sensitive credentials should never be stored in client-side JavaScript. Anything delivered to the browser can potentially be viewed by the user.
Authentication should instead be handled securely on the server side, and passwords should never be exposed within client-side code.
