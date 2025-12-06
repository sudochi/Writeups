# Neighbor – Writeup (Very Easy)

**Machine:** Neighbor
**Difficulty:** Very Easy  
**Tech Used:** Browser Inspection, Source Code Review, URL Parameter Manipulation (IDOR)  
**Objective:** Exploit insecure direct object references to escalate from a guest account to the admin account.

---

## Overview

This very simple TryHackMe challenge focuses on weak authentication controls and demonstrates how improper authorization checks can lead to privilege escalation.

The machine provides a basic **guest:guest** login and includes a hidden developer comment that exposes how the user system works.  
By modifying a single URL parameter, we can access the admin account and retrieve the flag.

---

## Enumeration

### Source Code Inspection

After opening the main page, viewing the source with:
```yaml
ctrl + u
```
reveals the following comment:

![]()


Key observations:

- Valid credentials: **guest : guest**
- An **admin user** exists  
- Developers trust client-side logic (a red flag)

---

## Initial Access

Log in using the provided credentials:

- Username: guest
- Password: guest

![]()

This logs you into a restricted guest dashboard.

During navigation, the URL displays a user identifier:

![]()

```yaml
?user=guest
```


This is a strong indicator of an **IDOR vulnerability**.

---

## Exploitation – IDOR

By changing the URL parameter directly:

![]()

```yaml
?user=admin
```



and refreshing the page, the application loads the **admin page** without requiring authentication.

The admin flag is displayed immediately, confirming:

- No server-side authorization checks  
- Client-controlled identifiers are fully trusted  
- Simple URL manipulation leads to privilege escalation  

---

## Conclusion

The machine highlights the importance of:

- Proper server-side authorization  
- Never trusting client-controlled data  
- Removing sensitive debugging comments before deployment  
- Understanding how simple IDOR vulnerabilities can lead to full compromise  
