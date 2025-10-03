# [OverTheWire] - [Natas7]

**Date:** 2025  
**Difficulty:** Easy  
**Objective:** Find the password for the next level.

---

## Step 1: Recon / Information Gathering
Here we are with Natas7. It's starting to get juicy here; I'm a strange combination of excited and nervous for what's next. So let's take a look at the page:

![Screenshot of challenge text](/Assets/natas7.png)

Okay, there isn't much to see here but that isn't unusual. Let's try out the links.

![Screenshot of challenge text](/Assets/natas7_home.png)
![Screenshot of challenge text](/Assets/natas7_about.png)

Well, that's a little on the nose but who am I to criticize design? Let's take a look at the source code (and therein prevent repeating past mistakes). 

![Screenshot of challenge text](/Assets/natas7_source.png)

I really have to remember to check the source first. That saved me a bit of a headache from the start.

---

## Step 2: Exploit / Solution

So we have a pretty clear hint: the password is in /etc/natas_webpass/natas8. Okay, awesome. So we just need to open that file. The file that is stored server side...in a protected...directory... Crap. Okay, no worries. Let's poke around a bit more and see what we can find here. 

Take another look at the Home/About page (either will work for this example). Specifically, take a look at the URL:

![Screenshot of challenge text](/Assets/natas7_url.png)

This is interesting. I mean, really interesting. I've personally not seen this in the wild myself, but it's definitely possible. This is a classic LFI (local file inclusion) vulnerability. Here's what that means: the server is showing us a specific file that it wants us to view; in this case, its the Home or About page, respectively. In an ideal setup, these are the only pages we should be able to view. There may be other publicly accessible pages on the site that we could view by changing this param (i.e. contact-us). However, in a non-idea setup, this can be leveraged to access files *local to the server*, which lends to the name Local File Inclusion (as opposed to remote file inclusion, where we specify a file *remote* to the server, i.e. our own malicious file, which is accessed by the server). 

This specific folder, /etc/, should never be accessible through the web browser. There is legitimately no technical reason I am aware of why this would ever be the case. In fact, most (if not all) web pages are stored in /var/www/, so /etc should be irrelevant here. 

For a little more background, every web server has a low-level account assigned to it in order to access services and resources necessary for functionality. This account should never have a need to access /etc/, since all of its necessary resources are stored in /var/www. 

That being said, if this was an ideal setup then the challenge wouldn't work quite so well. Fortunately for us, this is not ideal. Well, it is for us, which should make us rather happy.

Back to the challenge. So we know where the file is located, and now we have an idea how to get to the file. Let's see if this works out for us:

![Screenshot of challenge text](/Assets/natas7_solved.png)

---

## Step 3: Lessons Learned
- Never pass user input directly into file operations; always validate against a strict allowlist. Sensitive files must be protected with proper access controls, and the web server account should have the minimum permissions required (least privilege).  
- Dangers and efficiency of LFI (local file inclusion)  
- I'll keep saying it until I remember it: always check source code first. It may be a bust, but it may have a hidden gem of info.

---

## References
- [Link to the challenge](https://overthewire.org/wargames/natas/natas7.html) (if public)  
- [OWASP LFI Reference](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion)
- [Invicti LFI Reference](https://www.invicti.com/learn/local-file-inclusion-lfi/)
