---
layout: single
title: "Changing my Password Mentality" 
related: true
comments: true
---

When I was a bit younger, I used to have a favourite password for everything. Given that I’m still not that old, my ignorance was not that damaging. I still adhered to the basic password rules about not using a dictionary word, at least 8 characters, using a capital and a number and all that. The sad thing is even that is still too much effort for many people today.

I got interested in technology and then found my way into the information security space. After reading some things and using common sense, I came up with a new plan: make 3 passwords depending on the level of the data sensitivity, and change all my accounts accordingly. A super secure one for things like banking, one for moderately sensitive stuff like email, and one for the other random accounts I owned (e.g sites which had no credit card details attached like an IMDB account). I thought this was a pretty good idea and served me as my password policy for a number of years.

In the last year, my password mentality has evolved further. I’ve been reading Troy Hunt’s work for a while [now](https://www.troyhunt.com/) and following him on twitter (which you should too if you’re interested in security) and he pushes the use of a password manager very heavily. Hell, even if you don’t want to use a password manager, he suggests keeping a physical notebook to log your passwords. Anything is better than weak passwords, or reusing them.

He uses 1password as his password manager of choice and they have also done some great work integrating his HaveIBeenPwned service into their product. If you’re the type of person to have a single / few passwords, I would check them [here](https://haveibeenpwned.com/Passwords) to see if they have already been exposed in a breach (possibly by one of your accounts or someone else using the same password as you).

Anyway, I trust Troy’s advice and had been meaning to upgrade my password security to a password manager for a while so I decided to finally pull the trigger and fork out the few dollars a month for the 1password subscription.

In short, it’s been fantastic. I use it every single day, it integrates well over my mobile devices and the chrome extension and the interface is clean and easy to use. The password generator, although not anything special, is handy when I’m on a new website to sign up for something new and it autofills a suggested password. The watchtower functionality is awesome. It will check your passwords against the HaveIBeenPwned database and lets you know if your passwords have been seen in a breach. It will also tell you if your passwords are weak, reused or if your accounts have 2FA and you have not enabled them. You can read more about this [here](https://blog.1password.com/introducing-watchtower-2.0-the-turret-becomes-a-castle/) on the 1password blog.

![Watchtower Integration from the 1password blog](https://blog.1password.com/posts/2018/b5-wt-2/dashboard.png) *Watchtower Integration from the 1password blog*
 
The price you pay when using a password manager is always going to be convenience. It might be as much as ten seconds for you to enter your vault password or to scan your fingerprint. Or when on a device you don’t own, opening the app to copy down the right password to log in. This is a small trade off for the confidence you will have in the safety of your personal data.

Using a manager, the only password you will have to remember is the master password which lets you into your vault (when using a device with no biometrics). There is an excellent xkcd [comic](https://xkcd.com/936/) to help you with creating one, or a password in general if you haven’t jumped on the password manager bandwagon yet. 

I’ve also got a small post explaining how to use 1Password for your TOTP 2FA codes as well. You will no longer need to use an app like google authenticator. Having less apps is always better. Read that [here](https://adunns.github.io/Two-Factor-Authentication-with-1Password/).

It should be noted that there are free alternatives to 1password, such as Chromes built in password manager which is perfectly fine for most people. There is also a myriad of other paid password managers (I’ve heard LastPass is also good, and it has a free tier).

As an example, as many were, I was in the recent 2018 Quora breach. This was suddenly just a minor inconvenience. I simply went to Quora, generated a new password, changed it and I was on my way. That’s the kind of safety net having a password manager can afford.

What’s next? I’m thinking about signing up to Google’s [Advanced Protection Program](https://landing.google.com/advancedprotection/). This is basically enforcing the use of physical security keys like the Yubikey for any Google services. You physically need to insert a USB to your PC / Laptop or connect via NFC/USB-C on your phone to access the specified account. It’s the most secure form of 2FA we can possibly have for the everyday person. See an article [here](https://krebsonsecurity.com/2018/07/google-security-keys-neutralized-employee-phishing/#more-44128) from Brian Krebs on how Google has effectively eliminated phishing by using these security keys. 

This post is not a push towards you to use 1password, but more of a story of how my personal thinking around the topic of password security has changed over the years. I’m sure many who are in the same position had a similar path or you’re currently considering using a password manager. I strongly recommend it.

In retrospect, it’s hard to believe I didn’t transition earlier, but I’m glad I did. The next hurdle will be trying to convince the parents and the rest of the family to start using one. That may take some work.

