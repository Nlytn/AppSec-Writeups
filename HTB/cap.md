<p align="center">
  <img src="Assets/HtB/Cap/cap_title_card.png" alt="Sea Machine Card" width="400">
</p>


## :aerial_tramway: Attack Path Summary
- Foothold through IDOR vulnerability  
- Privilege escalation to user via plaintext username/password  
- Root via cap_setuid misconfiguration 

## :dart: Objective
Cap is an easy rated machine, and an introductory course to guided mode on HtB. We start with a simple web server hosting a scanning and monitoring service. Leveraging insecure functionality, we discover an IDOR vulnerability that we leverage to gain ultimately discover a username and password. Upon gaining access, we leverage a binary with loose permissions to elevate our access to root.

## :bulb: Key Concepts / What You’ll Learn
- Enumeration Tactics
	- Initial TCP scan to identify open ports
	- Perform full service & version detection
	- Enumerate web technologies through headers & tooling
	- Fully browsing web page for clues, hints, or foothold  
- Vulnerability Type
	- IDOR (Insecure Direct Object Reference)
	- Improper cap_setuid capability on binary  
- Tools Used (New & Familiar)
	- nmap
	- Wireshark
	- Python
	- linPEAS

## :gear: Initial Foothold
Running an initial nmap scan, it appears we have three ports open: FTP, SSH, and a web server.

![screenshot of nmap results](/Assets/HtB/Cap/cap_nmap.png)

Let's notate these findings for now; first thing to do is to check out the web server.

![screenshot of browser main page](/Assets/HtB/Cap/web_browser_monitor_page.png)

By looking around on the page, we can find an option to download a .pcap file. Opening this file opens in Wireshark for analysis.

![screenshot of wireshark analysis](/Assets/HtB/Cap/wireshark_analysis.png)

So we have a packet capture of all of our traffic to the server thus far, but there doesn't seem to be much else here. Going back to the browser, it looks like this file is referenced directly within the URL. That means we may be able to perform an IDOR (Insecure Direct Object Reference). So we may be able to find and access another capture, even though we theoretically shouldn't be able to get to the page. 

We are downloading /id/2, so if we change to /id/1, we get another capture. However, this one appears to be empty. Going back one more, we can find another .pcap file at /id/0. Download this for further analysis.

![screenshot of 0.pcap](/Assets/HtB/Cap/pcap0.png)

There are likely more efficient ways to peform this task, but I quickly scrolled through the captures until I found something of interest. In this case, it was a prompt for a username, then another prompt for the password. On line 40 I found the password for nathan:

![screenshot of captured password](/Assets/HtB/Cap/captured_password.png)

Reflecting back on our nmap scan, there are two other services that were available: FTP and SSH. Trying this username/password combination on either of these services allows access. Connecting via ssh will give more control and fluidity with commands, so I chose that route. Then we can read user.txt and move on to the next step.

![screenshot of grabbing user.txt](/Assets/HtB/Cap/ssh_user_txt.png)

## :gear: Root Privilege Escalation
Now things get a little trickier. I tried running:

	sudo -l

But apparently Nathan is unable to run anything as sudo. Fortunately, there is another enumeration script we can run: linPEAS. Once we copy this file to the server and run the script, it will enumerate for potential paths to escalate privileges. As with any tool, its not perfect but it can give us some valuable insight if we're lucky.

First, we have to copy the script from our machine to the remote server:

	scp <PATH_TO_LOCAL_FILE> <SERVER_USER@SERVER_IP_ADDRESS>:<PATH_TO_DESTINATION_FOLDER>

![screenshot of copying linpeas over](/Assets/HtB/Cap/copying_linpeas.png)
![screenshot of running linpeas](/Assets/HtB/Cap/running_linpeas.png)

It's worth reviewing the key at the beginning. LinPEAS is color coded for convenience, and this key give you an idea of what to look for as you peruse the data. Since there is typically a lot of data returned, its incredibly helpful to have the color coding.

Scrolling down a bit, I notice what appears to be the golden ticket:

![screenshot of linpeas results](/Assets/HtB/Cap/linpeas_results.png)

A quick search returns a PoC for this exploit. Its a simple python script that will leverage polkit to add a new user with sudo privileges, so let's try it out. Since this is a HtB machine, it doesn't have external internet access. So we can download this exploit to our local machine, then set up a python web server to serve the file. From the remote (victim) machine, issue a wget back to our web server to get the python file:

![screenshot of web server](/Assets/HtB/Cap/python_virtual_server.png)
![screenshot of exploit on server](/Assets/HtB/Cap/exploit_on_server.png)

Remember to give the script executable permissions:

	chmod +x [filename]

![screenshot of exploit on server](/Assets/HtB/Cap/exploit_failed.png)

It looks like linPEAS lied to us; this exploit isn't working due to invalid dependencies. I'll run linPEAS again. 

Scrolling down to the "Capabilities" section, there is another highlight to check out. This one turns out to be more promising.

![screenshot of new linpeas results](/Assets/HtB/Cap/linpeas_results2.png)

So we have binary with some special permissions. 

* cap_net_bind_service - binds specified service to a port with elevated permissions. So you can run one service as sudo without having to specify this every time.
* cap_setuid - among other things, this allows us to write a user id mapping in a user namespace. Basically, this lets us change our user id or group id. In this case, we are focusing on the user id. 

By exploiting cap_setuid, we can set our user to root. Then we will elevate to a shell and grab root.txt. Since this permission is set on the python executable, we'll have to run a little code to get where we need to be.

Open the python interpreter on the remote machine:

	/usr/bin/python3.8

I referenced the full path to be sure it grabbed the right binary. I'm not sure if there is another Python binary in the PATH or not, but this ensures we grab the one with potentially elevated permissions.

From here, I had to play around with the code to figure out what it needed. I know python has a module to run os commands so this is the first thing I tried. The problem I ran into is that running python this way still runs it under our normal user account (nathan), whereas I need to be root to read /root/root.txt. 

Let's set the uid to 0 in python, spawn a shell and get root.txt

![screenshot of new privesc](/Assets/HtB/Cap/privesc.png)

## :brain: Understanding the Technique
After running an initial nmap scan, I discovered a web server running a scanning/monitoring service. Accessing this service allowed me to download a network packet capture file that showed all of my own traffic. By manipulating the URL, I was able to access what appears to be a master packet capture file. Since no safeguards were in place, I was able to download and review this file wherein I discovered a plaintext username/password combination. At this point I pivoted to other running services in an attempt to recycle the account information and successfully gained entry to the server. Running an imported binary (linPEAS), I was able to determine the cap_setuid capability was enabled on a Python binary but accessible to all users. By running this binary I was able to set user permissions to root, spawn a shell and access root.txt

## :lock: Security Takeaway
One or two sentences showing real-world significance.

## :writing_hand: Final Thoughts
This is another challenge that continues to highlight my weaknesses. Enumeration continues to be a sticking point with me, as does privilege escalation. However, I'm learning new tricks and tools to grow my abilities. As the skills grow, so too does the momentum. 

## :recycle: Notes
Completed: 2025-11-25  
Machine: https://app.hackthebox.com/machines/Cap

This is another early box in a long series of learning experiences on HtB, and as such I am leaning heavily on the shoulders of giants in this industry. For this walkthrough, I had to reference IppSec quite a bit to gain full understanding of attack methods and how to abuse the underlying technologies. As this series goes forward my hope is to stand on my own two feet and, with a little bit of luck, maybe stand with these giants one day. Until that day comes, I believe in giving credit where it is due:

[Link to IppSec walkthrough]https://youtu.be/O_z6o2xuvlw
[Link to 0xdf walkthrough](https://0xdf.gitlab.io/2021/10/02/htb-cap.html)
