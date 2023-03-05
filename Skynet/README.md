
# Skynet CTF Walkthrough

<p>Skynet is a vulnerable terminator themed linux machine on TryHackMe.com<br />
<br />
<i>THM Difficulty: Easy <br />
My Difficulty: <br />
Completion Time: </i><br />
</p>

## Enumeration
<p>For our first command we will use rustscan, rustscan will scan all of the ports in around 3 seconds.<br />
rustscan -a 10.10.90.192<br />
SCREENSHOT<br />
Now we know which ports are open, I run an Nmap scan with the flag -sV to find the service versions on the open ports.<br />
nmap -sV -p22,80,110,139,143,445 10.10.90.192<br />
SCREENSHOT2<br />
We have found a few services that could be interesting... I saw they have port 80 open running Apache httpd 2.4.18. 
I open up my browser and head over to the website http://10.10.90.192/<br />
SCREENSHOT3<br />
It appears to be some kind of search engine but none of the search features are working, I decide to start up gobuster
and try to find other directories on the webserver. I entered the command below to enumerate directories.<br />
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.90.192/<br />
SCREENSHOT4<br />
After trying to access each directory, I receive a forbidden page from every directory other than /squirrelmail
Checking out this page I see a version 1.4.23, I do a little research to find out if this version is vulnerable...
Success! I can see from a simple google search that this version of squirrelmail is vulnerable to Remote Code Execution (RCE) 
However, we have a problem... After reading the exploit I realize you need to be an authenticated user... 
Let's go back to our Nmap scan results and try looking at one of the other interesting services for now.<br /></p>
<p>test</p>






Question 1:
**What is Miles password for his emails?**
