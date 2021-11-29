---
layout: "post"
title: "Nextcloud: Honest review"
date: "2020-06-17 22:15"
categories: [cloud]
tags: [google, privacy, cloud, nextcloud]
description: "If you are planning to install Nextcloud, this post should be your first step in the process. This is a self-hosted, honest review of Nextcloud and its integrations."
image: "assets/img/posts/2020/2020-06-17-nextcloud-review.jpg"
redirecturl: "https://www.crispy-engineering.com/nextcloud-honest-review/"
---

You have probably went through short tutorial on how to setup Nextcloud in your private environment. If you haven't, make sure to check [Setting up Nextcloud as alternative to Google Drive]({{ site.url }}/posts/2020/06/nextcloud-alternative-to-google-services).  

I am using Nextcloud for 4 weeks already, and this is my review.

## Initial Nextcloud setup

In post install process, we have to go through the process of setting up some Nextcloud features of interest. First I've moved to security and turned on 2FA, by installing [TOTP Provider](https://apps.nextcloud.com/apps/twofactor_totp) application. You can go one step further and play with IP geo-blocking and brute-force IP blocking. So far I haven't made any of these settings.

![Nextcloud Two-Factor authentication]({{ site.url }}/assets/img/posts/2020/2020-06-16-nextcloud-totp.png){: width="200px" }

## Installing the apps

Under the _Apps_ menu, there is a categorized _AppStore_ with plenty of apps to choose from. I have installed Calendar, Contacts, Tasks, Mail and Carnet. All of these applications can be synchronized via Cal/Card/WebDAV to mobile phone via the application I'll install later.  
Out of my current experience, those applications with 4.5 stars and applications with _Featured_ flag are worth checking.

Reviewing each application is pointless. It's enough to say that the applications are working reliably. There are usual UI glitches here and there on Mail and Calendar, and UI/UX isn't perfect but it's acceptable. I guess I get used to Google apps UI for years and expecting all other apps to work the same way.

Installing the applications is straight-forward process. In the beginning, I had a trouble of finding installed app, as it isn't under the _Apps_ menu, instead, it's under the _Settings_ menu. Some apps are configured in _Personal_ section, while some others are configured _Administration_ section. This tends to confuse the user.  
Navigation is confusing a bit in the beginning, but once you get to know what to expect, it's much better.

## External storage

Without going into details how (this will be explained in the future post), I have attached two directories from my home NAS as external storage.  

![Nextcloud external storage]({{ site.url }}/assets/img/posts/2020/2020-06-16-nextcloud-external-storage.jpg)

Now, I'm able to browse the photos from my NAS located in my home, via Nextcloud instance running on some remote VPS. As visible on the screenshot above, I've added _GDrive Pictures_ directory, which is the directory synchronized with Google Drive at the same time. If you upload something to Google Drive, that file will be downloaded to my NAS and it'll show up in Nextcloud as well.

As shown on screenshot above, setting up the external storage is simple as choosing the protocol and filling the fields. I had it working from 6th or 7th try, as you have to guess where to put `/` in the directory path and to guess the _domain_ value, `WORKGROUP` on the screenshot.  
Once you pass that, it'll work stable and without issues.

## Thumbnail generator

Having in mind that I've connected home NAS with photos originating from Google Drive and photos originating from mobile phones as well, generating thumbnails is massive work initially.  You might think, why not just opening the directory and thumbnails will be generated in flight? Yes, you're right, but that'd be quite slow. All those images have to be uploaded from home NAS to Nextcloud running on VPS in order to generate thumbnails. You'd have to wait for the whole process to complete in order to see actual photos in the gallery.  
For big galleries, this is a deal-breaker, as being unable to browse your photos "instantly", makes the product unusable.  
Therefore, we have to pre-generate small thumbnails (called preview files from now one) which will be loaded in gallery when certain directory is opened.

By default, Nextcloud will generate files from 32x32 to 4096x4096 with step of power of 4 in between. For some photos, this means using more storage than initial file takes. For my >200GB media collection, this means generating preview files with almost the same size as original file. This is also a deal-breaker as I don't have storage that large on my VPS. Instead, I have 20GB only :).

I come up with two strategies for preview files:

1. I'll force the nextcloud to store the preview files on NAS as well
2. I'll have to limit the size of thumbnails somehow.

### Preview files on NAS

Having preview files on NAS at home is a solution that works. Those files are small and are transfered at decent speeds to Nextcloud server. However, as VPS comes up with limited monthly bandwidth, that means that preview files would be transfered from home to VPS and from VPS to client browser. This would happen each time and for each user. Eventually, traffic builds up and bandwidth might be breached.  
Additionally, it's quite hard to store only preview files on NAS. Due to coupled directory structure, storing preview files assumes storing some other metadata on NAS as well and this slows down the Nextcloud too much.

### Preview generator

In the AppStore, there is an app called [_Preview generator_](https://apps.nextcloud.com/apps/previewgenerator). This application is used to generate previews, of course, but additionally we can do it from the console, instead of opening the directory containing the photos.  

- First issue you'll discover is that there is no GUI setup for it. This has to be done through the configuration files on the filesystem.
- Next you'll realize that simply running the console commands won't do much as you'll get preview files with the same size. Therefore we have to find a way to configure the preview generator to generate smaller files.
- Preview generator configuration isn't simple. Additionally, you won't find unique solution in the forums. Instead, you'll find a couple of solutions, claiming to be correct, but mutually quite different.

After whole week of try'n'errors with preview generator configurations I've manage to set it up and to have only necessary preview sizes for web gallery and mobile gallery, including bigger image compression. I've ended up with 3-4 GB of preview files, which is acceptable.

By utilizing the crontab, I'm running preview generator 4 times an hour, to generate previews for freshly uploaded content.

I'm not sure whether this become a huge issue for me as I have a tiny VPS, but I cannot escape the impression of this feature not being scalable enough. If you're offering the product called private cloud, I know you're assuming that users will install it on massive storage, but there are many users like me installing it on small VPS or on Raspberry with external storage attached and getting into preview generator problem instantly.  
Another issue with these preview files is the computing power required to generate a couple of versions of same image and compress it. Running this on Raspberry takes days and days. I haven't tried it thought, that's something I read on forums.

If you're connecting your home storage to remote nextcloud, make sure to run the preview generator once everything is set up, as you'll find yourself re-uploading whole collection multiple times (as I did).

## Mobile app

As a mobile developer myself, I might be too harsh on reviewing the mobile app. I've installed android version and connected it to my nextcloud instance. That worked instantly!  
Out of main features for mobile application, I'd single-out browsing the nextcloud files and auto-upload feature.  

### Auto-Upload

Auto-Upload feature scans for changes on local directories on mobile phones (read: detects new photos) and uploads them to Nextcloud. It offers some predefined directory upload configuration, however I haven't found those much useful as it doesn't fit my environment. There is a super-cool _Custom directory_ option where we can choose local directory to watch and remote directory to upload the files to. That's what I used.

I see this feature as a main use-case for the app, and you expect it to work flawlesly, however, it's pretty buggy.

![Nextcloud Auto-Upload fail]({{ site.url }}/assets/img/posts/2020/2020-06-16-nextcloud-auto-upload-fail.png)

There are some related features to auto-upload like "conflict resolve" (this reminds me on git) and choosing the default resolve strategy and I thought these features are advanced, but the more I use the app, the more I realise that these features are introduced to patch the unstable upload and transfer responsability to user. This is the result:

![Multiple uploads of the same file]({{ site.url }}/assets/img/posts/2020/2020-06-17-nextcloud-duplicated-files.png)

I can only guess what happened in the background. This is the result of file conflict resolve where `(2)` is added to the file name. And it happened 8 times in this case. This is almost always the case with large video files.

Having this feature as unstable as now makes the whole product almost unusable. I want to have photos from my and my wife's mobile device uploaded to Nextcloud. At this point I'm not sure if some photos are lost, skipped or duplicated.  
Truth to be said, as unstable as is, eventually it uploads the content to server, but it's definitelly something I can't rely on.

### Browsing photos

You can browse all files from mobile app. It's nice to have this feature and somehow its assumed to have it, however, it's quite slow and UX/UI should be improved.  
By opening the photos directory you'll be presented with empty thumbnails for at least 10-15 seconds. And yes, each photo has a share button on it...

![Nextcloud mobile photo browsing]({{ site.url }}/assets/img/posts/2020/2020-06-17-nextcloud-mobile-browse-photos.jpg){: width="200px"}

### Disclaimer

I am very grateful for all the hard work developers are putting into this application, it is built in free time and I am aware that it's community driven product. That doesn't justify the lower quality of the product though. It has to be tightened up a bit and it'll work like a charm.  
I get the feeling that there is a lack of proper QA. It must not happen that main use-case gets shipped and failing for everyone.

After I have installed the application, Auto-Upload wasn't working at all. By reading the reviews on the PlayStore I figured that developers wrote _"if you want this feature to work, install beta version"_. That's not how its done people.

There is a huge potential in this product and, as other users in the reviews said, _Not working Auto-Upload is a deal breaker for me, I'm gonna search for better product_ and I think the same. You see, this one feature makes people ditch whole Nextcloud product and that's the problem.

## Synchronizing Calendar and Contacts

As mentioned, Calendar and Contacts can be synchronized through CardDAV/CalDAV. To have it working on Android, we have to install [DAVx5](https://play.google.com/store/apps/details?id=at.bitfire.davdroid). It costs 1$ and it's worth much more.

In order to login to nextcloud with this application, we have to generate an application password on Nextcloud. This is a must as DAVx5 app has to authenticate without 2FA involvement. With application password, we'll proceed to create an account on DAVx5 for our Nextcloud instance and it'll work by itself. To avoid confusion, this is just a local in-app account.

![DAVx5 application]({{ site.url }}/assets/img/posts/2020/2020-06-17-davx5.png)

Mindblown moment for me was when I figured out that you can use it to connect native Calendar app on your mobile device (in my case Samsung Calendar) and create events in native Calendar app, to have it synchronized to Nextcloud eventually. 

Its UI could be improved a bit, however as this application is made to work in the background, without much interraction, it's not a big issue.

## Final word

Nextcloud is definitelly a solution to use in private environment. Its web app is awesome. Mobile app needs more work to stabilize. Nextcloud configuration and optimization could be improved overall.

From personal perspective, Auto-Upload feature works but it is not reliable and I'm looking to other ways of syncing mobile photos to Nextcloud. Using script and cron it to pull Google Photos might be one of the solutions (as Google Photos backup works reliably anyway), but then I don't need the Nextcloud at all and I'm stick to Google service which I'm trying to find alternative for in the first place.  
If I ditch Nextcloud, I'm losing Calendar and Tasks features which I find as nice replacement for Google Calendar and Keep Notes. 

Do you have similar experience with Nextcloud or you'd like to share a solution for the issues I've mentioned above. Please, tell me about it.
