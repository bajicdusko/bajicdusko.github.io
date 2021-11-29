---
layout: "post"
title: "Moving away from Google services"
date: "2020-06-07 23:00"
categories: [cloud]
tags: [google, privacy, cloud]
description: "Danger of having locked Google account is real. I've started my migration from Google services and dispersing access to my data. This is how."
image: "assets/img/posts/2020/2020-06-06-google-services.png"
redirecturl: "https://www.crispy-engineering.com/moving-away-from-google-services/"
---

There isn't a person in the modern world which isn't affected with at least one of its services. Their services are great products and we're happy to use them, for FREE.

## The big Google

![Most popular Google services]({{ site.url }}/assets/img/posts/2020/2020-06-06-google-services.png)

As an Android developer I was naturally drawn to wast of services provided by Google: **Search**, **Gmail**, **Google Drive**, **Contacts**, **Chrome**, **Calendar**, **Keep Notes**, **YouTube**, **Android**, **Play Music**, **Docs**, **Spreadsheets**, **Maps**, **Photos**, **PlayStore**, **DNS**. And this is not just random list. I actually use these services on a daily basis.

## The Fear

This is a frightening list, all accessed through single, Google, account. And I realized, all of my memories are there, documents, contracts, email, my whole life and how I function. And then it hit me...

**Someone might stole my account!**  

I saw the access to my Google account as (we developers like to call it) "_single point of failure_".  
Using Two-Factor authentication, rotating high-complexity password 2-3 times a year, and having protected recovery email helps with it.  
For someone to log into my account, he should have access to my password manager (biometric protected), to copy account password, to bypass location login protection and actually log in, and at the end to use Authenticator to confirm authentication. I might argue now that its highly unlikely for someone to access my account from the outside.  
But what if my account is accessed from the inside ?

**What if I get locked out of my account, by Google itself?**

This happens as well. And there is no 2FA to protect you from it. It could happen as simply as bot restricting access to my account as it might appear to it that I broke some ToC. And just like that, my family photos, client android applications inaccessible, my music out of reach. Everything I have ever written.

To avoid such scenario, only option I have is to disperse services. Photos on one cloud provider, new email account with private paying service, different music streaming service and so on.

### Privacy factor

I've accepted that everyone's are collecting the data about us. So is the Google.  
I'm browsing the internet using Chrome. Searching for something using Google search engine. Opening sites via their DNS. Using Android. Through the sensors of my watch, I'm telling them how much I have slept last night and what's my current heart rate. Navigating Maps and discovering my GPS location. Discussing topics near microphone and getting the ads minutes after it.

These privacy issues kinda align with the idea of dispersing services and there is a chance of at least decentrelizing data they have about me.

## Steps into right direction

In the beginning of 2020 I've created a list of steps to migrate from Google services as painless as possible. Here it is:

1. **Avoid using their DNS** ✅  
It's easy to replace 8.8.8.8 with [OpenDNS](https://www.opendns.com/). Addresses not easily remembered, but who cares.
2. **Stop using Chrome** ✅  
Switch to other browsers. I've reviewed Brave and Firefox, sticked to Firefox. Never went back to Chrome. Desktop and mobile.  
3. **Change search engine** ✅  
All it takes is to set [DuckDuckGo](https://duckduckgo.com/) as default search engine in browser. Desktop and mobile as well. DuckDuckGo results are decent and getting better every day. I'm using it for a couple of months and I was forced to "Google" something specific on 2-3 occasions so far.
4. **PlayMusic** (read: **YouTube Music**) ✅  
We have many alternatives here and any will work. I'm sticked to YouTube Music because Bosnia  🇧🇦 ...

    ![Spotify not available in your country]({{ site.url }}/assets/img/posts/2020/2020-06-06-spotify-not-available.png)

    YouTube Music costs 6.99$ per month and considering the amount of my daily usage, the price is acceptable. One of my first music streaming subscriptions was [Deezer](https://www.deezer.com/us/offers). I was quite dissapointed with its algorithm back then, and quit the subscription. I guess they've improved their implementation and suggestion algorithm in the meantime.

    So far it was easy. We're moving to more complex parts now.

5. **Drive**, **Photos** 🔄  
Finding alternative to Google Drive deserves its own post. Currently I'm paying Google One 200GB package (29.99$ annually) and it's full already.

    ![Google Drive storage almost full]({{ site.url }}/assets/img/posts/2020/2020-06-07-drive-running-out-of-storage.png)

    Paying 2TB package (99.99$ annually) is unacceptable for me. My current approach is [Nextcloud on a private VPS]({{ site.url }}/posts/2020/06/nextcloud-alternative-to-google-services).  
    Some other alternatives are OneDrive and Dropbox, but no one can't beat 200GB package I'm using. It's worth mentioning that [Office 365 Personal (1TB)](https://onedrive.live.com/about/en-us/plans/) has acceptable price of 69.99$ annually as you get Office applications as well. However, if we scale it to 2TB it's more expensive than Google One.

6. **Android** ❌  
There isn't much to do here. Whatever I use, they'll collect data from my mobile anyway. Switching to Apple or some chinese manufacturer makes sense only to disperse data. So my next phone will be either iPhone or Xiaomi :).
7. **Docs**, **Spreadsheet**, **Calendar**, **Contacts** and **Notes** are usually part of email service or in my case, Nextcloud solution. 🔄
8. **YouTube** and **Maps** have no alternative, therefore I'll stick with these. ❎  
9. **Gmail** ❌  
I'll never be able to get rid of Gmail as I need it to access to PlayStore at least. Setting up different email as your primary communication hub after years of Gmail usage wont be straightforward. [FastMail](https://www.fastmail.com/) is quite popular, starting with 30$ annually. [KolabNOW](https://kolabnow.com/) have good reviews as well, and it comes with 55$ annual package. I haven't tried any of these yet, as I haven't reviewed much of these services.
Migration from Gmail will be gradual and slow.

It's a work in progress and it'll take at least a year to migrate to different services and get used to different processes. I find storage and documents management as most complicated service to migrate. That's why I'm putting a lot of effort to try out different approaches. Setting up Nextcloud as alternative to Google Drive is my first approach..  
Your solutions are very welcome in the comment section below.
