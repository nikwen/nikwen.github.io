---
layout:     post
title:      "Plans regarding my free software activities"
date:       2016-01-22 01:30:00 +0100
categories: ubuntu
---

It's really time for me to give you an update on what I've been doing recently.

You may have noticed that you haven't heard as much from me as you used to during the last few weeks and there is a reason for that. In October I started studying mathematics at a big German university. This has taken away a big chunk of time from me that I used to spend on free software topics but, nevertheless, the courses are a lot of fun to me and I am glad I made this decision.

So what about the free software work that I actually *did*?

Some time ago I promised to write something up about the account-polld changes that landed with the OTA-8 update, so I will start with these.

In the past, some users ran into an issue where they changed their account passwords and thus account-polld wasn't able to log in anymore. Instead of recognizing that, however, account-polld continued to retry to log in. This resulted in account-polld retrying until the battery died and that happened really fast as the processor never went into suspend mode anymore. I prepared two patches for this issue and they eventually got merged.

There also was an issue which appeared when account-polld found a new message directly after startup and you had an image in the address book for the person who sent you the message. This was a really tricky bug as it turned out to be an issue where Go and Qt threading interfered in a very bad way, leading to the message not being displayed. This was also resolved by two patches I submitted and which you received with the OTA-8 update.

Besides that, Jonas Drange fixed a few bugs in both account-polld and ubuntu-push. These also fixed some rare cases where a message would not be displayed and they made the screen light up when the phone was locked and a message came in. Overall, good stuff!

In addition to that, I have had a few fixes merged into the calculator and calendar apps, with the most important feature being physical keyboard paste support for the calculator app. I also [gave a presentation on the Ubuntu Phone](http://nikwen.github.io/ubuntu/2015/11/08/first-time-talking-at-a-conference.html) in November.

Recently, however, I decided that I really wanted to become more active again. I honestly don't know how much time I will be able to devote to free software per week and there will definitely be weeks during which I will have no time for software development at all. However, I really want to pick up the [IMAP notifications](http://nikwen.github.io/ubuntu/2015/09/17/my-work-on-email-notifications.html) thing again.

Today I started by reviewing a great patch which Rene Paulokat sent in for the [Ubuntu Go QML template](https://github.com/nikwen/ubuntu-go-qml-template) and afterwards I reviewed some merge proposals for the Ubuntu terminal app.

During the next few days I plan to go through my marked Launchpad issues and unassign myself from a few of them in order to be able to focus on the things that are most important to me.

Here is hope that you will hear more from me again very soon.
