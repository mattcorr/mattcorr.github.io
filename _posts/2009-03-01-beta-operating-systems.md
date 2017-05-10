---
title:  "The joys of playing with beta software on beta operating systems"
excerpt: "Fun and games with beta versions of Kaspersky Anti Virus on beta Windows 7!"
categories: 
  - Technology
  - Computers
---

Like a lot of other technical people out there I have installed the Windows 7 Build 7000 on my main PC. Overall it’s been a refreshing experience and I am happy with it. I do look forward to the final release _apparently_ later on this year. I installed the trial of of Kaspersky Anti Virus 8.0

However over the last week or so I have been getting a lot of the dreaded Blue Screens of Death (BSOD). It was always with the error IRQL_NOT_LESS_OR_EQUAL and Stop code 0x0000000A. This randomly appearing BSOD usually seemed to occur during heavy network activity such as Skype calls _(much to the frustration of Femke)_ and network file copies.

After trying different drivers and rolling back newly installed drivers with no result, I looked into issues with the Windows 7 supported anti virus applications. There were lots of forum posts saying about how Windows 7 was blue screening with most anti virus apps installed (not just Kaspersky). It seemed to be more frequent when network access was performed. This seemed rather similar to my symptoms so I tried to uninstall it but it would not uninstall? WTF? I googled more and was lead to [this page](http://support.kaspersky.com/faq/?qid=208279463) which offered a separate application to uninstall the Kaspersky Anti Virus.

It did not say anything about working with Windows 7, but I figured “What the hell. Give it a try”. 
I ran it and worked and the AV did disappear from my system. “Cool!” I thought as I rebooted. Then my USB devices would not respond (mouse and keyboard) which meant I could not log into my PC. 

After trying in vain to repair my installation, I realised it was hosed and reformatted and put Win 7 back on. This time without Kaspersky! So far so good, but the real test will come when I do some video Skype calls.

Moral of the story 1: Installing beta software on a beta operating system can be a bitch sometimes!

Moral of the story 2: Don’t install Kaspersky Anti-Virus 8.0 on Windows 7 Build 7000!

If anyone else out there has had experiences with the dreaded IRQL_NOT_LESS_OR_EQUAL BSOD please let me know of your results!
