
# Ignite CTF Walkthrough

Ignite is a vulnerable new start up with issues in their web server on TryHackMe.com<br />
<br />
<i>URL: https://tryhackme.com/room/ignite<br />
THM Difficulty: Easy<br />
My Difficulty: Easy<br />
Target IP: 10.10.105.97<br />
AttackBox IP: 10.10.189.128<br />
</i>

## Enumeration

For our first command we will use rustscan, rustscan will scan all of the ports in around 3 seconds.<br />
```rustscan -a 10.10.105.97```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT1.png?raw=true)<br />
Now we know which ports are open, I run an Nmap scan with the flag -sV to find the service versions on the open port.<br />
```nmap -sV -p80 10.10.105.97```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT2.png?raw=true)<br />
We have only found one service, this CTF may just be a webservice. Let's try to find some vulnerabilities!

## HTTP Service

Let's load up our web browser and head over to http://10.10.105.97/<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT3.png?raw=true)<br />
Straight away we see the name "Fuel CMS" and a version "Version 1.4". Let's see what else we can find.
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT4.png?raw=true)<br />
Further down the webpage we see the link to an admin page and a default login, let's try logging in!
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT5.png?raw=true)<br />
Success! We've managed to login to the admin panel with the default username and password. Let's check the search engines for Fuel CMS 1.4 to see if we can find any vulnerabilities.
Another hit, we see an exploit-db result that affects Fuel CMS versions 1.41 and below. CVE-2018-16763 Remote Code Execution.
Now we have two ways we can go, we're going to try the first way... Using the exploit we found! Why not?
First let's download the exploit onto our AttackBox - https://www.exploit-db.com/raw/50477
I downloaded it and named it exploit.py, let's run it and see if we can get a shell! This exploit requires no additional configuration.
python exploit.py
It tells us we need to add the -u flag with the URL! Let's go.
python exploit.py -u http://10.10.105.97
If the exploit was succesful you will be asked to enter a command like I've done below
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT6.png?raw=true)<br />





Question 1:<br />
**What is the user flag?** <br />

Question 2:<br />
**What is the root flag?** <br />

### Farewell, I hope you all enjoyed this write-up!

