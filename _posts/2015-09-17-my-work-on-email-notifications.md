---
layout:     post
title:      "My work on email notifications (including an installable preview)"
date:       2015-09-17 23:30:00 +0200
categories: ubuntu
---

One day after the [UbuContest](http://ubucon.de/2015/contest) was announced, I started working on email notifications. I was looking for a new project anyway and as I [have](https://code.launchpad.net/~nikwen/account-polld/donechan-fix/+merge/238898) [fixed](https://code.launchpad.net/~nikwen/account-polld/authenticate-again-fix/+merge/238900) [multiple](https://code.launchpad.net/~nikwen/account-polld/directly-poll-with-new-account-data-fix/+merge/238901) [issues](https://code.launchpad.net/~nikwen/indicator-messages/clear-all-unescape-fix/+merge/258261) [related to](https://code.launchpad.net/~nikwen/ubuntu-push/gmail-missed-messages-cleanup-fix-new/+merge/264919) Gmail notifications in the past (they use the Gmail API), implementing general-purpose email notifications looked like a [great challenge](https://bugs.launchpad.net/dekko/+bug/1421923) to me.

I'll start by saying that the work hasn't been finished yet. Indeed, there's still much to do. However, I've been making huge progress and the countless hours I've invested have really been paying off recently.

Let's cover some basic background information first.

## Why email notifications are so difficult ##

As you might know, on Ubuntu apps are not allowed to run any kind of background services. The moment an app is sent into the background by the user, the application is stopped. (Well, actually 5 seconds later.) As an app has no way of running any code when it is paused, email clients like Dekko aren't able to check for incomming emails when they are in the background. Hence, they are also not able to display any notifications for those messages.

## How this can be solved ##

There are basically two ways of how this can be solved, both of which have their advantages and disadvantages.

The first one would be to run a server somewhere on the internet which checks for new messages regularly and then sends push notifications to the phone using Canonical's push servers which work in the same way as Google's cloud messaging framework. That, however, requires either running your own server, giving your login credentials to someone who runs a public server or convincing your vendor to support Ubuntu's push messaging framework. The first option is not really suitable for the masses, the second one gives other people full access to my inbox and the third one is nothing that I would consider realistic at the moment.

The second way would be implementing this as a system service.

You might have been wondering how the Gmail, Facebook and Twitter notifications work if apps cannot run in the background. The answer is that those notifications are provided by a system service called `account-polld`. This service takes a supported online account and checks in regular intervals if there are new messages for which notifications need to be displayed. This is the way I've also gone for email notifications.

## The user story ##

First of all, the user has to set up an email account in the system settings.

[![Screenshot of the setup dialog]({{ site.url }}/img/posts/2015-09-17-my-work-on-email-notifications/small/account-setup.png)]({{ site.url }}/img/posts/2015-09-17-my-work-on-email-notifications/large/account-setup.png)

Ultimately, he will have to enter his email address, his password and the location of the respective [IMAP](https://en.wikipedia.org/wiki/Internet_Message_Access_Protocol) server here. As you can see from the screenshot, the last bit hasn't been fully implemented yet and it defaults to the Gmail IMAP server.

Afterwards, the user will have to grant an application access to the online account.

[![Screenshot of the grant access screen]({{ site.url }}/img/posts/2015-09-17-my-work-on-email-notifications/small/grant-access.png)]({{ site.url }}/img/posts/2015-09-17-my-work-on-email-notifications/large/grant-access.png)

When that has been done, account-polld can instantiate the IMAP plugin and include email notification in its regular poll cycles.

From then on, the user will receive notifications for unread emails.

[![Screenshot of an IMAP notification]({{ site.url }}/img/posts/2015-09-17-my-work-on-email-notifications/small/notification.png)]({{ site.url }}/img/posts/2015-09-17-my-work-on-email-notifications/large/notification.png)

When he clicks on a notification, the system calls a custom URI which the respective email application will then be able to use to display the respective message.

[![Screenshot of the demo app utilizing the URL dispatcher]({{ site.url }}/img/posts/2015-09-17-my-work-on-email-notifications/small/uri-handling.png)]({{ site.url }}/img/posts/2015-09-17-my-work-on-email-notifications/large/uri-handling.png)

The process can be sped up a bit if the user registers the online account directly from inside an application that uses the online accounts API.

## What works and what doesn't ##

As you can tell from the screenshots above, I've been receiving email notifications for quite some time now.

account-polld instantiates the IMAP plugin correctly and checks for unread emails. It is able to display emails correctly (including MIME/multipart messages) and places an "overflow item" in the notifications tray when there are too many unread emails at once.

I have also put time into writing a preliminary version of the online accounts plugin, which can currently be used to set up Gmail accounts just fine. It is currently still part of my test application as that makes development easier but will eventually have to be included in the default set of online accounts shipped by the OS.

Said test application is also an example of how a developer can make use of the email notification. It shows how he has to configure his application for using the online accounts plugin, how he has to set up a push helper so that his application can be used for displaying notifications and how he can deal with the URIs sent by the URL dispatcher.

What doesn't work at the moment is setting up email accounts from other providers than Gmail. That is not because it would be difficult to tell the library I used to access the respective IMAP servers but because account-polld so far only supported OAuth accounts. I had to rewrite parts of the account detection code so that it first of all can make use of accounts with username/password authentication and secondly also retrieve other information from the online account besides the login credentials. The latter part is still work in progress. (This was particularly hard because the documentation for password authenticated accounts is close to none. Someone definitely needs to improve this!)

Additionally, there are still a few things to improve in the plugin for account-polld itself to lower its data usage when querying for unread emails and to improve it in other aspects. Finally, as mentioned, the online accounts plugin needs to be included in the default set from the `account-plugins` package.

## How to install the preview on your phone ##

Now the part that all of you have been waiting for. This is how to install a preview of this on your phone.

I'll start with the usual disclaimer. Updating the phone while having the modified account-polld debian package installed can break your phone if the update ships a new version of account-polld. Additionally, there is the (relatively low) risk that the current version might not work with a future version of the Ubuntu installation image so that the notifications break. I'm not responsible for anything related to that. You'll have to install this on your own risk and I would only recommend that if you know what you are doing.

As soon as email notifications get included in the images by default, you will furthermore have to make sure to remove the demo application.

To install our custom debian package, you'll first need to [make your image writable](http://developer.ubuntu.com/en/start/ubuntu-for-devices/installing-ubuntu-for-devices/#install-options). Read up on the dangers related to that on the referenced page before proceeding!

Afterwards, download the modified account-polld package as well as the required test application to your phone using the following commands:

{% highlight bash %}
wget {{ site.url }}/assets/imap-accounts/preview-1/account-polld_0.1+15.04.20150410-0ubuntu1_armhf.deb
wget {{ site.url }}/assets/imap-accounts/preview-1/imap-accounts.nikwen_0.1_all.click
{% endhighlight %}

These packages have been tested on the Nexus 4 daily build of the rc-proposed channel from September 17, 2015. There might be incompatibilities with later versions.

Then install the packages and reboot your phone:

{% highlight bash %}
pkcon install-local imap-accounts.nikwen_0.1_all.click --allow-untrusted
sudo dpkg -i account-polld_0.1+15.04.20150410-0ubuntu1_armhf.deb
sudo reboot
{% endhighlight %}

You should be good to go now! :) Proceed with setting up your accounts as outlined above.

## A note to client developers ##

This passage is dedicated to email client developers.

Please note that this is a preliminary version. Things *will* change so I do not recommend incorporating this into your apps already. This mostly refers to the online accounts plugin being shipped with the OS in the future and the URI received from the URL dispatcher being changed in the future. The latter will, however, be along the lines of `imap://{accountId}/uid/{messageUid}` but somehow incorporating the target application, either in the protocol prefix or as kind of a "domain name".

## A request of my own ##

I did all of this work with the UbuContest in mind. While I haven't finished this yet, I have done a substantial part of the work. I now need you to [nominate](http://ubucon.de/2015/contest/nominate-individual) me for the contest and add a link to this blog post. I promise I will bring this to an end and finish the implementation.

Thanks a lot in advance and have fun!

**EDIT:** I [won](http://ubucon.de/node/971)!
