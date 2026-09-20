# W1seGuy – Writeup (Easy)

**Machine:** W1seGuy  
**Difficulty:** Easy  
**Tech Used:** Netcat, CyberChef, Hex Encoding, XOR, Cryptanalysis  
**Objective:** Demonstrate how to identify and recover a repeating XOR encryption key and decode an encrypted flag.

**By:** Ce'Bria Haynes |  **Published:** 9/20/2026

---

## Overview

W1seGuy is an easy-level room focused on basic cryptography and understanding how XOR encryption works. The server listens on TCP port 1337 and provides an encrypted message when connected to using Netcat.

The ciphertext provided by the server was hex-encoded and then encrypted using XOR with a repeating key. The objective was to determine the encryption key and use it to decode the hidden flag.

I began by connecting to the server using Netcat. The server provided a ciphertext along with a message containing a fake flag. Since TryHackMe flags normally begin with `THM{`, I used the known beginning of the flag to perform a known-plaintext attack and recover the first four characters of the XOR key.

I then used CyberChef to decode the hexadecimal data and apply the recovered XOR key. The first four characters of the key were `R9ZC`. Since the ciphertext was encrypted using a repeating key, I determined that the key was longer than four characters. I brute-forced the final character by testing numbers and lowercase letters until the correct character was identified.

The final key was `R9ZCj`, which successfully decoded the ciphertext and revealed the first flag.

---

## Enumeration

### Connecting to the Server:

The room instructions stated that the server was listening on TCP port `1337` and could be accessed using Netcat.

I connected to the target using:

```bash
nc 10.67.177.20 1337
```
After connecting, the server returned the following message:

```text
This XOR encoded text has flag 1: 067117381a6358362d1e17412e021e260d392809135728700b3e75232b3f204d23731f2041153117

What is the encryption key? No way you got it! Here is your flag THM{Try_Again} :)
```

<img width="845" height="310" alt="webpage" src="https://github.com/user-attachments/assets/4600e2b7-d7e1-416c-bae1-b90c15247c33" />

The server explained that the text was XOR encoded, and the objective was to determine the encryption key.
Cryptography Analysis

The ciphertext was represented as hexadecimal characters. This meant that the first step was to convert the hexadecimal data back into bytes.

The ciphertext began with:

```text
06 71 17 38
```

Since TryHackMe flags generally begin with:

```text
THM{
```

I could use this known plaintext to determine the corresponding bytes of the XOR key.

The ASCII values for THM{ are:

```text
T = 54
H = 48
M = 4d
{ = 7b
```

XOR encryption follows the relationship:

```text
Ciphertext XOR Key = Plaintext
```

Therefore, I could recover the key by performing:

```text
Ciphertext XOR Plaintext = Key
```

I XORed each ciphertext byte with the corresponding known plaintext byte:

```text
06 XOR 54 = 52
71 XOR 48 = 39
17 XOR 4d = 5a
38 XOR 7b = 43
```

This gave me the first four bytes of the encryption key:

```text
52 39 5a 43
```

Converting those hexadecimal values to ASCII produced:

```text
R9ZC
```

I now know the first four characters of the key. However, the key appears to be longer than 4 characters.

## Decoding with CyberChef

I decided to use CyberChef to make the decoding process easier.

First, I entered the encrypted hexadecimal string into the input field.

I then added the From Hex operation to the recipe. This converted the hexadecimal ciphertext into its original byte representation.

<img width="959" height="383" alt="flag 1" src="https://github.com/user-attachments/assets/3476ec2e-118b-4083-a204-eb893cbd6391" />

Next, I added the XOR operation after From Hex.

For the XOR operation, I entered the key I had recovered so far:

```text
R9ZC
```

I then ran the recipe.

<img width="769" height="392" alt="xor attempt 1" src="https://github.com/user-attachments/assets/39c40324-1090-41a6-af3b-b3635a19d52a" />

The output began with the expected:

```text
THM{
```

This confirmed that R9ZC was part of the correct key.

However, the complete plaintext was not decoded correctly, indicating that the key contained an additional character.

## Brute Forcing the Final Character

Since I knew the first four characters of the key, I just needed to determine the final character.

I tested possible characters, including numbers and lowercase letters, until the output produced readable plaintext and a valid TryHackMe flag.

After testing the possible characters, I found that the missing character was:

```text
j
```

This gave me the complete XOR key:

```text
R9ZCj
```

I entered the completed key into the XOR operation in CyberChef.

<img width="959" height="383" alt="flag 1" src="https://github.com/user-attachments/assets/ff86ac1a-5db9-4730-a298-1f03bbf9e6a6" />

The ciphertext successfully decoded and revealed the first flag.

## Conclusion

W1seGuy was a simple introduction to XOR cryptanalysis and demonstrated how known plaintext can be used to recover an encryption key.

The server provided a hexadecimal ciphertext that had been encrypted using a repeating XOR key. By connecting to the service with Netcat, I was able to retrieve the ciphertext and begin analyzing it.

Because TryHackMe flags follow the predictable THM{ format, I used the first four characters of the expected plaintext to perform a known-plaintext attack. XORing the first four ciphertext bytes with the hexadecimal representation of THM{ revealed the first four characters of the key: R9ZC.

I then used CyberChef to convert the ciphertext from hexadecimal and apply the recovered XOR key. The decoded output began with THM{, confirming that the key was correct so far. Since the key was longer than four characters, I brute-forced the remaining character using numbers and lowercase letters.

The final key was R9ZCj, which successfully decoded the ciphertext and revealed the first flag.

The main techniques demonstrated by this machine were:

- Connecting to a TCP service using Netcat

- Identifying hexadecimal encoding

- Understanding XOR encryption

- Using known plaintext to recover an XOR key

- Converting hexadecimal data using CyberChef

- Applying a repeating XOR key

- Brute-forcing an unknown key character

- Decoding an encrypted flag

Overall, W1seGuy demonstrates how weaknesses in simple XOR encryption can be exploited when part of the plaintext is predictable.
Knowing that the plaintext begins with THM{ provided enough information to recover part of the key and eventually determine the complete encryption key.
