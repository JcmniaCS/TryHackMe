
# Box CTF Walkthrough

Box is a vulnerable server on TryHackMe.com<br />
<br />
<i>URL: https://tryhackme.com/room/box<br />
THM Difficulty: <br />
My Difficulty: <br />
Target IP: <br />
AttackBox IP: <br />
</i>

# Recon

For our first command we will use rustscan, rustscan will scan all of the ports in around 3 seconds.<br />
```rustscan -a 10.10.105.97```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Box/screenshots/SCREENSHOT1.png?raw=true)<br />
Now we know which ports are open, I run an Nmap scan with the flag -sV to find the service versions on the open port.<br />
```nmap -sV -p80 10.10.105.97```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Box/screenshots/SCREENSHOT2.png?raw=true)<br />
We have only found one service, this CTF may just be a webservice. Let's try to find some vulnerabilities!

## ?? Service Recon



## Questions & Answers

Question 1:<br />
**What is the user flag?** <br />

Question 2:<br />
**What is the root flag?** <br />

### Farewell, I hope you all enjoyed this write-up!

