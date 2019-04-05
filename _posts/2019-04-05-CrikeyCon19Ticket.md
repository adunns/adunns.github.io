---
layout: single
title: "Crikey Con 2019 Free Ticket Challenge" 
related: true
---

I was reading a Medium post about the OSCP from Luke Stephens ([his Twitter](https://twitter.com/hakluke)) a few days ago and I noticed he had another article titled "How I Hacked My Way to a Free CrikeyCon Ticket and a New Job". This looked interesting. It *was* interesting and I decided to follow Luke on Twitter. I then saw the 2019 edition of the CrikeyCon free ticket challenge and thought it would be a fun idea to give it a shot, whether or not I intended to go in the running for the free ticket itself.

One of the recruiters from Humanised Group (Australian recruiting company) posted this picture:

![Poster](/assets/images/CrickeyCon4.PNG)

So, our starting point is this string from the twitter post. 

It looks like a Base64 string. Lets try to decode it. My favorite tool for this is [CyberChef](<https://gchq.github.io/CyberChef/>).

Running it through the Base64 decoder, we see some sort of pattern here. 

![Base64 decode](/assets/images/CrickeyCon5.PNG)

In these CTF style challenges, simple substitution ciphers are popular. My first thought was to try the most obvious one, ROT13. 

![Rot13](/assets/images/CrickeyCon6.PNG)

Success! What do we have? A string that corresponds to a IP address.

Navigating to the IP address we see a message about DNS. 

![IP](/assets/images/CrickeyCon7.PNG)

Trying a DNS lookup tool (or even running something like Traceroute or going to the Shodan page for that IP) returns the real domain name. The tool I like to use is [View DNS](viewdns.info).

![DNS](/assets/images/CrickeyCon8.PNG)

What's here? A site with a form with a search box for the users and some indication about a database. The user `droppyadmin` seems like a subtle hint to an SQL Injection vulnerability ([relevant xkcd](https://xkcd.com/327/)).

**Correction:** I discovered that 'droppy' is actually the name of the con mascot (the koala in the picture at the top of this page).

![SQL Vulnerability](/assets/images/CrickeyCon1.PNG)

I fired up SQL map and ran a test on the `/?query=` field.

```shell
sqlmap -u humanisedcc2019challenge.crikeycon.com/index.php?query=
```

Whaddayaknow, it's vulnerable. `sqlmap` shows us what the payload looks like:

![SQL Vulnerability](/assets/images/CrickeyCon9.PNG)

Lets see what tables we have in the database:

```shell
sqlmap -u humanisedcc2019challenge.crikeycon.com/index.php?query= --tables
```

We see there is a `users` table. Lets dump that:

```shell
sqlmap -u humanisedcc2019challenge.crikeycon.com/index.php?query= -T users --dump
```

The result for `droppyadmin`'s `homefolder` is a subdirectory called:

```http
http://humanisedcc2019challenge.crikeycon.com/droppysmegasecretsuperhomedir2019
```

Navigating to that URL we see the next step:

![Hidden Directory](/assets/images/CrickeyCon2.PNG)There is a hidden subdirectory with the name of a month plus one of the previous years of CrikeyCon. Looking on the `crikeycon.com` website archives I notice that there was a 2017 and 2018 con. The twitter account begins in 2014 and scrolling through confirms there was also a 2014, 2015 and 2016 con. 

I tried a couple by hand to see if I could get lucky. No cigar. Time to brute force. 

I wrote a quick and dirty bash script to generate my wordlist:

![Wordlist](/assets/images/CrickeyCon3.PNG)

And off to `gobuster` (`dirbuster` works well too if you want a GUI):

```shell
gobuster -u http://humanisedcc2019challenge.crikeycon.com/droppysmegasecretsuperhomedir2019/ -w list.txt
```

We find our hidden directory is called `November2016`. In here we find an image with the name and logo of the company that made this challenge along with the following text:

> Good job, you made it this far...in the image below is the email address and subject you need to use to email to be in the running for a free CrikeyCon2019 ticket. Trust me, the info is in there....somewhere :)

After downloading the image, the first step is to see if there's anything wrong with the image. I like to use an online tool called [Foto Forensics](fotoforensics.com) as my starting point. Nothing wrong there.

Next we want to see if there are any files hidden in the image. Using `binwalk`, we can see there is a ZIP file hiding away. 

```shell
binwalk HumanisedPic.jpg
```

We can extract the zip with a tool like `foremost`. 

```shell
foremost HumanisedPic.jpg
```

We see there's also a text file inside the zip folder called `Congratulations.txt`. Using `7zip` we extract the `Congratulations.txt` file from the zip.

```shell
7z e *zipname*
```

And there we have it. The contact information to be into win the free ticket. Not a difficult challenge by most CTF standards, but an enjoyable test nevertheless. It should be noted that although this writeup seems brief and to the point, it did take me a couple of hours and I had to think about what I needed to do (and try a few things that didn't work) at some of the steps. I am still a beginner after all.

Thanks to Humanised Group for the fun challenge.
