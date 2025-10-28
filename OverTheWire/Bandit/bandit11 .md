# :trident: Bandit11: All Your Base64 Are Belong To Us"

## :aerial_tramway: TL:DR

Bandit 11 presents us with a single file titled "data.txt", similar to previous challenges. We leverage the base64 -d command to decode the file’s contents directly from cat output.

## :dart: Objective

Now that we’ve sharpened our parsing skills in previous challenges, it’s time to take on a classic CTF staple — base64 encoding. While not as common today, you’ll almost certainly encounter base64 encoding at some point in your career. From my experience, many APIs still use this for authentication (though this is actively being phased out by some). However, if you continue on the CTF journey you **will** see this quite a bit. Therefore it benefits us to learn about this scheme, even at a foundational level. Don't worry; this is one of the simplest challenges we've taken on in a while. With that being said, let's get to it. I think you're going to enjoy this.

---

## :bulb: Key Concepts / What You’ll Learn    
- How to decode base64 strings using `base64`  

---

## :gear: Walkthrough  

### :one: Exploring the Environment  
It's pretty likely what we're going to find here, but in the event that we have a surprise in store let's go ahead and run a list command.  

```bash
ls -la
```

![ls -la showing no obvious files](/Assets/bandit11_ls.png)
|:--:|
| * Listing of home directory contents * |

Nope, no surprises. Just another simple old `data.txt` file.    


## :two: Printing Contents of File

We may as well see what's inside.

Use 
```bash
cat data.txt
```

| ![cat output of base64 file](/Assets/bandit11_cat.png) |
|:--:|
| * cat output of non-text file * |


## :three: Work Smart - Decoding the file

Oh, that's fun. I like this. So right from the beginning we can already tell this is a base64 encoded string. I know what you're thinking: "Nlytn, my man. Of course we know this. It says as much in the hint." Well, you're right. But there's another way. Let's take a look at this string for just a moment:

`VGhlIHBhc3N3b3JkIGlzIGR0UjE3M2ZaS2IwUlJzREZTR3NnMlJXbnBOVmozcVJyCg==`

Did you notice how it ends in `==`? As a rule of thumb, when you are running through any CTF and come across a string ending in two equals signs (==), you can assume its base64 encoded. 

So we have the file with the password, and we know the encoding scheme. From here, we have a few options. A quick search shows there are several options online to decode base64 input. Yet, instead of reading the output, copying/pasting to a browser, then copying that password to our notes, there is a simpler and more secure way to accomplish this. Linux has a built-in tool: base64. I'm going to give you a few moments to think about what this does.

Yep, it encodes and *decodes* base64 output. Check it out:

```bash
cat data.txt | base64 -d
```

| ![output of base64 command on data.txt](/Assets/bandit11_decode.png) |
|:--:|
| * output of base64 command on data.txt * |

This command is shockingly straightforward, so I'm only going to cover the newest piece below: 

* base64 - encode or decode base64. 

* -d - switch to signify decode operation 

## :brain: Understanding the Technique

This was a rather short, direct challenge but it was a fun one. When we read the output of the file, we sent this directly to `base64 -d` before `stdout`, which showed us the decoded text.   

[Documentation for base64](https://linux.die.net/man/1/base64)


## :lock: Security Takeaway

Base64 isn’t encryption—it’s simply a way to encode data for safe transmission or storage. In security work, spotting base64 often signals that sensitive data is being hidden in plain sight—something every analyst should be ready to decode quickly

## :toolbox: Commands Summary

```bash
ls -la
cat data.txt
cat data.txt | base64 -d
```

## :writing_hand: Final Thoughts

This challenge was simple and quick but essential. Having these building blocks will make further challenges simpler. These also help to further drive understanding of the Linux OS at large. 
 

## :recycle: Notes
Completed: 10.27.2025   
Challenge: https://overthewire.org/wargames/bandit/bandit11.html