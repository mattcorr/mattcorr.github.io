---
title:  "MSDE is a real pain!"
header:
categories: 
  - Blog
  - Technology
---

After various issues at work, I am starting to really dislike Microsoft SQL Server Desktop edition. The SP4 update is nothing short of utter crap.

If you ever find yourself needing to install it, ensure you have the following:
_(this is for the update and not a clean install)_

* Your filesystem is NTFS
* The log on account for the MSSQLSERVER is the local system account
* You double check that the file copied across is correct. If say only half the file is copied it won’t let you know at all.
These two seemingly undocumented “features” have caused me a world of pain.

Hopefully no-one else will have to go through what I have suffered!