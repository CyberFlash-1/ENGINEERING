# Meow – Hack The Box Write-up

**Difficulty:** Very Easy  
**OS:** Linux  
**IP Address:** `10.10.10.246`  
**Author:** Hack The Box  
**Tags:** `beginner`, `telnet`, `linux`, `simple enumeration`

---

## 📝 Summary

The Meow machine on HTB is a very simple box that introduces new users to the platform. It involves connecting to an open **Telnet** service and retrieving the flag. No exploitation or complex enumeration is needed.

---

## 📡 Reconnaissance

### Nmap Scan

nmap -sV  10.10.10.246


```
telnet 10.129.86.117

```
Trying 10.129.86.117..
Connected to 10.129.86.117.
Escape character is '^]'.

Welcome to Meow!
Please enter your command:

```
ls
cat flag.txt
```
- to list the files
- then show the content of the file using cat



![image](https://github.com/user-attachments/assets/40e5933e-a865-41f3-a106-e1fbeb23930a)
