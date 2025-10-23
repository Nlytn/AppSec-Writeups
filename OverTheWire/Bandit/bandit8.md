# :trident: “Grep-ing for a Clue: Sorting Through Bandit8”

## :aerial_tramway: TL:DR

Leverage `grep` to search files containing millions of words for specific word match and print entire line. 

## :dart: Objective

Let's set the scene: you have a file that contains a password. Perhaps this is your main password storage solution. Within this file, you need one and only one entry so you can login but you don't want to have to peruse millions of lines to find the right one. Not only would this take forever, but you could overlook it. This is also assuming proper formatting; what if the format was corrupted but the data is intact? What do you do?

---

## :bulb: Key Concepts / What You’ll Learn    
- How to search files using `grep`  
- Return entire line containing match found with `grep`

---

## :gear: Walkthrough  

### :one: Exploring the Environment  
When connecting as the Bandit7 user, there is the one file that was specified in the challenge.  

```bash
ls -la
```

![ls -la showing no obvious files](/Assets/bandit8_ls.png)
|:--:|
| * Listing of home directory contents * |

Okay, so far so good. It looks like we aren't being led astray quite yet. In fact, this actually looks a bit more straightforward than some of our previous challenges. 


## :two: Printing Contents of File

So we know what file we need, and we know where it is. Now we just need to get the password from this file. My first thought is to just `cat` the file and read the contents. Surely it can't be that much, right? Let's try it out:

Use 
```bash
cat data.txt
```

| ![find output before filtering stderr](/Assets/bandit8_cat.png) |
|:--:|
| * Output of ridiculous cat output * |


Oh. Oh wow. I didn't expect that. This isn't going to work. I mean, one could *try* to manually parse this and I would give my blessing. Yet I prefer to work smart, not hard.

## :three: Work Smart - Searching the file

As I mentioned, you could manually parse this file. It would take hours, if not longer, and you would undoubtedly be exhausted by the end of it. This also assumes time is not a commodity; but when time becomes an important factor this becomes cumbersome quickly.

So what do we do? Well, we're going to search the file for the given match (millionth). We've been told the password is right next to this word, and is on the same line. We are going to leverage a new tool: `grep`. Using `grep`, you are able to search a file based on position, word match, pattern match, and other options. By default, grep will print the entire line as long as the match based on one of the following criteria:

* Match is at beginning of line 

* Match is not preceded by a number/letter/underscore

* Match is at the end of the line

* Match is not followed by a number/letter/underscore

Let's put this in layman's terms: If the match is the first or last word in the line, the line will be returned. Likewise, if the word is in the middle of the line but is alone (i.e. not part of another word/string), the line will be returned. However, if the word is part of a string, `grep` will overlook it as non-matching (i.e. `grep chick` will not return a line showing `chicken` since `chick` doesn't match `chicken`).

Now that we have that cleared up, let's see what happens if we `grep millionth`:
```bash
grep millionth data.txt

```

* `grep` — search the specified file for the specified match

* millionth — word to match in our search

* data.txt — specified file to search

| ![output of grep](/Assets/bandit8_grep.png) |
|:--:|
| * Output of `grep` command * |


## :brain: Understanding the Technique

While it was covered above, let's briefly review this once more. Using `grep`, you can search a given file for a match. This tool is significantly more robust than the use we have shown here and can be used with many types of files/outputs. [Documentation for grep](https://linux.die.net/man/1/grep)

## :lock: Security Takeaway

This challenge reinforces the importance of proper file permissions. While this file was owned by a different user *and* group, improper file permissions allowed read access by an unauthorized user.


## :toolbox: Commands Summary

```bash
ls -la
cat data.txt
grep millionth data.txt
```

## :writing_hand: Final Thoughts

I rather enjoyed this challenge, and I appreciate the progression of this series. While we have been focusing on searching for the right *file*, this time we had to search a singular file for the right *line within that file*. A nice pivot that taught me a new technique I'm sure to refer to in later challenges (and life). 

One key takeaway here is that security through obscurity almost never works. Assuming something won't be found really means it only takes a bit longer to find. This file should have had proper permissions applied, in an altogether different directory inaccessible by any users except the owner. Ideally, passwords should be kept in a more secure fasion (i.e. password manager) and not a simple text file in the home directory.

## :sun_with_face: Time to Shine

Have you run into a situation where you needed to find a hay in a needle stack? What was it, and how did you find it? Let us share in your pain and victory! Share your story in the comments. 

## :recycle: Notes
Completed: 10.20.2025   
Challenge: https://overthewire.org/wargames/bandit/bandit7.html