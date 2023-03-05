
# UltraTech CTF Walkthrough

UltraTech is a vulnerable server on TryHackMe.com<br />
<br />
<i>URL: https://tryhackme.com/room/ultratech<br />
THM Difficulty: Medium<br />
My Difficulty: <br />
Target IP: <br />
AttackUltraTech IP: 10.10.157.51<br />
</i>

# Recon

For our first command we will use rustscan, rustscan will scan all of the ports in around 3 seconds.<br />
```rustscan -a 10.10.105.97```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT1.png?raw=true)<br />
Now we know which ports are open, I run an Nmap scan with the flag -sV to find the service versions on the open port.<br />
```nmap -sV -p80 10.10.105.97```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT2.png?raw=true)<br />
We have only found one service, this CTF may just be a webservice. Let's try to find some vulnerabilities!

## ??? Service Recon



## Questions & Answers

Question 1:<br />
**Which software is using the port 8081** <br />

Question 2:<br />
**Which other non-standard port is used?** <br />

Question 2:<br />
**Which software is using this port?** <br />

Question 2:<br />
**Which GNU/Linux distribution seems to be used?** <br />

Question 2:<br />
**The software using the port 8081 is a REST api, how many of its routes are used by the web application? ** <br />

Question 2:<br />
**There is a database lying around, what is the filename?** <br />

Question 2:<br />
**What is the first user's password hash?** <br />

Question 2:<br />
**What is the password associated with this hash?** <br />

Question 2:<br />
**What are the first 9 characters of the root users private SSH key?** <br />

### Farewell, I hope you all enjoyed this write-up!

