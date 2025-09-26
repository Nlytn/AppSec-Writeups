# [OverTheWire] - [Natas2]

**Date:** 2025  
**Difficulty:** Easy  
**Objective:** Find the password for the next level.

---

## Step 1: Recon / Information Gathering

Another challenge, another message. Apparently there is nothing on this page. Don't take my word for it:

![Screenshot of challenge text](/Assets/Natas2.png)

---

## Step 2: Exploit / Solution

For better or worse, I'm stubborn. I just have to know for myself. Call it a curse, but sometimes it's a virtue. 

In this case, its definitely the latter. Let's check out the source code (noticing a trend yet?) and see if there is anything juicy.

![Screenshot of source code](/Assets/Natas2_source.png)

At first, there doesn't appear to be much. We aren't getting lucky enough to have the password shown in the source, unfortunately. 

Wait...there is something interesting here. I don't see the evidence on the page, but the source code shows an image file is being linked from another directory.

![Screenshot of directory path](/Assets/Natas2_pixel.png)

Let's go to that page and see if it's even accessible. 

![Screenshot of directory path](/Assets/Natas2_pix_page.png)

Maybe that's why we couldn't see it: it's a literal pixel. As fun as this is, it seems we are at another dead end. The source code shows nothing of interest.

So what now? 

I wonder...can we just go back a directory? I input the whole path as a test to see if it would work, which it did. So can we access the parent directory that houses this image?

![Screenshot of directory path](/Assets/Natas2_files.png)

I feel like that's a pretty confident success. Obviously my eyes immediately go to that users.txt file; there has to be something good for us in there:

![Screenshot of directory path](/Assets/Natas2_users.png)

That was a windy, twisty path. But it was a fun one. Now we have our next password. Onward and upward!

---

## Step 3: Lessons Learned
- Source code is complex. So is access control. There is no reason I should have been able to access that file, much less the directory.
- When we publish apps to the public, we have to ensure all necessary restrictions are in place. Access control can prohibit a normal user from performing or even viewing administrative tasks.  

---

## References
- [Link to the challenge](https://overthewire.org/wargames/natas/natas0.html)  
