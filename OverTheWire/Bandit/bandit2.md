# [OverTheWire] - [Bandit2]

**Date:** 10/9/2025  
**Difficulty:** Easy  
**Objective:** The password for the next level is stored in a file called - located in the home directory

---

## Step 1: Recon / Information Gathering
Since we have the password from the previous challenge, we can ssh to the same host using the new username/password combination. To see how to connect via SSH, refer to my previous post: ![Bandit0 Walkthrough](/bandit0.png) 

![Screenshot of challenge text](/Assets/bandit1.png)

---

## Step 2: Exploit / Solution
Now that we've logged in, let's check the home directory. Let's once more run a list command to figure out what is in the home directory.

![Screenshot of list contents](/Assets/bandit1_ls.png)

Okay, there it is. One simple filed named - . That should be simple enough to open, right? So let's try to cat this file and get that password:

![Screenshot of reading file](/Assets/bandit1_cat1.png)

Since a screenshot doesn't show a time lapse very well, rest assured I waited for a solid 2-3 minutes at this screen with no improvement. It looks like it may have frozen. So let's dig a little deeper into this.

I had to search around to find an answer on how to open this file. I'm sure there is a reason why someone would name their file starting with a hyphen, but those reasons escape me currently. It took a little searching, but apparently we can open the file with a small syncatical change:

![Screenshot of reading file](/Assets/bandit1_cat2.png)

When a file begins with special characters, the best way is to specify the absolute path to the file. 

---

## Step 3: Lessons Learned
- How to open file with unconventional naming convention  

---

## References
- [Link to the challenge](https://overthewire.org/wargames/bandit/bandit2.html)  
- [Link to forum post detailing how to open file](https://serverfault.com/questions/124659/how-can-i-open-a-file-whose-name-starts-with)