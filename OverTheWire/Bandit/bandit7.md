# :trident: “Broken Paths, Open Files: The Trick Behind Bandit7”

## :aerial_tramway: TL:DR

Use `find` filters plus stderr redirection (2>/dev/null) to reveal matches buried under permission errors; verify ownership with `ls`/`file`. 

## :dart: Objective

What if the password you need is somewhere in the file system, but hidden beneath thousands of ‘Permission denied’ messages? Such is our challenge today. We are charged with finding a file that meets this criteria:

* Owned by user bandit7
* Owned by group bandit6
* Size of 33 bytes

Though this is quite similar to the last challenge, we have a few new qualifiers. So you know what that means: time to learn new commands and switches!  

---

## :bulb: Key Concepts / What You’ll Learn    
- How to locate files of a specified size using the `find` command  
- How to find files owned by a specific user and/or group in Linux
- Understand how and why to filter standard errors (stderr)   

---

## :gear: Walkthrough  

### :one: Exploring the Environment  
When connecting as the Bandit6 user, there isn't much to see in our home directory.  

```bash
ls -la
```

![ls -la showing no obvious files](/Assets/bandit7_ls.png)
|:--:|
| * Listing of home directory contents * |

The challenge text says the file is stored **somewhere on the server**, so I'm willing to bet its not conveniently in the home directory this time. Let's crack those knuckles and start perusing the filesystem at large.

## :two: Filtering Candidates

Fortunately, we know what we are looking for: the file is owned by user bandit7 and group bandit6, and it is only 33 bytes in size. So this is only a slight variation on what we performed in previous challenges, and we get to use some more fun and interesting Linux commands. 

We are going to rely on our old pal `find` again, but with some slight modifications this time. Since we know both the group and owner of the file, we can throw that into our `find` command and hopefully pull back what we need. Let's try it out:

Use 
```bash
find / -type f -size 33c -user bandit7 -group bandit6
```

| ![find output before filtering stderr](/Assets/bandit7_find1.png) |
|:--:|
| * Output of nasty, messy `find` command * |


Explanation:

* find / — search in root directory

* -type f — locate all regular files 

* -size 33c — file must be 33 bytes in size

* -group bandit6 — file is owned by group bandit6

* -user bandit7 — file is owned by user bandit7

As you can see, there is a sea of errors. There is a chance the file we need is somewhere in this list, but after searching several times I'm just not seeing it. If only there was a way to purge the errors and only see the successes...

## :three: Identifying the Target

Well, that's exactly what we are going to do. Sometimes there is just too much information to be helpful, so let's see if we can condense this to something easier to digest. We are going to redirect stderr to `/dev/null` to hide permission errors. Basically, we are saying we don't want to see these messages but that instead these errors should be routed to a "null device", or black hole. In other words, just get rid of those messages since they aren't helpful for what we need right now.

```bash
find / -type f -size 33c -user bandit7 -group bandit6 2>/dev/null

```

* 2 — Specifies standard errors (stderr)

* `>` — operand to route specified output

* /dev/null — null device to send output to

| ![find output before filtering stderr](/Assets/bandit7_find2.png) |
|:--:|
| * Output of clean, filtered `find` command * |

## :fire_engine: Verify the file

It's always a good idea to confirm we know what we are seeing. This is admittedly something I did not do in this challenge, but this will become part of my evolving procedure going forward.

```bash
ls -l path/to/file 		# confirm ownership and permissions
file path/to/file 		# confirm human-readable/text
wc -c path/to/file 		# confirm size in bytes
```

![Screenshot of find command](/Assets/bandit7_cat.png)
|:--:|
| * Opening file containing password * |

## :brain: Understanding the Technique

The find command searches the filesystem (or specified directory) based on given criteria. The command can be customized to search for many paramaters, including users, groups, filesizes, formats, etc. [Documentation for find](https://man7.org/linux/man-pages/man1/find.1.html)

## :lock: Security Takeaway

This challenge reinforces the importance of proper file permissions. While this file was owned by a different user *and* group, improper file permissions allowed read access by an unauthorized user.


## :toolbox: Commands Summary

```bash
ls -la
find / -type f -size 33c -user bandit7 -group bandit6
find / -type f -size 33c -user bandit7 -group bandit6 2>/dev/null
cat ./path/to/confirmed-file # contains next-level password (redacted)
```

## :writing_hand: Final Thoughts

This is another level where efficient searching is using the right filters and not just raw persistence.
`find` is very strong when used properly, and can be combined with other commands to increase its power. 

In production systems, being able to search by metadata and understand ownership/permissions is crucial for forensic investigation and secure configurations.

## :sun_with_face: Time to Shine

What commands do you use when you’re buried under a wall of ‘Permission denied’? Drop your favorite find/filter combo in the comments. 

## :recycle: Notes
Completed: 10.20.2025   
Challenge: https://overthewire.org/wargames/bandit/bandit7.html