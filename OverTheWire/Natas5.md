# [OverTheWire] - [Natas5]

**Date:** YYYY-MM-DD  
**Difficulty:** Easy  
**Objective:** Find the password for the next level.

---

## Step 1: Recon / Information Gathering
Upon logging in, we are met with this very gentle and helpful message:

![Screenshot of challenge text](/Assets/Natas5.png)

I'm going to be honest; I don't believe you. There is a way for me to login. There must be, otherwise this challenge is kind of pointless, right?

---

## Step 2: Exploit / Solution

Maybe there is a hint of some kind in the source code. To be fair, I'm noticing a pattern over the past few challenges. It feels rather foolish to not at least inspect the source for potential hints.

![Screenshot of source code](/Assets/Natas5_source.png)

Well, I must admit I'm a little disappointed. I was so sure there was going to be something here. This one is tricky. The URL doesn't betray any hints, nor does the source code. 

Here's something curious that just occurred to me. It explicitly says I cannot log in

![Screenshot of challenge error](/Assets/Natas5_error.png)

See? It really does say that; I didn't make it up. But what is it talking about? There isn't even a place for me to attempt to log in? I would normally lean towards SQLi, but where? 

This took a little time to think on, but when the answer hit me I almost fell over. You know what we haven't tried yet? I wonder what the intercepted request will show us...

![Screenshot of burp request](/Assets/Natas5_burp.png)

I refreshed the page and sent the intercepted request to Repeater. Now that you're caught up with me, let's look over this request for a moment. Do you notice anything interesting? Take your time; it took me a moment to catch it.

![Screenshot of burp request](/Assets/Natas5_not_logged_in.png)

Did you notice this little guy? One little parameter that may change everything. Let's change this to a 1 and send the request; maybe this will work?

![Screenshot of burp request](/Assets/Natas5_logged_in.png)

Well would ya look at that? It actually worked! So by changing one paramater mid-flight, I was able to get access to the next challenge.

---

## Step 3: Lessons Learned
- Authentication and authorization should never rely on client-controlled values (like cookies) without server-side validation, since they can be modified at will.  

---

## References
- [Link to the challenge](http://natas5.natas.labs.overthewire.org/)  
- [Burp Repeater Documenation](https://portswigger.net/burp/documentation/desktop/tools/repeater)