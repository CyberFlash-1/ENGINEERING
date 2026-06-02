# **FAWN – Hack The Box Write-up**

**Difficulty:** Very Easy  
**OS:** Linux  
**IP Address:** `10.129.73.183`  
**Author:** Hack The Box  
**Tags:** `beginner`, `telnet`, `linux`, `simple enumeration`

---

What does the 3-letter acronym FTP stand for?  
**Answer:** File Transfer Protocol

Which port does the FTP service listen on usually?  
**Answer:** 21

FTP sends data in the clear, without any encryption. What acronym is used for a later protocol designed to provide similar functionality to FTP but securely, as an extension of the SSH protocol?  
**Answer:** SFTP

What is the command we can use to send an ICMP echo request to test our connection to the target?  
**Answer:** `ping`

```bash
ping 10.129.73.183
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/5972e277-8224-4790-905b-26422b3d6129" alt="Ping Output">
</p>

From your scans, what version is FTP running on the target?

```bash
nmap -sV 10.129.73.183
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/97511f5b-69e9-41bc-87d7-edb7a3c0a0f2" alt="Nmap Output">
</p>

**Answer:** vsftpd 3.0.3

From your scans, what OS type is running on the target?  
**Answer:** Unix

What is the command we need to run in order to display the 'ftp' client help menu?  
**Answer:** `ftp -?`

What is the username used to login anonymously to FTP?  
**Answer:** Anonymous

What is the response code we get for the FTP message 'Login successful'?  
**Answer:** 230

To install the FTP package if not already installed:

```bash
sudo apt install ftp -y
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/a278e0cb-6ab1-4568-a071-534d3c124e98" alt="FTP Install">
</p>

There are a couple of commands we can use to list the files and directories available on the FTP server. One is `dir`. What is the other that is a common way to list files on a Linux system?  
**Answer:** `ls`

What is the command used to download the file we found on the FTP server?  
**Answer:** `get`

<p align="center">
  <img src="https://github.com/user-attachments/assets/e3bd0ef5-1cd9-4f22-bbb5-f7ba9a9bdd3c" alt="FTP Get Command">
</p>

Download the file to your host using:

```bash
get flag.txt
```

Exit the FTP session:

```bash
bye
```

You will receive:

```
221 Goodbye
```

To read the flag:

```bash
cat flag.txt
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/97c0e3b2-7daf-415c-a138-87dcdaf3a43e" alt="Flag Output">
</p>
