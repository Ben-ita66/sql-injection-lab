# 💉 SQL Injection Lab – Attacking a MySQL Database

A hands-on lab documenting how a SQL injection attack unfolds at the packet level using Wireshark on a CyberOps Workstation VM.

---

## 🧪 Lab Overview

| Detail | Info |
|---|---|
| **Tool** | Wireshark |
| **Environment** | CyberOps Workstation VM |
| **Attack Type** | SQL Injection |
| **Target Application** | DVWA (Damn Vulnerable Web App) |
| **IPs Involved** | Attacker: `10.0.2.4` / Target: `10.0.2.15` |
| **Capture Duration** | ~8 minutes (441 seconds) |

---

## 📁 Structure

```
sql-injection-lab/
├── README.md
├── screenshots/        # Wireshark captures from the lab
└── analysis/
    └── sql-injection-analysis.md   # Full breakdown of the attack
```

---

## 🗂️ Parts

- [Full Attack Analysis](analysis/sql-injection-analysis.md)

---

## 🔐 Key Takeaways

- SQL injection exploits poor input validation to manipulate database queries
- An attacker can extract database names, versions, table names, and password hashes
- Captured hashes can be cracked using tools like CrackStation
- Prevention requires parameterized queries, input validation, and least privilege principles

---

## 🛠️ Tools Used

- Wireshark
- CyberOps Workstation VM
- DVWA (Damn Vulnerable Web Application)
- CrackStation (hash cracking)

---

> Part of my ongoing cybersecurity learning journey. Documenting everything in public.  
> Follow along on [LinkedIn](https://www.linkedin.com/in/benitanwabueze) | [X @Ogechee_](https://x.com/Ogechee_)
