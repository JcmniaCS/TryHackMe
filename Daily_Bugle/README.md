
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
Here we have our answer to the first question! 




## Questions & Answers

Question 1:<br />
**Access the web server, who robbed the bank?** Spiderman<br />

Question 2:<br />
**What is the Joomla version?** <br />

Question 3:<br />
**What is Jonah's cracked password?** <br />

Question 4:<br />
**What is the user flag?** <br />

Question 5:<br />
**What is the root flag?** <br />

### Farewell, I hope you all enjoyed this write-up!

