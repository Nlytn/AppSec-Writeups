# :trident: Clearing the Fog to Find the Line of Truth: Bandit10 and the Beauty of Strings"

## :aerial_tramway: TL:DR

For Bandit 10's challenge, we have to search an encoded file for one line of text. By leveraging `strings` with `grep` we can establish a quick, simple and repeatable process.

## :dart: Objective

You have just been informed there has been a potential breach. During the investigation you discover an unusual and unrecognized file format, and you are unable to find the proper program to open the file. In addition, you aren't quite sure how the file will act if it is opened. After some research you discover there is a possibility of human readable text in the file; all you have to do is open the file in a text editor of choice. So you `cat` the file only to see what appears to be a jumbled mess of symbols. Any seemingly human readable content is either truncated or mixed with these symbols, so reading it manually could take hours, days, or longer. So how do you find the text in the file within a reasonable amount of time?

---

## :bulb: Key Concepts / What You’ll Learn    
- How to programatically search files for text using `strings`  

---

## :gear: Walkthrough  

### :one: Exploring the Environment  
As usual, let's run a list command and see what we are working with here  

```bash
ls -la
```

![ls -la showing no obvious files](/Assets/bandit10_ls.png)
|:--:|
| * Listing of home directory contents * |

I'm getting a bit of déjà vu, but I do appreciate some familiarity. Since it's the only file in the directory, that also further simplifies our search. Let's dig in and see what this one holds for us.   


## :two: Printing Contents of File

No point in dragging our feet - let's cat the file and see what it holds. 

Use 
```bash
cat data.txt
```

| ![cat output of non-text file](/Assets/bandit10_cat.png) |
|:--:|
| * cat output of non-text file * |


Oh, what the flying pigs?! (If I'm being honest, I did not verbally say pigs but trust me when I say that's the more acceptable version). Running a `file` command reveals this isn't really a text file after all.

```bash
file data.txt
```

| ![output of file command on data.txt](/Assets/bandit10_file.png) |
|:--:|
| * output of file command on data.txt * |


## :three: Work Smart - Searching the file

This is one of those challenges that can be *really* simple or *really* difficult. If you are weak in Linux, then this is a classic learning experience. We are given some pretty powerful hints in one line:

* Password is in file **data.txt**

* It is one of the few **human-readable strings**

* It is **preceded by several = (equals) characters**

This is all provided in the final sentence of the hint, and combined they paint a pretty clear picture of how to move forward. It's time to learn a new trick.

Linux has a tool, aptly named `strings`. This tool will (as you may expect) pull text strings from a file (as with many Linux tools, `strings` is not limited to reading from a file). 

```bash
strings data.txt
```

| ![output of strings](/Assets/bandit10_strings.png) |
|:--:|
| * Output of `strings` command * |

So we could start to sift through this data, and we would definitely find the password within a few minutes. But what if we could just print that line right out to us? Luckily for you, we can!

What if we combined this with the power of `grep`. Stay with me now. We know the line is preceded by a series of = characters, right? So what if we pass the output of strings *directly to* `grep`? 

```bash
strings data.txt | grep ===
```

| ![output of strings piped through grep](/Assets/bandit10_strings_grep.png) |
|:--:|
| * Output of `strings` command piped through `grep` * |

Okay, that's honestly pretty cool. As you can see in the screenshot, you can use a single `=`, but using three gives us this neat little message. I hope it makes sense how we've gotten this far, but as always I'll put a breakdown of the commands right below: 

* strings - pull text strings from file. 

* data.txt - text file to be searched

* | - pipe operand; send output from first command to second command for further processing

* `grep ===` - search the specified file for any line containing ===

## :brain: Understanding the Technique

By leveraging `strings`, we were able to find a human readable text within a wall of jumbled symbols. By sending this output to `grep`, we were able to further filter the output based on the clues given. A task that appeared seemingly impossible was able to be completed by combining two built-in Linux tools.  

[Documentation for strings](https://linux.die.net/man/1/strings)

[Documentation for grep](https://man7.org/linux/man-pages/man1/grep.1.html)


## :lock: Security Takeaway

When performing forensics, being able to quickly extract and interpret human-readable text from binary or encoded data can reveal hidden credentials, attacker notes, or other indicators of compromise that would otherwise go unnoticed. 

## :toolbox: Commands Summary

```bash
ls -la
cat data.txt
file data.txt
strings data.txt
strings data.txt | grep ===
```

## :writing_hand: Final Thoughts

These challenges are getting trickier but also more fun. So far, we've experienced several tricks that will save our time and energy. While our concentration has been on CTF style challenges, we are also learning how to be more efficient in a Linux environment. 


## :sun_with_face: Time to Shine

Searching is a powerful feature that can be implemented in a number of ways depending on the scenario. Have you faced a situation where you had to find something (a file, text within a file, etc) in a sea of noise while quickly running out of time? How did you maximize your efficiency to get the job done? I would love to hear your war stories and successes! 

## :recycle: Notes
Completed: 10.24.2025   
Challenge: https://overthewire.org/wargames/bandit/bandit10.html