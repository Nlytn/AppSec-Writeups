# [OverTheWire] - [Natas6]

**Date:** 2025  
**Difficulty:** Medium  
**Objective:** Find the password for the next level.

---

## Step 1: Recon / Information Gathering

Another challenge, another page with a cryptic hint. 

![Screenshot of challenge text](/Assets/natas6.png)

---

## Step 2: Exploit / Solution

Oh, I like this one. There is a nice input field and everything. The URL, like most others, betrays nothing. So I'm going to hit this box and see if I can manipulate my way through. I'm guessing some kind of SQLi will work here. Let's try it out and see what happens:

![Screenshot of SQLi attempt](/Assets/natas6_sqli_one.png)

To explain the methodology here, we are trying to log into the app using only the admin username. This assumes: 1) there is a username titled "admin" (while we cannot be sure, this is a safe bet to start with), and 2) There is no input sanitization, which allows us to skip the password altogether. 

So when we input 'admin--, the server processes the query as SELECT * FROM members WHERE username = 'admin'-- AND password = 'password'. Notice the -- before "AND password..." The "--" is a comment to remove the rest of the query; the quote at the end of admin closes the query. So in reality, what we are telling the server is "SELECT * FROM members WHERE username = admin". I'm sure you can determine why it would really not be a good idea to let just anyone log in with an admin account. But alas, this is not the real world (thankfully), so there is a chance it will work here. (Don't tell anyone, but sometimes there is a chance this works in the real world too. Shhhhh!)

So let's try this here; it's worth a shot, right? 

![Screenshot of invalid message](/Assets/natas6_invalid.png)

Dammit. I suppose that would have been too easy. Maybe there's something in the request? I suppose I could try to intercept and see what we find.

![Screenshot of intercepted request](/Assets/natas6_burp.png)

I'm going to be honest. If there is something here, I'm not seeing it. It was at this point I started to get a little concerned. Until I realized something seemingly simple: I haven't checked the source code yet. It's a long shot, but its worth a shot nonetheless. 

Initially, I right-clicked to view source code (you know, the normal way aside from F12), but I didn't really see anything of value. Then I noticed the link right under the input box -> View sourcecode. Let's take a look here:

![Screenshot of intercepted request](/Assets/natas6_source.png)

Is that php code? I'm going to try to explain the flow of this function and its relevancy. We can see it's including a file, "includes/secret.inc". The function checks if there is a value that has been submitted. Then it checks to see if the submitted value is equal to ['secret'], whatever that value is. If the values match, then the password for Natas7 is shown; otherwise we get the error "Wrong secret". 

At this point, I tried a few different tactics. I tried inputting secret, 'secret' and some other variations. I tried encoding. I tried everything I could think of. Until I took another look and noticed the very first line -> include "includes/secret.inc". I'm going to be straight, I don't really know PHP. I'm not a web designer by trade or hobby, but I am a security researcher by both. I say this to say it took me a little longer than I want to admit to notice this caveat. What, exactly, is in secret.inc? Can we even get to it?

![Screenshot of secret page](/Assets/natas6_secret.png)

Apparently so, but I'm still not sure how this helps at all. It's a blank page, right? I intercepted the request in Burp and viewed there, but I didn't get anything at all of interest. Simply for kicks, let's try to view the source code here:

![Screenshot of secret page](/Assets/natas6_secret2.png)

Okay, what? Is that a secret...in the source code?! Seriously?! Well okay then, let's grab this bad boy and see what we can do here.

You know what? I may be tired and therefore lazy right now, but Occam's Razor has some merit (especially in this field). We might need to look up some obfuscation techniques for this, or possibly research decryption methods, but before we do that let's just try for the simple approach.

![Screenshot of secret page](/Assets/natas6_input_secret.png)

![Screenshot of secret page](/Assets/natas6_solved.png)

---

## Step 3: Lessons Learned
- Never expose secrets or sensitive logic in publicly accessible files; authentication and access control must be enforced server-side, not hidden in linked resources.  
- Usefuleness of path traversal and when/where/how to execute  
- Always view source code and any relevant information provided

---

## References
- [Link to the challenge](https://overthewire.org/wargames/natas/natas6.html)   
- [SQLi Cheat Sheet](https://www.invicti.com/blog/web-security/sql-injection-cheat-sheet/)
- [Occam's Razor](https://en.wikipedia.org/wiki/Occam%27s_razor)