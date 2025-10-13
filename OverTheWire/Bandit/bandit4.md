# [OverTheWire] - [Bandit4]

**Date:** 2025-10-10  
**Difficulty:** Easy   
**Objective:** The password for the next level is stored in a hidden file in the inhere directory.

---

## Step 1: Recon / Information Gathering
As with the previous challenges, we need to connect to Bandit3 over ssh. Then we can start some enumeration and get our bearings.

![Screenshot of challenge text](/Assets/bandit4.png)

---

## Step 2: Exploit / Solution
Let's run a ls command and see what is listed in the home directory:

![Screenshot of home directory](/Assets/bandit4_ls.png)

Okay, simple enough. We have a directory listed here, called "inhere". So far that's about what we were told, so let's see what's in the directory.

![Screenshot of home directory](/Assets/bandit4_ls.png)

I'm going to drop a little more context for us here. Running your standard list (ls) command doesn't work here. That's because the file is hidden. Once we run ls with the operands -la then we can see the hidden file. 

    -l -> displays information about files/directories (long format)
    -a -> show all files, including hidden files

Technically you only need the -a switch, but I prefer to use both to get more information about our environment. 

Now we can just cat the file to get the next level password. 

![Screenshot of home directory](/Assets/bandit4_file_contents.png)


---

## Step 3: Lessons Learned
- How to find and open hidden files  
- Security through obscurity should not be your only layer of defense  


---

## References
- [Link to the challenge](https://overthewire.org/wargames/bandit/bandit4.html)   
- [Reference to Linux ls command](https://www.geeksforgeeks.org/linux-unix/ls-command-in-linux/)