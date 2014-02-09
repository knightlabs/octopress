---
layout: post
title: "HOWTO enjoy changing your exchange password"
date: 2013-11-14 08:09
comments: true
categories: 
---
Login to Outlook Web Access. Go to change password. Type in new password. Have new password rejected by the AD. Just decide to increment a number at the end of your old password. Hit save.

You did collect every device you have that talks to your mailbox in front of you before clicking save, right?

Edit passwords in Mail.app accounts for both Exchange (disabled) account and BBBB IMAP (enabled) accounts. Don't forget to change the SMTP server password. Repeat for Sparrow. And Outlook if you have it. Repeat for Calendar.app, if you use it. Repeat for System Preferences > Mail, Contacts & Calendars, to be sure.

Repeat for your phone and tablet – hurry on these because by now you're probably a couple of failed attempts into getting mail, and some number greater than 2 is the magic one where you get cut off for an hour.

Repeat for all your VPN clients. Repeat for stored keychain passwords for the BbWireless networks in the offices. Repeat for any other software you have that auto-logs-in to the AD.

Technically, at this point, you're probably good – saved web passwords can be updated as needed. But to be sure, wait 10 minutes. Then go try logging back into OWA.
