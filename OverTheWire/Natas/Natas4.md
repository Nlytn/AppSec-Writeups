# [OverTheWire] - [Natas4]

**Date:** 2025  
**Difficulty:** Medium  
**Objective:** Find the password for the next level.

---

## Step 1: Recon / Information Gathering
As with most (if not all) of these challenges, we are presented with a cryptic clue. This one is a bit better than others, but it admittedly took me a moment to understand what was being expected.

![Screenshot of challenge text](/Assets/natas4.png)

---

## Step 2: Exploit / Solution

Checking the source code, there isn't much of interest. We were lucky for a little while, but now its time to get creative. 

So the site tells us that we are not referred by the correct site. The strange part is that it tells us we must be referred by the next challenge. But to get to the next challenge, we have to get past this one. 

Well, I get to start using one of my personal favorite tools: BurpSuite. This tool is aptly named as it comes with a literal suite of products. My main focus is typically the proxy functionality, though that's not to say I don't leverage other parts (Intruder, Sequencer) where necessary.

For this challenge, let's just intercept a request and see what we can find. I'll assume you know how to route traffic from your browser to Burp Proxy, but if not there is a lot of documentation online for setting this up. 

![Screenshot of request](/Assets/natas4_request.png)

So even here there isn't much to be seen. Maybe that is itself a clue; there isn't a referer header either. I feel like this is a long shot, but that it's also what we are being told to do. So what happens if I insert a referer header here:

![Screenshot of challenge text](/Assets/natas4_referer.png)

Now let's send the request. 

![Screenshot of challenge text](/Assets/natas4_reveal.png)

Wow, that actually worked! That was a fun challenge; I'm curious to see how the next ones ramp up.

---

## Step 3: Lessons Learned
- Never trust client-controlled data, such as HTTP headers, for authentication or access control decisions, as they can be easily manipulated.   
- Request interception/manipulation  


---

## References
- [Link to the challenge](https://overthewire.org/wargames/natas/natas4.html)  
- [BurpSuite Documentation](https://portswigger.net/burp/documentation/contents)