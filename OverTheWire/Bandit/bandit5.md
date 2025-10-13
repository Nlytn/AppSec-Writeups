# [OverTheWire] - [Bandit5]

**Date:** 2025-10-13  
**Difficulty:** Easy   
**Objective:** The password for the next level is stored in the only human-readable file in the inhere directory. Tip: if your terminal is messed up, try the “reset” command.

---

## Step 1: Recon / Information Gathering
The first steps are pretty standard: login with SSH using the credentials from the previous challenge and poke around to see what we can find.

![Screenshot of challenge text](/Assets/bandit5.png)

Can I take a moment to be honest with you here? When I first saw this challenge, I had absolutely no clue how to approach this. On my first attempt, I actually tried to cat each individual file. Now this is a valid methodology with a list of only nine or so files, but what if we were given a directory of 1,000 files and told to find the *one* human readable file within? Individually opening each file could take days (likely longer), but there is a solution to find and open the file within minutes. 

My methodology can no doubt be improved upon and I would love any input that can be provided, so please don't be shy if you see any opportunites. 

---

## Step 2: Exploit / Solution
We can't really do much until we list the current directory contents, so let's start there.

![Screenshot of list directory contents](/Assets/bandit5_ls.png)

It looks like this is our list of files, but there isn't really a lot of information on these. Even running an extended ls command doesn't return any meaningful information. Take another look at the challenge and you'll notice a small nudge: there are a selection of commands we are recommended to use:

    ls
    cd
    cat
    file
    du
    find

For anyone who's spent any amount of time in Linux, most of these commands will be pretty familiar. For all others, I'm going to give a brief overview of each command below:

    ls -> list directory contents
    cd -> change directory
    cat -> read file 
    file -> show information on file(s); i.e. type
    du -> disk usage; how much space is the file taking on the hard drive
    find -> find a file by name or pattern

This answer took an embarrasingly long amount of time to find, but once discovered it seemed so obvious. At a glance, most of these commands are either not applicable or have a very niche use case in this challenge. We can list the files in several ways, but that won't show us the human-readable content. 

Instead of explaining how each command won't work, let's jump to the one that will. 

Did you notice that little note beside the file command? That little guy is our little hero in this challenge. In brief, here's how the file command works. When running *file*against a file, the command inspects the content, not just the name of the file. This starts with checking filesystem metadata, then comparing the file's initial bytes (magic numbers) againsta a database of known signatures to identify known formats. If no signature is found, it analyzes byte ranges to determine whether the contents are text (human-readable) or binary data.

Is the picture starting to come together more clearly now? No matter if we have nine or 9,000 files, we can run this command to find the human-readable file(s). The only limitation is how long it takes the OS to check the bulk of the files; more files means more time. Luckily for us, we only have seven files so it should be rather quick.

Yet, I still don't really want to run *file* on each and every file in this directory. Fortunately, bash let's us iterate just about anything. So let's try it out:

    file inhere/*

I know what you're thinking. "Nlytn, what are you doing? What's up with the /* after the directory? Did you typo or something?" See, this is why we work so well together: you're asking the right questions. I knew I liked you.

The *file* command requires an argument to run against. We have to specify a file (or files) to run against, otherwise it would theoretically run against the entire filesystem. That would be problematic to say the least. Realistically, it would just fail, which is also problematic. By addending /* to the end, we are saying to run this file against *every file in the specified directory*. Let's see how it works:

![Screenshot of challenge text](/Assets/bandit5_file.png)

What's that? Is there an "ASCII text" file in the list? Well, you know what we have to do now. Let's crack it open and hope that we find....

![Screenshot of challenge text](/Assets/bandit5_cat.png)

YES!! We got it! I feel like the difficulty ramp is slowly becoming a wall. Well, I'm glad I'm good at climbing.  

---

## Step 3: Lessons Learned
- Usefulness and application of file command  
- How to find and identify human readable files  

---

## References
- [Bandit5](https://overthewire.org/wargames/bandit/bandit5.html)  
- [Reference on file command](https://man7.org/linux/man-pages/man1/file.1.html?utm_source=chatgpt.com)