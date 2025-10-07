# OverTheWire - Natas0

**Date:** 2025  
**Difficulty:** Easy  
**Objective:** Find the password for the next level.

---

## Step 1: Recon / Information Gathering
We have been provided the initial username and password for the first challenge: natas0/natas0

We are presented with a message upon first logging onto the site:

![Screenshot of challenge text](/Assets/Natas0.png)

---

## Step 2: Exploit / Solution

There isn't anywhere obvious to interact on this screen. This is also an introductory challenge, so I imagine the answer is somewhere *close*.

What happens if we view the source code? Any hints or tips of where to look next?

![Screenshot of source code](/Assets/Natas0_source.png)

Well, that was a bit easier than expected. With the password in hand, onto the next level!

---

## Step 3: Lessons Learned
- Secrets should never be stored in client-side code or web pages, because anything sent to the browser can be inspected by the user.  


---

## References
- [Link to the challenge](https://overthewire.org/wargames/natas/natas0.html)
