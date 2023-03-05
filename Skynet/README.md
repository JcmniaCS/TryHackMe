
# Skynet CTF Walkthrough

<p>Skynet is a vulnerable terminator themed linux machine on TryHackMe.com<br />
<br />
<i>THM Difficulty: Easy <br />
My Difficulty: <br />
Completion Time: </i><br />
</p>

## Enumeration

<p>For our first command we will use rustscan, rustscan will scan all of the ports in around 3 seconds.<br />
rustscan -a 10.10.90.192<br />
SCREENSHOT<br />
Now we know which ports are open, I run an Nmap scan with the flag -sV to find the service versions on the open ports.<br />
nmap -sV -p22,80,110,139,143,445 10.10.90.192<br />
SCREENSHOT2<br /></p>

## Finding Vulnerabilities

<p>We have found a few services that could be interesting... Let's try to find some vulnerabilities in the services.</p>

### HTTP Service

<p>I saw they have port 80 open running Apache httpd 2.4.18. 
I open up my browser and head over to the website http://10.10.90.192/<br />
SCREENSHOT3<br />
It appears to be some kind of search engine but none of the search features are working, I decide to start up gobuster
and try to find other directories on the webserver. I entered the command below to enumerate directories.<br />
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.90.192/<br />
SCREENSHOT4<br />
After trying to access each directory, I receive a forbidden page from every directory other than /squirrelmail
Checking out this page I see a version 1.4.23, I do a little research to find out if this version is vulnerable...
Success! I can see from a simple google search that this version of squirrelmail is vulnerable to Remote Code Execution (RCE) 
However, we have a problem... After reading the exploit I realize you need to be an authenticated user... 
Let's go back to our Nmap scan results and try looking at one of the other interesting services for now.<br /></p>

### Samba Service

<p>We can see in our Nmap scan that they have ports 139 and 445 open for a Samba. Let's try to enumerate the shares 
and see if there's anything we can access.<br />
smbclient -L 10.10.90.192<br />
SCREENSHOT5<br />
We were able to list the shares on the server without a password for root, we can see we have access to the shares "anonymous" and "milesdyson".
Let's try to connect to them now and see if they have any files of interest!
smbclient \\\\10.10.90.192\\anonymous<br />
Success! We were able to connect to the anonymous share without a password. Now we need to list the files avaialable on the share. <br />
ls<br />
SCREENSHOT6<br />
We can see we have a file named "attention.txt" and a directory named "logs" let's get the file attention.txt and then
change to the "logs directory" and list the files<br />
get attention.txt<br />
cd logs<br />
ls<br />
SCREENSHOT7<br />
We can see we have another 3 files inside of the "logs" directory, log1.txt, log2.txt, log3.txt. 
Let's get the files and check out our other share.<br />
get log1.txt<br />
get log2.txt<br />
get log3.txt<br />
exit<br />
smbclient \\\\10.10.90.192\\milesdyson<br />
We receive the error "NT_STATUS_ACCESS_DENIED" when trying to connect to the share milesdyson with an empty password for root. 
Let's try reading the files we got from the "anonymous" share. "attention.txt" "log1.txt" "log2.txt" "log3.txt"<br />
cat attention.txt<br />
SCREENSHOT8<br />
Apparently a recent malfunction has caused passwords to be changed, we also get the name "Miles Dyson" A possible username?<br />
cat log1.txt<br />
SCREENSHOT9<br />
log1.txt looks like it's full of passwords or usernames. Maybe we could try the possible username "Miles" or "MilesDyson" with this password list? 
I see there's nothing inside of the log2.txt and log3.txt so I remove those.<br /></p>

<p>Let's try bruteforcing the user "milesdyson" with the passwords we got from "log1.txt" on the squirrelmail service. 
We know there's a vulnerability in the SquirrelMail service with an available exploit. To do this we will use hydra and burp suite.<br />
hydra -l milesdyson -P log1.txt 10.10.90.192 http-post-form "/squirrelmail/src/login.php:login_username=^USER^&secretkey=^PASS^&js_autodetect_results=1&just_logged_in=1:F=Unknown user:H=Cookie: squirrelmail_language=en_US; SQMSESSID=jbgcof2ofcgqh0jb5ukapj8pu3;"<br />
Success! We have successfuly logged in with the password "cyborg007haloterminator"<br /></p>
Question 1:<br />
**What is Miles password for his emails?** cyborg007haloterminator

## Exploiting Vulnerabilities

<p>Now that we have a login for the SquirrelMail server milesdyson:cyborg007haloterminator, we know about a vulnerability in this version(1.4.23) and we know about an exploit for this service(CVE-2017-7692) 
Let's get to it!</p>

### Exploiting SquirrelMail

<p>Firstly, let's login with the aquired username:password. We are greeted with the screen below.<br />
SCREENSHOT10<br />
Before we exploit the vulnerability, let's have a look around and see if we can find anything interesting...<br />
The first thing that comes to my attention is the e-mails, let's have a look at each e-mail and see what's inside.<br />
We see inside the bottom e-mail what looks like ????, let's decode it and see what it means...<br /><br />

We see inside the second e-mail what looks like binary code, let's decode it and see what it means...<br />
After converting the binary we see the string "balls have zero to me to me to me to me to me to me to me to me to"<br /><br />

Inside the top e-mail we see a password reset for Samba, maybe we can access the milesdyson share with this password? ")s{A&2Z=F^n_E.B`" <br />
Let's try connecting to the share... <br />
smbclient \\\\10.10.90.192\\milesdyson -U milesdyson<br />
Success! We managed to login to the SMB server, let's see what files are available on the share.<br />
ls<br />
SCREENSHOT11<br />
Nothing really stands out to us here except from the directory "notes" Let's change our directory and list what's inside.<br />
cd notes<br />
ls<br />
SCREENSHOT12<br />
There's an interesting file amongst all of the random files, let's download and take a look at "important.txt"<br />
get important.txt<br />
exit<br />
cat important.txt<br />
SCREENSHOT13<br />
Oooh! An interesting find... It looks like a directory on the webserver. It's probably the answer to our second question!</p>
**What is the hidden directory?** /45kra24zxs28v3yd <br />


