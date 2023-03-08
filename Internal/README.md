
# Internal CTF Walkthrough

Internal is a vulnerable server on TryHackMe.com<br />
<br />
<i>URL: https://tryhackme.com/room/Internal<br />
THM Difficulty: Hard<br />
My Difficulty: TBA<br />
Target IP: 10.10.44.166<br />
AttackBox IP: 10.10.153.77<br />
</i>

# Enumeration

For our first command we will use rustscan, rustscan will scan all of the ports in around 3 seconds.<br />
```shell
rustscan -a 10.10.44.166
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT1.png?raw=true)<br />
Now we know which ports are open, I run an Nmap scan with the flag -sV to find the service versions on the open ports.<br />
```shell
nmap -sV -p80 10.10.44.166
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT2.png?raw=true)<br />


## ?? Service Enumeration



## Questions & Answers

Question 1:<br />
**What is the user flag?** <br />

Question 2:<br />
**What is the root flag?** <br />

### Farewell, I hope you all enjoyed this write-up!

