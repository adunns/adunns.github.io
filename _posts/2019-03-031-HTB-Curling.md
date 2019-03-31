---
layout: single
title: "HackTheBox Curling Writeup" 
related: true
---

Curling is an easy rated Linux box on www.hackthebox.eu worth 20 points. This is my second ever box so I'm still learning the ropes. Unfortunately I was trying this box on the day before it closed so I only got user before it closed. I managed to get root within the hour after it closed so no points for that. Yet somehow my user points were now 0 as well. Not sure what happened there.

First step as always is to run `nmap` and store it in our `nmap` folder:

`nmap -sC -sV -oA nmap/curling 10.10.10.150`

This will use the default scripts, do service detection and store the results in three formats in the folder specified.

![web credentials](/assets/images/curling1.PNG)

Not a lot going on here, but we can see there is a web service running Joomla. This seems like it would be a good entry point for a exploit. Maybe then we'll get our credentials to be able to log into the box via SSH.

So, lets fire up the webpage for the box. Ah, its a blog about curling. How neat.

![web credentials](/assets/images/curling2.PNG)Looking around the site, we don't find any new entry points. Looking at the interesting URLS like `/robots.txt` doesn't reveal anything. So, we'll fire up a directory brute forcer like `dirbuster`.

Using `dirbuster` with the medium wordlist and recursive scan we get a interruption shortly after starting:

![web credentials](/assets/images/curling3.PNG)

Apparently the hostfile at `http://10.10.10.150/index.php/2-uncategorised/logo/thereIsNoWayThat-You-CanBeThere/` has been removed. This means the directories and the displayed file probably no longer exist. If we examine the response HTML carefully, we see a listing for a `secret.txt` that has been commented out. Interesting. 

This actually made me think, did I actually check the source code of the index page properly? Scanning to the bottom of the live website we see the exact same commented out `secret.txt`. Welp. That's a lesson to myself, **always check the source code thoroughly.** We didn't even need `dirbuster` after all.

![web credentials](/assets/images/curling7.PNG)

Anyway, trying `10.10.10.150/secret.txt` reveals a secret file!

![web credentials](/assets/images/curling4.PNG)

This is a base64 string which we can translate back into ASCII. This gives us a string `Curling2018!`. This seems like a good candidate for a password for something. Lets try it against the the site admin panel. 

But wait, we don't have a username yet. We'll try admin, administrator, user, the common stuff. Nothing. Looking over the webpage posts again for hints we see the first post was signed by `floris`. Let's try that.

And it works. If we log in as `floris` with `Curling2018!` as the password we are now the admin of the site. 

![web credentials](/assets/images/curling5.PNG)

Theres not much we can do on the blog so let's see if we can get into the Joomla admin. To get there its simply:

`10.10.10.150/administrator`.

Trying the same login works and we're now into the Joomla administrator section.

![web credentials](/assets/images/curling6.PNG)****

Now we have admin, the next thing to do is to upload a PHP shell to the server to get access to the filesystem.

There are a lot of different ways to do this and a lot of different PHP shell variations. The tutorial I feel like a lot of people saw was [here](http://www.thehackerstore.net/2015/01/how-to-upload-shell-in-joomla-via-admin.html). Basically, go to the template manager and rewrite the `index.php` file of the template to your shell PHP code. Then navigate to `10.10.10.150/index.php` and your shell will be there. How do I know a lot of people probably used this? 

Well over the course of doing this box, there were **SO MANY** people overwriting the `index.php` file. In some cases this effectively destroyed the box until a reset (sometimes just the webpage would be dead, other times the admin area would also be dead). All that was accessible was the shell. A lot of people used the [p0wny shell](<https://github.com/flozz/p0wny-shell>) for this.

I wanted to find a way to get a shell **without** killing the box or disrupting it for others who were currently working on it. It took a bit more research but I found a good method (**Note:** see the link to LiveOverFlow's video at the bottom for a *much* simpler way to get the shell):

There's a built in PHP reverse shell called `weevely` that's actually conveniently built into Kali Linux (you can find instructions on [github](https://github.com/epinna/weevely3)). You generate a PHP file with a password and upload it somewhere to the server where the PHP code will be executed. I needed to find another location besides that damn `index.php` file. 

Luckily the Joomla administrator area has the ability to install extensions. I found a file explorer called [eXtplorer](https://extplorer.net/) which is a PHP based file manager. You can install this by downloading the zip and uploading it to `Extensions > Manage > Install`.

![web credentials](/assets/images/curling8.PNG)

Ok cool, our file explorer is installed. Next we go to `Components > eXtplorer` and it will open it up in a new window. Here we have access to a lot more files that exist on the website.

![web credentials](/assets/images/curling9.PNG)



In researching this where to put this non invasive file, I saw some posts of people asking what to do with malware they had found in their `/tmp` folders. This is because the `/tmp` folder usually has quite high (if not the highest) permissions so attackers often store malware here. 

What we're concerned with for our `weevely` shell is making the path to the shell accessible. There's a blank file called `index.html` already in the `tmp` folder.

![web credentials](/assets/images/curling10.PNG)

Let's see if the path is accessible i.e. `http://10.10.10.150/tmp/index.html`

![web credentials](/assets/images/curling11.PNG)

Success! The files are accessible. Seems we've found a perfect place to store the shell script. Let's upload it then. Checking that its there:

![web credentials](/assets/images/curling12.PNG)

Cool. Now we can go to our terminal and type:

`weevely http://10.10.10.150/tmp/shell.php *password*`

![web credentials](/assets/images/curling13.PNG)

And we're in. We have access to the web server as `www-data`. Time to find that `user.txt` file. 

Hunting around the file system, we get to the`floris` home directory. Here we can see our `user.txt` file. Unfortunately, we don't have permission to read it. 

![web credentials](/assets/images/curling16.PNG)

We'll have to be logged in as `floris` to get at that file. There is another interesting file here though:

![web credentials](/assets/images/curling14.PNG)

Lets see what's in password_backup:

![web credentials](/assets/images/curling17.PNG)

A hex dump. The first few characters (BZh) indicate that this dump is of a `BZip2` file. There's a few operations you can do to sort out this data in the terminal, but I have a tool for dealing with this mess: [CyberChef](https://gchq.github.io/CyberChef/). Dropping the hex dump in and using the `magic` recipe, CyberChef will attempt to decode the data with as many operations it needs:

![web credentials](/assets/images/curling15.PNG)

As you can see, I loaded up the recipe it suggested and viola, we have a password file. This MUST be our `SSH` password for the user `floris`. Lets try:

`ssh floris@10.10.10.150`

and if we enter the password, we are successfully logged in as user `floris`. Now let's get that `user.txt` file:

![web credentials](/assets/images/curling18.PNG)

Now we need to get root. Since we don't have any further information or files that could help us, we're looking at some sort of privilege escalation or something else sneaky. If we `ls -la` we see something interesting:

![web credentials](/assets/images/curling19.PNG)

`.bash_history` has a symbolic link to `/dev/null`. Hmm. For the method I used to get `root.txt` this was not anything of value. What is of value is what's in the`admin-area` folder. 

We have a `input` file and a `report` file:

![web credentials](/assets/images/curling22.PNG)

The contents of the `input` file are:

`url = "http://127.0.0.1"` and the `report` file contains the HTML for the root page of our webserver i.e. `index.php` of the curling site. 

With a name like Curling, this box probably has something to do with the `curl` command. A lot of these boxes have a hidden meaning to them e.g. Valentine focused on the Heartbleed bug. The `input` file matches one of the options for `curl -K`. The `-K` [flag](<https://curl.haxx.se/docs/manpage.html#-K>) means "specify an input file" to read arguments from. With this option we can also get files by using the `file://` URI. 

So, some process is running that gets what lives at the address in the `input` file gets and stores the output in the `report`. We can tell this is happening because if we try rewrite the `input` file with something like:

`url = "file:///etc/passwd"`

We get our `etc/passwd` file in `report` and then after a short interval the `input` file is rewritten to the default `url = "http://127.0.0.1"`. 

Also, if you look at the last modified dates you cans see the `input` and `report` files have been changed less than a minute ago. Something is definitely periodically changing these files.

![web credentials](/assets/images/curling23.PNG)

Let's use `watch` to see what if we can identify the process:

`watch -n1 'ps -ef | grep root | tail -15'`

The `-n *num*` flag will refresh the executing program every `*num*` seconds. There's a bunch of other unimportant processes running so we cut down to the `root` permission processes at the bottom of the list with `| grep root | tail -15`:

![web credentials](/assets/images/curling20.PNG)

And there we have our `curl` command that is being run. This confirms what we thought. Let's try use that file URI to see if we can get our `root.txt`. The `input` and `report` files belong to root so will run with root permissions. 

Let's change our `input` file to the location of our `root.txt` file:

`url = file:///root/root.txt`

Now if we `watch` the report file with:

`watch cat report`

And wait for a few moments we get this:

![web credentials](/assets/images/curling21.PNG)

Our `root.txt` hash. Awesome. If you wait around for a bit longer you'll see the `report` file revert to the `index.php` file due to that process kicking in again. 

So there you have it, that's how I solved the Curling box.

**Note:** LiveOverflow's [video](<https://www.youtube.com/watch?v=Paajc2Dupms>) for a more thorough rundown of this box (and getting a root shell at the end). He is a PRO at this stuff so a lot of the steps he takes are a lot simpler and smarter than what I've done.
