# :trident: Bandit6 – “Finding the Rare Cut Gem in a Pile of Rubble”

## :dart: Objective  
Find the only **human-readable file** of a specified size that is *not executable* hidden within a massive directory full of random data.

---

## :bulb: Key Concepts / What You’ll Learn  
- How to identify file types using the `file` command  
- How to locate files of a specified size using the `find` command  
- How to safely verify if a file is executable or not
- Understanding what “human-readable” really means in Linux  
- Why proper analysis beats guessing when working with unknown data  

---

## :gear: Walkthrough  

### :one: Exploring the Environment  
When connecting as the Bandit5 user, you’ll land in a directory loaded with, well, other directories.  

```bash
ls -la
```

![Screenshot of list directory contents](/Assets/bandit6_ls.png)

Much like the last challenge, there are just too many files to check manually. Reviewing even one directory makes this evident; we're going to need to leverage some Linux automation again.

## :two: Filtering Candidates

We know the goal: find the human-readable file. Though we have more files to check, we also have more context: the file also must be 1033 bytes in size, and it must not be executable. It looks like we get to use a new command (and one of my personal favorites): `find`. 

Due to the intrinsic nature of `find`, the command will automatically search a full directory. Therefore part of the command is specifying the directory to run in; it won't automatically run in the current directory. I've failed this command enough times to remember this :

```bash
find ./ -type f -size 1033c
```

![Screenshot of find command](/Assets/bandit6_find.png)


Explanation:

* find ./ — search in current directory

* -type f — locate all regular files 

* -size 1033c — file must be 1033 bytes in size


## :three: Identifying the Target

This filters the thousands of random files down to one candidate — the only file that meets all three criteria.
A quick cat confirms it contains the next password.

```bash
cat ./maybehere07/.file2
```
![Screenshot of find command](/Assets/bandit6_cat.png)

## :brain: Understanding the Technique

The find command searches the filesystem (or specified directory) based on given criteria. This command is immensely helpful to locate a particular file within a filesystem containing thousands or more files.

## :lock: Security Takeaway

This challenge reinforces that metadata isn’t always trustworthy.
Just because something looks like a text file (or an image, or a script) doesn’t mean it is.

File inspection tools like file or mimetype are vital for safe file-handling logic — especially in upload or ingestion pipelines.

## :toolbox: Commands Summary

```bash
ls -la
find ./ -type f -size 1033c
cat ./maybehere07/.file2
```

## :writing_hand: Final Thoughts

This level reminded me that sometimes brute-force searching is about using the right filters, not raw persistence.
`find` is an immensely useful tool that when used correctly can greatly cut down your time and effort to find the gem in the rough.

:thought_balloon: Have you ever found yourself in a position where you spent too long looking for something only to find a simpler solution after the fact? We've all been there. Share an instance you've experienced — let's embrace the catharsis together.

## :recycle: Notes
Completed: 10.14.2025
Challenge: https://overthewire.org/wargames/bandit/bandit6.html