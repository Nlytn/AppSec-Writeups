# [OverTheWire] - [Bandit1]

**Date:** 10/7/2025  
**Difficulty:** Easy  
**Objective:** Log into the game using SSH.

---

## Step 1: Recon / Information Gathering
This first level is relatively simple. We need to login to the game via SSH (Secure Shell). 

Secure Shell is a technology that allows secure remote connections. This allows secure communication and file transfer employing robust security mechanisms for safety. As long as the service is installed on both sides, you should be able to connect with either a password, passphrase or SSH key.

![Screenshot of challenge text](/Assets/bandit0.png)

So once we login, we can get the next level password and really start crackin'. That sounds like a plan. Before we proceed, do you know how to login via ssh?

---

## Step 2: Exploit / Solution

Logging in via SSH is relatively simple, especially given that we already have the username/password combo for the challenge. 

Logging in via ssh is as simple as this:

	ssh -p <port number> <username>@<hostname or IP address>

So in our case, its as simple as:

	ssh -p 2220 bandit0@bandit.labs.overthewire.org

![Screenshot of login terminal](/Assets/bandit0_login.png)
![Screenshot of password prompt](/Assets/bandit0_password.png)

So now let's reference the rest of the challenge to see where to go next. According to the challenge, the password is in a "readme" in the home directory. Let's start by listing the home directory contents. We can do this by running an *ls* command:

![Screenshot of home directory contents](/Assets/bandit0_home.png)

Nice! So from here, let's open the file and get that password. We have two options (really there are more, but these are the two I default to): *cat* and *nano*. There isn't a wrong answer between these or others, but since we just need to read I'm going to use *cat*:

![Screenshot of home readme contents](/Assets/bandit0_cat.png)

And there's the password. Let's grab that, save it, and move on to the next level. Make sure you terminate your existing SSH connection before moving on. 

---

## Step 3: Lessons Learned
- How to properly SSH to a remote host on an alternate port using password authentication 


---

## References
- [Link to the challenge](https://overthewire.org/wargames/bandit/bandit0.html)  
- [SSH Documentation](https://www.ssh.com/academy/ssh/command)
