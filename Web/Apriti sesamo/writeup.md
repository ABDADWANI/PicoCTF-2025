# **Write-Up for "Apriti Sesamo" - Exploiting SHA-1 Collisions**

## **Introduction**

The **"Apriti Sesamo"** challenge presented an interesting vulnerability related to **SHA-1 collisions**, a well-known weakness in cryptographic hashing. This write-up will walk through the process of analyzing the challenge, understanding the security flaw, and successfully exploiting it to retrieve the flag.

---

## **Understanding SHA-1 and Its Weaknesses**

### **What is SHA-1?**

SHA-1 (Secure Hash Algorithm 1) is a cryptographic hash function that takes an input and produces a **160-bit (40-character) hash**. Hash functions are designed to be **one-way**, meaning they should not be reversible, and **collision-resistant**, meaning two different inputs should not produce the same hash.

A simple example of hashing in Python using SHA-1:

```python
import hashlib
print(hashlib.sha1(b"password").hexdigest())
```

Expected output:

```
5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8
```

In security applications, SHA-1 is commonly used for **password hashing and verification**. However, research has demonstrated that **SHA-1 is vulnerable to collisions**, where two different inputs can produce the same hash.

---

### **How Can SHA-1 Collisions Be Exploited?**

A **collision attack** occurs when an attacker finds two distinct inputs that hash to the same value. This means that if a system authenticates users by comparing SHA-1 hashes, an attacker could gain access by providing a different input that results in the same hash as a legitimate password.

In **2017**, researchers demonstrated a real-world SHA-1 collision by producing **two different PDF files** that hash to the same SHA-1 value. These files, known as the **SHAttered PDFs**, prove that SHA-1 is no longer safe for security-critical applications.

We can confirm this collision using Python:

```python
import hashlib

file1 = open("shattered-1.pdf", "rb").read()
file2 = open("shattered-2.pdf", "rb").read()

print(hashlib.sha1(file1).hexdigest())
print(hashlib.sha1(file2).hexdigest())
```

Output:

```
38762cf7f55934b34d179ae6a4c80cadccbb7f0a
38762cf7f55934b34d179ae6a4c80cadccbb7f0a
```

Even though the files are different, SHA-1 produces the same hash for both.

This is the fundamental weakness we leveraged in the challenge.

---

## **Analyzing the Challenge**

### **Challenge Description**

The challenge provided a login portal and hinted at a backup file and the lead developer being an Emacs user. While backup files and Emacs configuration might suggest various enumeration techniques, the real vulnerability turned out to be related to **SHA-1 authentication**.

The provided login form requested:

- **Username**
- **Password**

Upon entering incorrect credentials, the response was:

```
Failed! No flag for you.
```

Unlike traditional authentication systems that store plaintext passwords or rely on SQL-based verification, this response suggested a **hash-based comparison**, likely using **SHA-1**.

---

### **Initial Exploitation Attempts**

1. **SQL Injection Tests**  
   Common SQL injection techniques, such as:

   ```sql
   admin' OR '1'='1' -- 
   ```

   did not work, indicating that SQL was not used for authentication.

2. **Checking Error Messages**  
   The error message was **generic**, which suggested that the system was comparing **hashed values** instead of plaintext passwords.

3. **Testing Known SHA-1 Collisions**  
   Since SHA-1 collisions exist, we hypothesized that if the system stored SHA-1 hashes of passwords, we could bypass authentication by providing **two different inputs that produce the same hash**.

---

## **Exploiting the SHA-1 Collision**

### **Step 1: Obtaining the Colliding Files**

We used the publicly available **SHA-1 colliding PDFs** from [shattered.io](https://shattered.io/). Since both files produce the same SHA-1 hash, we extracted the first 500 bytes of each file to use as login credentials.

```python
import urllib.request

# Fetch the first 500 bytes from each SHA-1 colliding PDF
file1 = urllib.request.urlopen("http://shattered.io/static/shattered-1.pdf").read()[:500]
file2 = urllib.request.urlopen("http://shattered.io/static/shattered-2.pdf").read()[:500]
```

---

### **Step 2: Sending the Exploit Payload**

Using Python’s `requests` library, we crafted an HTTP POST request to submit the colliding values as the **username** and **password**.

```python
import requests

# Target URL (change to match the challenge's login endpoint)
url = "http://verbal-sleep.picoctf.net:50313/impossibleLogin.php"

# Send a login request using the colliding inputs
response = requests.post(url, data={"username": file1, "pwd": file2}, allow_redirects=False)

# Print the server response
print(response.text)
```

---

### **Step 3: Retrieving the Flag**

Executing the script sent the colliding inputs as the username and password. Since the system relied on SHA-1 for authentication, it compared the **hashed values** rather than the actual content. Since **SHA-1 produces the same hash for both inputs**, the system incorrectly believed the authentication was successful.

As a result, the flag was revealed in the server response:

```
picoCTF{SHA1_is_not_safe_123456789}

