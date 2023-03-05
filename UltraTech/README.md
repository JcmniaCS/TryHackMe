
# UltraTech CTF Walkthrough

UltraTech is a vulnerable server based on real-life on TryHackMe.com<br />
<br />
<i>URL: https://tryhackme.com/room/ultratech<br />
THM Difficulty: Medium<br />
My Difficulty: <br />
Target IP: 10.10.39.165<br />
AttackUltraTech IP: 10.10.157.51<br />
</i>

# Recon

For our first command we will use rustscan, rustscan will scan all of the ports in around 3 seconds.<br />
```rustscan -a 10.10.39.165```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT1.png?raw=true)<br />
Now we know which ports are open, I run an Nmap scan with the flags -sV and -sC to find the service versions and to run default scripts on the open ports.<br />
```nmap -sV -sC -p21,22,8081,31331 10.10.39.165```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT2.png?raw=true)<br />
We see 4 ports open running different services! Let's have a look into some of the services and see what we can find.<br />

## FTP Service Recon

We're going to start with FTP, let's try to connect via FTP with the anonymous account <br />
```ftp 10.10.39.165```<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT3.png?raw=true)<br />
No luck there, let's move on for now.<br />

## HTTP Service Recon

We've moved onto HTTP, let's head over to the website http://10.10.39.165:31331 <br />
Nothing really of notice, there's an e-mail address on the contact form of the bottom of the webpage, we'll save it for now. 
Let's try to bruteforce some directories with gobuster<br />
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.39.165:31331/<br />

Nothing really of interest, let's try something else.<br />

## 


## Questions & Answers

Question 1:<br />
**Which software is using the port 8081** Node.js<br />

Question 2:<br />
**Which other non-standard port is used?** 31331<br />

Question 2:<br />
**Which software is using this port?** Apache<br />

Question 2:<br />
**Which GNU/Linux distribution seems to be used?** Ubuntu<br />

Question 2:<br />
**The software using the port 8081 is a REST api, how many of its routes are used by the web application? ** 2<br />

Question 2:<br />
**There is a database lying around, what is the filename?** <br />

Question 2:<br />
**What is the first user's password hash?** <br />

Question 2:<br />
**What is the password associated with this hash?** <br />

Question 2:<br />
**What are the first 9 characters of the root users private SSH key?** <br />

### Farewell, I hope you all enjoyed this write-up!
