---
layout: single
title: "Two Factor Authentication with 1Password" 
related: true
---

There’s a lot of chatter in the information security community how SMS 2FA is awful, and with all the recent high-profile cases of sim swapping attacks due to useless Telco’s or susceptible employees, you would have good reason to think that. For further reading, see Brian Krebs *fairly recent* [article](https://krebsonsecurity.com/2018/08/reddit-breach-highlights-limits-of-sms-based-authentication/) on an example of why SMS based 2FA is bad and what we should be using in place of it.

A common safer alternative to SMS based 2FA is TOTP or [Time-based One-time Password.](https://en.wikipedia.org/wiki/Time-based_One-time_Password_algorithm) TOTP is similar to SMS 2FA by virtue that you still need a code to authenticate your identity into whatever application it is you are signing into. This code is computed based on the current time and a shared secret seed. It will be refreshed after a time period, usually about 30 seconds. You can read more about the technology behind it [here.](https://medium.freecodecamp.org/how-time-based-one-time-passwords-work-and-why-you-should-use-them-in-your-app-fdd2b9ed43c3)

Typically, users will have to install an application like [Google Authenticator](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2&hl=en) to see their TOTP code for their intended applications. Setting up 2FA is quite easy, users usually (and this is the typical use case) scan the QR code the application provides which will register it to your authenticator app.
 
![Google Authenticator app in action](/assets/images/GoogleAuth.png) 

*Google authenticator in action.*

So where does 1Password come in? If you have 1Password, there is no a need for an extra application like Google Authenticator. 1Password has this TOTP feature baked right in to the Android and IOS applications and Chrome extension. For the Chrome extension, go to your desired website and begin setting up 2FA. When you see the QR code on your screen, open the 1Password extension and you will see this:

![Scan the QR code](/assets/images/2FAavailable.png)

*Scan the QR Code.*

Click the circled QR code button and 1Password will scan the QR code on the web page and viola, it's registered. You should see a new section below your other details with the TOTP code. On some sites, the 2FA code will even autofill for you. 

For the Android and IOS apps, go to your login details, click `add new section` then `add new field` and click `one time password`. You'll see a QR code option, click that and the camera will open to allow you to scan the QR code on your screen. 

![1Password TOTP](https://blog.1password.com/posts/2015/totp-for-1password-users/countdown.gif) 

*From the 1Password blog, an example of a one-time password.*

The main reason I prefer this is that it’s device agnostic. That is to say, it will work over many devices and systems (e.g. IOS, Android, Chrome, Safari, Windows etc.). I’ve always worried about losing my mobile phone and the authenticator app along with it. If that situation occurs and you do not have your backup recovery codes for your applications, you’re in a world of trouble. Or imagine another situation at an internet café or library and you, for some reason, don’t have your mobile on you and need to login to an account. 

These days, I use 1Password for 2FA for the sites and applications that support it, and store my backup codes in the notes section for that login. Everything is in one place, accessible from any device I have an internet connection to. Convenience at no extra cost. 
