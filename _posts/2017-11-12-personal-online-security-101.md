---
layout: post
title: Personal Online Security 101
tags: [security]
comments: false
---
As a programmer, I take information security seriously because security needs to be baked into the process of developing web applications. For that reason, I try to keep up with recent security news and practices by following security experts such as [Troy Hunt](https://www.troyhunt.com/), listening to the [Security Now podcast](https://twit.tv/shows/security-now), and taking [Pluralsight](https://www.pluralsight.com/) courses on online security. As a result, while I am not a security researcher, I think I know more about information security than most. At the same time, I see a lot of people not taking basic steps necessary to keep their information protected. So here I present five basic tips that everyone should follow to help protect themselves online.

## Keep your computers/devices updated

Pretty much every week on Security Now I hear Steve Gibson explain the ins and outs of a new security exploit. While some of them take advantages of “zero day” vulnerabilities -- meaning unknown security flaws that attackers had been using -- many exploits take advantage of known vulnerabilities in unpatched systems. In other words, they rely on the fact that many systems have not been updated even though there is an update available that fixes the problem.

Responsible security researchers report vulnerabilities to the vendor and give the vendor time to fix the issue with their systems. This means many vulnerabilities are fixed before the bad guys learn about them and try to exploit them. Companies like Google have bug bounties and their own security researchers to try to find vulnerabilities so they can be fixed before attackers exploit them. But if your system isn’t updated, you can be left vulnerable. While running system updates can be an annoyance, it’s best not to put it off too long or you may be left dealing with a malware infected system. Better yet, when possible have your computers and devices set to auto-update.

## Use strong, unique passwords

Most people still use weak passwords. It’s been [reported](https://keepersecurity.com/public/Most-Common-Passwords-of-2016-Keeper-Security-Study.pdf) that really simple passwords such as “123456” and “password” are most commonly used. But even more obscure phrases are relatively easy to crack using automated programs that try a dictionary of words and phrases. The best thing to do is to use a [strong password generator](https://strongpasswordgenerator.com/) to create a random password.

Just as important as using a strong password, it is important to use unique passwords. In other words, each account you have should have a different password. One reason for this is that it is inevitable that some sites will be hacked and password data exposed. You can check the [Have I been pwned](https://haveibeenpwned.com/) website to see if your email has been exposed in a known security breach. By using a unique password for each site, you can ensure that any password data leaked will not help attackers access your accounts on other sites.

Of course it is difficult, if not impossible, to remember many strong passwords. That is where password managers come in. Applications such as [LastPass](https://www.lastpass.com/) and [Encryptr](https://spideroak.com/encryptr/) should be used to store your passwords.

Also, for sites that have ‘security questions’ (such as ‘What concert did you first attend?’), you are better off using random generated password and save the question and answer instead of using the real answer. This will thwart an attack where a hacker may have access to your personal history.

## Use Two Factor Authentication (2FA) (but not with SMS)

The basic idea behind Two Factor Authentication is that if a hacker gets a hold your password, they still need something else (a second factor, if you will) to be able to access your account. The most common way this is done is with text messages (SMS): you register your cell phone number with your account and when you sign on, the site occasionally requires you to enter a code they send you via text message.

Unfortunately SMS (text messaging) is not secure. It is easily spoofed and attackers have been known to convince phone companies to switch numbers to their own phones. For this reason, I highly recommend that you use an authenticator app like Google Authenticator (available for [Android](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2) and [iOS](https://itunes.apple.com/us/app/google-authenticator/id388497605)), Microsoft Authenticator (available for [Android](https://play.google.com/store/apps/details?id=com.azure.authenticator), and [iOS](https://itunes.apple.com/app/id983156458), and [Windows Phone](https://www.microsoft.com/en-us/store/p/microsoft-authenticator/9nblgggzmcj6)), or Authy (available for [Android](https://play.google.com/store/apps/details?id=com.authy.authy), [iOS](https://itunes.apple.com/us/app/authy/id494168017), and [Google Chrome](https://chrome.google.com/webstore/detail/authy/gaedmjdfmmahhbjefcbgaolhhanlaolb?hl=en)). These apps generate short-lived codes for providing a second factor for authentication. I prefer Authy because it supports synchronizing your accounts across multiple devices).

## Look for HTTPS

Site’s that have a user system (in other words, sites that allow you to register and login) or sites that have forms that collect sensitive information, should always use HTTPS to encrypt their communications. In fact, as a web developer, I recommend that [all sites use HTTPS](https://www.flightpath.com/blog/2016/12/5-reasons-to-use-https-everywhere/). Browsers will display a lock icon (green in Chrome and Firefox) if the site is secure. If you are banking or shopping online (or entering any other sensitive information), make sure that the site is loading over HTTPS.

## Be wary of what apps and sites you trust

This one is a little fuzzier then the previous guidelines but just as important. It’s important to be suspicious and limit the apps and sites you trust. Think twice about downloading and installing free programs. Is it coming from a trusted developer? Does it have a good reputation? That doesn’t guarantee you are safe (a download file for CCleaner, a popular free program for cleaning your PC was recently hacked and included Malware), but you are safer when you limit your exposure. The same goes for websites - avoid shopping and banking at sites you don’t know are trustworthy.

I hope the above tips are helpful and welcome your comments!