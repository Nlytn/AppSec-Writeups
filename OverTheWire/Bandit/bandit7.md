# :trident: Bandit7 – “Broken Paths, Open Files: The Trick Behind Bandit7”

## :dart: Objective  
Find the only file of a specified size that is owned by *user* bandit7 **and** owned by *group* bandit6. The file can be located anywhere within the filesystem. 

---

## :bulb: Key Concepts / What You’ll Learn  
- How to identify file types using the `file` command  
- How to locate files of a specified size using the `find` command  
- How to find files owned by a specific user and/or in Linux
- Understand how and why to filter standard errors (stderr)   

---

## :gear: Walkthrough  

### :one: Exploring the Environment  
When connecting as the Bandit6 user, there isn't much to see in our home directory.  

```bash
ls -la
```

![Screenshot of list directory contents](/Assets/bandit7_ls.png)

The challenge text says the file is stored **somewhere on the server**, so I'm willing to bet its not conveniently in the home directory this time. Let's crack those knuckles and start perusing the filesystem at large.

## :two: Filtering Candidates

Fortunately, we know what we are looking for: the file is owned by user bandit7 and group bandit6, and it is only 33 bytes in size. So this is only a slight variation on what we performed in previous challenges, and we get to use some more fun and interesting Linux commands. 

We are going to rely on our old pal `find` again, but with some slight modifications this time. Since we know both the group and owner of the file, we can throw that into our `find` command and hopefully pull back what we need. Let's try it out:

```bash
find / -type f -size 33c -user bandit7 -group bandit6
```

![Screenshot of find command](/Assets/bandit7_find1.png)


Explanation:

* find / — search in root directory

* -type f — locate all regular files 

* -size 33c — file must be 33 bytes in size

* -group bandit6 — file is owned by user bandit6

* -group bandit7 — file is owned by group bandit7

As you can see, there is a sea of errors. There is a chance the file we need is somewhere in this list, but after searching several times I'm just not seeing it. If only there was a way to purge the errors and only see the successes...

## :three: Identifying the Target

Well, that's exactly what we are going to do. Sometimes there is just too much information to be helpful, so let's see if we can condense this to something easier to digest. We are going to tell Linux to route any standard errors to a null folder. Basically, we are saying we don't want to see these messages but that instead these errors should be routed to a "null device", or black hole. In other words, just get rid of those messages since they aren't helpful for what we need right now.

```bash
find / -type f -size 33c -user bandit7 -group bandit6 2>/dev/null

```

* 2 — Specifies standard errors (stderr)

* > — operand to route specified output

* /dev/null — null device to send output to

![Screenshot of find command](/Assets/bandit7_cat.png)

## :brain: Understanding the Technique

The find command searches the filesystem (or specified directory) based on given criteria. The command can be customized to search for many paramaters, including users, groups, filesizes, formats, etc.

## :lock: Security Takeaway

This challenge reinforces the importance of proper file permissions.
While this file was owned by a different user *and* group, improper file permissions allowed execution by an unauthorized user.


## :toolbox: Commands Summary

```bash
ls -la
find / -type f -size 33c -user bandit7 -group bandit6
find / -type f -size 33c -user bandit7 -group bandit6 2>/dev/null
cat /var/lib/dpkg/info/bandit7.password
```

## :writing_hand: Final Thoughts

This is another level where brute-force searching is about using the right filters, not raw persistence.
`find` is very strong when used properly, and can be combined with other commands to increase its power. 

:thought_balloon: When have you been in a position where you needed to find that singular file within millions on a timeline? How and why was it so critical? Please share some experiences with me so I don't feel so alone. 

## :recycle: Notes
Completed: 10.20.2025   
Challenge: https://overthewire.org/wargames/bandit/bandit7.html