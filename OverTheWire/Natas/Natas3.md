# [OverTheWire] - [Natas3]

**Date:** 2025  
**Difficulty:** Easy  
**Objective:** Find the password for the next level.

---

## Step 1: Recon / Information Gathering

This looks familiar

![Screenshot of challenge text](/Assets/natas3.png)

---

## Step 2: Exploit / Solution
Did they just copy the source code from the last challenge? Actually, that's a good thought. What does the source code show here?

![Screenshot of source code](/Assets/natas3_source.png)

Well, this is interesting. So there doesn't appear to be a direct object reference for an image or the like, but there is a comment:

    <!-- No more information leaks!! Not even Google will find it this time... -->

To provide some background, every site should contain a file titled "robots.txt". When a search engine (in this case, Google) indexes a site through its iterative processes, this is the first file that should be reviewed. This tells the search engine which pages on the site should not be listed as a first-level security measure. 

With that in mind, let's take a look at the robots.txt file to see if there is any useful information:

![Screenshot of robots.txt file](/Assets/natas3_robots.png)

As I mentioned above, robots.txt tells the search engine which pages to ignore indexing. This is typically because, well, the page needs to be hidden for some reason. It could be an admin or other login portal, and with the right security measures in place we still won't be able to get through. But let's take a look anyway:

![Screenshot of robots.txt file](/Assets/natas3_secret.png)

It looks like there is another users file. Open that file to get the password for the next level.

---

## Step 3: Lessons Learned
- Client-facing configuration files (like robots.txt) are not secure storage — they’re hints for crawlers, not an access-control mechanism.


---

## References
- [Link to the challenge](https://overthewire.org/wargames/natas/natas3.html) (if public)  
- [Robots.txt Reference](https://developers.google.com/search/docs/crawling-indexing/robots/intro)