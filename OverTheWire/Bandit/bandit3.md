# [OverTheWire] - [Bandit3]

**Date:** 2025-10-09  
**Difficulty:** Easy   
**Objective:** The password for the next level is stored in a file called --spaces in this filename-- located in the home directory

---

## Step 1: Recon / Information Gathering
This challenge is similar to the one before. We have a specified file, listed in our home directory, and we are charged with reading that file for the password to the next challenge. Simple enough, right?

![Screenshot of challenge text](/Assets/Bandit3.png)

---

## Step 2: Exploit / Solution
So far it's pretty standard. Let's just run a ls command to see what's actually here:

![Screenshot of challenge text](/Assets/bandit3_ls.png)

So let's try to cat the file like we did before: 

![Screenshot of first cat attempt](/Assets/bandit3_cat1.png)

So as you can see from the above screenshot, this took me a moment to figure out. Apparently this is not the same as the last challenge. I thought since the last challenge consisted of opening a file labeled with just a hyphen, that this challenge was naturally a progression of this logic. Well, I was close but that's not quite the case.

After a *lot* of searching, I finally found the answer. I was starting to feel hopeless, but that's when I stumbled upon this little nugget of hope. 

Let's take a look at the error again. We're not always this lucky, but the error is telling us the issue. When we try to run the **cat** command followed by --spaces in this filename--, cat is interpreting --spaces as an argument. When encapsulating in single or double quotes, this just passes the entire line as an argument (i.e. --spaces in this filename--). 

To go a step further in explaining, cat is like many commands in Linux in that it has various switches to customize the data grabbed and output, as well as how that data is shown. Using an example file test.txt, we can run cat -n test.txt, which will read the file and output this on our screen with numbered lines (thanks to the -n operand). If you read the documentation for cat, you'll notice there is no operand for --spaces in this filename. I suppose that's a good thing since we need to open a file, but not necessarily in a specific way.

So where does this leave us? How does one recoup a file that was given an ill named convention by a junior admin? 

We're going to throw on some more hyphens. Two, to be exact. When we put them in the right place, we are telling cat to process the rest as raw input, not as an operand. So, let's try it out (and remember not to typo like I did at first):

![Screenshot of successful cat attempt](/Assets/bandit3_cat2.png)

---

## Step 3: Lessons Learned
- New method to escape perceived operand   

---

## References
- [Link to the challenge](https://overthewire.org/wargames/bandit/bandit3.html)  
- [Reference to GNU documentation - refer to Section 9.1.3](https://www.gnu.org/software/coreutils/manual/coreutils.html#cat-invocation)