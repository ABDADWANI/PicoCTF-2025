# **Write-up for Web CTF Challenge: "3v@l"**

---

## **Challenge Overview**

### **Challenge Name:** 3v@l  
### **Category:** Web Exploitation  
### **Description:**  

ABC Bank's website includes a loan calculator that evaluates user input using Python’s `eval()` function. This function is known for its security risks, as it can lead to **Remote Code Execution (RCE)** if not properly sanitized. The objective of this challenge is to exploit this vulnerability and read the flag stored in `/flag.txt`.

---

## **Understanding the Vulnerability**

The `eval()` function is designed to evaluate strings as Python expressions. If user input is passed into `eval()` without strict filtering, an attacker can execute arbitrary Python commands.  

For example:  
```python
eval("2 + 2")  # Output: 4
eval("print('Hello')")  # Output: Hello
```
This becomes dangerous when an attacker can inject their own Python code. In this challenge, we need to work around input restrictions to achieve **RCE**.

---

## **Analyzing Input Restrictions**

The challenge applies several security filters to block common attack patterns:  

- **Blocked keywords:**  
  ```
  os, eval, exec, bind, connect, python, socket, ls, cat, shell
  ```
  These keywords are commonly used for command execution and file manipulation.

- **Regular expression filter:**  
  ```
  r'0x[0-9A-Fa-f]+|\\u[0-9A-Fa-f]{4}|%[0-9A-Fa-f]{2}|\.[A-Za-z0-9]{1,3}\b|[\\\/]|\.\.'
  ```
  This regex blocks:
  - Hex encoding (`0x...`)
  - Unicode encoding (`\uXXXX`)
  - URL encoding (`%xx`)
  - File path characters (`/`, `\`, and `..`)

Since `eval()` is still being used, we can attempt to bypass these restrictions by dynamically constructing our payload.

---

## **Exploitation Strategy**

### **Step 1: Accessing Python’s Built-in Functions**
Since direct use of `open()` is blocked, we look for alternative ways to access it. Python’s built-in functions are stored in `__builtins__`, allowing us to retrieve `open()` dynamically:

```python
getattr(__builtins__, "open")
```
This retrieves the `open()` function without needing to reference it explicitly.

---

### **Step 2: Constructing the File Path**
Since `/flag.txt` cannot be written directly due to the filtering rules, we reconstruct it using the `chr()` function:

- `/` (ASCII 47) → `chr(47)`
- `.` (ASCII 46) → `chr(46)`

Rewriting `/flag.txt` using `chr()`:

```python
chr(47) + "flag" + chr(46) + "txt"
```
This dynamically generates `"/flag.txt"` without using restricted characters.

---

### **Step 3: Reading the Flag**
Now that we have a method to access `open()` and construct the file path, we combine them into a working payload:

```python
getattr(__builtins__, chr(111) + chr(112) + chr(101) + chr(110))(chr(47) + 'flag' + chr(46) + 'txt').read()
```

Breaking it down:
- `chr(111) + chr(112) + chr(101) + chr(110)` reconstructs `"open"`.
- `chr(47) + 'flag' + chr(46) + 'txt'` reconstructs `"/flag.txt"`.
- This is equivalent to:  
  ```python
  open("/flag.txt").read()
  ```
  but written in a way that bypasses filters.

---

### **Step 4: Executing the Exploit**
Entering the above payload into the loan calculator input field returns:

```
picoCTF{D0nt_Use_Unsecure_f@nctions6798a2d8}
```

This confirms that we successfully exploited the `eval()` vulnerability to achieve **remote code execution** and retrieve the flag.
