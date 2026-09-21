# Mongod – Writeup (Easy)

**Machine:** Mongod  
**Difficulty:** Easy  
**Tech Used:** Nmap, MongoDB, Mongosh, Database Enumeration  
**Objective:** Demonstrate network enumeration, MongoDB enumeration, database discovery, collection enumeration, and retrieval of an exposed flag.

**By:** Ce'Bria Haynes |  **Published:** 9/21/2026

---

## Overview

Mongod is an easy-level HackTheBox machine focused on network enumeration and identifying an exposed MongoDB service. The machine demonstrates how an improperly secured database service can allow an unauthenticated user to access databases and sensitive information.

I began by performing a full TCP port scan using Nmap. The scan identified two open ports: SSH on port 22 and MongoDB on port 27017. Since MongoDB was directly exposed, I focused my enumeration on the database service.

After installing `mongosh`, I connected directly to the MongoDB service using the target IP address and port 27017. The connection was successful, and I was able to interact with the database without needing additional authentication.

I then enumerated the available databases and discovered five databases: `admin`, `config`, `local`, `sensitive_information`, and `users`.

I started by examining the `users` database. It contained a collection called `ecommerceWebapp`, which contained information about users, including usernames, email addresses, account creation dates, and hashed passwords.

I then moved on to the `sensitive_information` database. This database contained a collection called `flag`. After viewing the contents of this collection, I was able to retrieve the flag and complete the machine.

---

## Enumeration

### Nmap Scan:

I started with a full TCP port scan using Nmap with service detection and default scripts:

```bash
nmap -sC -sV -p- 10.129.228.30 -oA mongod_scan
```

<img width="602" height="348" alt="nmap" src="https://github.com/user-attachments/assets/59a465b6-dcef-410e-8c48-220beda0480e" />

The scan identified two open ports:

```text
22/tcp    open    ssh
27017/tcp open    mongodb
```

Port 22 was running SSH, while port 27017 was running MongoDB.

Since MongoDB was directly exposed to the network, I decided to investigate the database service.

## MongoDB Enumeration

### Connecting to MongoDB:

I first installed mongosh, the MongoDB shell, so I could interact with the database.

Once installed, I connected to the MongoDB service using:

```bash
mongosh "mongodb://10.129.228.30:27017"
```

<img width="609" height="367" alt="mongodb connect" src="https://github.com/user-attachments/assets/c2f22193-eb93-4672-9274-bdefb4b1861e" />

The connection was successful, and I was able to access the MongoDB instance as the test user.

This indicated that the database could be accessed without requiring additional authentication.

### Database Enumeration

After connecting to MongoDB, I listed the available databases.

<img width="497" height="119" alt="show dbs" src="https://github.com/user-attachments/assets/e947a19e-8e8b-4b57-8406-22d6cd88f371" />

The database server contained five databases:

```text
admin
config
local
sensitive_information
users
```

The admin, sensitive_information, and users databases stood out because their names suggested they might contain useful information.

I decided to start by examining the users database.

### Users Database

I switched to the users database and enumerated its collections.

The database contained a collection called:

```text
ecommerceWebapp
```

I viewed the contents of the collection using:

```bash
db.ecommerceWebapp.find().pretty()
```

<img width="586" height="363" alt="user info" src="https://github.com/user-attachments/assets/dbc08d09-5bb8-4158-9606-10a3b4cd9212" />

The collection contained information about the application's users.

The records included:

```text
Usernames
Email addresses
Account creation dates
Hashed passwords
```

This demonstrated that the exposed MongoDB instance contained sensitive user information.

Although the passwords were hashed, the fact that usernames, email addresses, account creation information, and password hashes were accessible without proper authorization represented a significant security issue.

### Sensitive Information Database

After examining the users database, I moved on to the sensitive_information database.

<img width="385" height="66" alt="flag collection" src="https://github.com/user-attachments/assets/782134c0-37dd-4874-a0b9-d97786d54345" />

This database contained a collection called:

```text
flag
```

I viewed the contents of the collection and retrieved the flag.

<img width="841" height="235" alt="flag" src="https://github.com/user-attachments/assets/e40334a5-063c-4853-b43a-8d16ad3b3070" />

No further exploitation was necessary because the flag was directly accessible through the exposed database service.

## Conclusion

Mongod was a straightforward machine that demonstrated the risks associated with exposing a database service without proper authentication and access controls.

The initial Nmap scan identified two open ports: SSH on port 22 and MongoDB on port 27017. Since MongoDB was directly accessible, I focused my enumeration on the database service.

Using mongosh, I connected to the MongoDB instance with:

```bash
mongosh "mongodb://10.129.228.30:27017"
```

The connection was successful, allowing me to interact with the database and enumerate its contents.

I discovered five databases: admin, config, local, sensitive_information, and users. The users database contained an ecommerceWebapp collection with usernames, email addresses, account creation dates, and hashed passwords.

I then examined the sensitive_information database and discovered a collection named flag. Viewing this collection revealed the flag, completing the machine.

The main techniques demonstrated by this machine were:

- Full TCP port enumeration with Nmap

- Identifying MongoDB on port 27017

- Connecting to MongoDB using Mongosh

- Enumerating MongoDB databases

- Enumerating database collections

- Extracting sensitive user information

- Identifying an exposed flag collection

- Retrieving a flag from MongoDB

Overall, Mongod demonstrates how an improperly secured database can expose sensitive application data to unauthorized users.
Proper authentication, network restrictions, and access controls should be implemented on database services to prevent unauthorized access to sensitive information.
