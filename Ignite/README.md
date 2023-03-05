
# Ignite CTF Walkthrough

Ignite is a vulnerable new start up with issues in their web server on TryHackMe.com<br />
<br />
<i>URL: https://tryhackme.com/room/ignite<br />
THM Difficulty: Easy<br />
My Difficulty: Easy<br />
Target IP: 10.10.105.97<br />
AttackBox IP: 10.10.<br />
</i>

## Enumeration

For our first command we will use rustscan, rustscan will scan all of the ports in around 3 seconds.<br />
```rustscan -a 10.10.90.192```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT1.png?raw=true)<br />
Now we know which ports are open, I run an Nmap scan with the flag -sV to find the service versions on the open ports.<br />
```nmap -sV -p22,80,110,139,143,445 10.10.90.192```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Ignite/screenshots/SCREENSHOT2.png?raw=true)<br />
We have found a few services that could be interesting... Let's try to find some vulnerabilities in the services.

Question 1:<br />
**What is the user flag?** <br />

Question 2:<br />
**What is the root flag?** <br />

### Farewell, I hope you all enjoyed this write-up!

