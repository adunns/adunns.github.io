---
layout: single
title: "OverTheWire Leviathan Challenges" 
related: true
---

Leviathan is another set of beginner Linux challenges at [OverTheWire](www.overthewire.org). These challenges previously lived at `intruded.net` and are quite different from the Bandit challenges. You **SHOULD** be trying these challenges yourself. Writeups and answer guides like this should be your last resort if you need a hint.

These challenges have no instructions, we just have to use some Linux command line knowledge to find the passwords to the next levels.

**Note:** if we get sick of typing the passwords we can save them in a file and enter them with the `ssh` command:

![img](/assets/images/Leviathan1.png)

`$()` is a way of executing a command inside the command, or as stack overflow describes it: 

> The expression `$(command)` is a modern synonym for `command` which stands for command substitution; it means, run command and put its output here.

## Leviathan 0

We have to search the leviathan home directory for hidden files / folders. We find a backup folder which contains a web browser bookmarks file. There is a lot of information here, so let’s try something obvious like grepping for the word *“leviathan”* (we can also `grep` for password or whatever):

![img](/assets/images/Leviathan2.png)

Viola, the password to leviathan 1.

**Leviathan 1 pass:** rioGegei8m

## Leviathan 1

Here we have to know of a certain command called `ltrace` here to make it easier. `ltrace` means library trace and will trace the library calls of an executable. We have a `setuid` executable here for leviathan 2 and when we run it, it asks for a password. Give it the wrong password and it exits.

**Note:** we can use `strace` for even more heavy lifting checking system calls rather than library calls. 

![img](/assets/images/Leviathan3.png)

We can` hexdump` or `strings` the binary to find some readable text and if we are looking hard enough, we can see a hint to the 1995 movie [Hackers](<https://www.imdb.com/title/tt0113243/?ref_=nv_sr_1>) ...

![gif](https://media.giphy.com/media/14kdiJUblbWBXy/giphy.gif)

Which, in the movie, mentions the most common passwords are secret, love, sex and god:

![img](/assets/images/Leviathan4.png)

Still, this isn’t clear so let’s run ltrace on the binary.

![img](/assets/images/Leviathan5.png)

We can see that whatever we enter (in this case I just pressed <kbd>Spacebar</kbd>) is checked against the string *“sex”* using a `C` function called `strcmp` (string compare). So, this should be obvious that the password is *“sex”*. We can try this and we will have leviathan 2 privileges. We can then check the password file in `/etc/leviathan_pass/leviathan2`.

![img](/assets/images/Leviathan6.png)

**Leviathan 2 pass:** ougahZi8Ta

## Leviathan 2

We find a file called `printfile`. It is a `setuid` binary meaning it is owned by leviathan 3. We can therefore, like in the bandit challenges, use the binary file privileges to get the password. The `printfile` binary does what it says - prints a file. 

First thing to check would be trying to print the `leviathan3` password:

![img](/assets/images/Leviathan7.png)

But it does not let us for some reason. Next thing is to go to `ltrace`:

![img](/assets/images/Leviathan8.png)

We can see it runs the `access` function that will check the permissions on the file and as above we have a return code of `-1` which means it fails. Let’s run `printfile` on something we can print and see what happens:

![img](/assets/images/Leviathan9.png)

We can see it checks the permissions and then takes the file path and throws it on the end of the `/bin/cat` command to `cat` the file out. This is tricky, but there is a command execution bug with this that we can exploit. We need to make a file in our `tmp` directory first.

**Useful note:** we can use the `mktemp` command to make a temporary file or in this case a temporary directory.

![img](/assets/images/Leviathan10.png)

Let’s make a file called `fake;bash`. The semi colon indicates a new command on the same line (`&&` does the same). If we run this:

![img](/assets/images/Leviathan11.png)

We can see it created the bash file and put us in a new shell, hence why exit does not remove us from the server. We need to use `‘’` or`“”` to tell Linux to interpret the file name literally:

**Note:** when away from the home directory using `~` will reference the current home directory.

![img](/assets/images/Leviathan12.png)

And you can see if we run it with the quotes, we get a permission denied as it was trying to find a file called `fake` which does not exist. The file was called `fake;bash` but the bash part got executed. Therefore, we find ourselves in a new shell as `leviathan3` and we can get the password.

**Solution 2:**

We can use symbolic links to also get the password here using a syntax hack on the binary. If we create the temp file with spaces in it and run it against `printfile` we get this:

![img](/assets/images/Leviathan13.png)

As we can see there is a syntax security hole here where the binary is looking to cat only the second half of the file name **if it has a space.** This means that it is treating the `new` and `file` as two separate files. We can exploit this by creating a symbolic link to the first or second file:

![img](/assets/images/Leviathan14.png)

`ln` creates a link, and `-s` means symbolic. We don’t have the permissions to create a hard link. Now as we can see the file called `new` has a symbolic link to the password file. Let’s run `printfile` again:

![img](/assets/images/Leviathan15.png)

Notice we can replace the quotations for the escape character for the space.

**Note:** For this solution we must create our own `tmp` directory by hand like in the above picture else we do will not have permissions to read the files. Very important.

**Leviathan 3 pass:** Ahdiemoo1j

## Leviathan 3

We have a file called `level3`. Once again it is a `setuid` file with the owner being `leviathan4`. It wants a password. Let’s `ltrace`:

![img](/assets/images/Leviathan16.png)

We can see it is trying to obfuscate the password by calling `strcmp` twice, the first redundantly. If we look at the one that compares the input it compares it to `snlprintf`. If we try that, boom, we have permissions for `leviathan4`. Easy as that.

**Leviathan 4 pass:** vuH0coox6m

## Leviathan 4

When we check the home directory, we see nothing. We’ll try list all. What we get is a trash directory. If we navigate to it, we find a binary called `bin` with `setuid` privileges for `leviathan5`. Running the file, we get a binary stream. If we `ltrace` it we can see it using `fopen`, which is file open, to open the leviathan file in `r` mode (read mode).

![img](/assets/images/Leviathan17.png)

The binary is obviously the password, and we can use a web converter to convert this easily. As part of the learning experience, we will to try convert this in the command line.

We will need to use `bc` called binary converter / base converter to get hex then `xxd` to get our ASCII readable text. We can use a few tools here, Python or Perl or just Bash stuff, whatever works. I’ll show the Bash way.

We can write a Bash line with a lot of pipes to get the hex output then convert it to ASCII. If we translate the white space and replace it with new lines, we get each character on a new line. Then we can convert the binary strings to hexadecimal using the `bc` tool. Then we can delete the new lines (with the `-d` flag) and pipe it to `xxd` and convert it to ASCII using the `-r` and `-p` flags (reverse the operation, turn hex to binary and print postscript / without the columns or line numberings).

![img](/assets/images/Leviathan18.png)

Basically, the `xxd -r -p` command takes our hex and reverts it to text and prints it out the plain dump.

![img](/assets/images/Leviathan19.png)

**Leviathan 5 pass:** Tith4cokei

## Leviathan 5

Using some of the tools from previous challenges we can get the password quite easily. We have a binary which looks for a file in `tmp` called `file.log`. If we create this file, we can see how the binary interacts:

![img](/assets/images/Leviathan20.png)

It gets the characters from the file and basically prints them out then deletes the file. So, is there a way where we can read a file from another file, given that we have `leviathan6` permissions when we run this? Yes, using symbolic links. If we create the `file.log` it will not work, we have to create the file during the link operation. And then run the binary and the password is ours.

![img](/assets/images/Leviathan21.png)

**Leviathan 6 pass:** UgaoFee4li           

## Leviathan 6

The home directory contains a binary called `leviathan6.bin`. Running it tells us we need a 4-digit pin, and the correct one will give us the correct password. The obvious solution is to brute force all 10000 combinations. 

![img](/assets/images/Leviathan22.png)

One way is to write a small script like so in our `tmp` folder:

**Note:** this can also just be done in one line in our shell like so:

![img](/assets/images/Leviathan23.png)

As a script:

![img](/assets/images/Leviathan24.png)

This will try all the combinations and run them against the binary:

![img](/assets/images/Leviathan25.png) 

Until it stops. We now know that 7123 is the pin as we have a shell for leviathan7.

![img](/assets/images/Leviathan26.png)

If we enter this against the binary (or just keep typing in the above step) we get our shell for `leviathan7` privileges.

![img](/assets/images/Leviathan27.png)

**Leviathan 7 pass:** ahy7MaeBo9

## Leviathan 7

This is just a congratulatory level with a nice message and a request that I have so brazenly ignored by publishing this:

![img](/assets/images/Leviathan28.png)



And that's all the Leviathan challenges! If you have any questions, ask away.
