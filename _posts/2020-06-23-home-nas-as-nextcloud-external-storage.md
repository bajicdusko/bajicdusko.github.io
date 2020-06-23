---
layout: "post"
title: "Add home NAS as Nextcloud external storage"
date: "2020-06-23 23:30"
categories: [cloud]
tags: [nas, privacy, cloud, nextcloud]
description: "Keeping files and documents in NAS, located at home is a preferable way and privacy vise. Real advantage of such storage is achieved by attaching it to your Nextcloud instance, without exposing it to the internet."
image: "assets/img/posts/2020/2020-06-23-nextcloud-external-storage-cover.jpg"
---

In the introduction article [Nextcloud as alternative to Google services]({{ site.url }}/posts/2020/06/nextcloud-alternative-to-google-services) I've mentioned that my solution is almost private as my Nextcloud instance is hosted on VPS which is in theory physically accessible to super-admins in data center.  
That doesn't mean that I have to store my photos and sensitive documents on their storage though. I want to store those files on my NAS, which I have physical control of.

I've decided to somehow connect my NAS to Nextcloud instance "out-there". But how?

As explained in [Nextcloud honest review]({{ site.url }}/posts/2020/06/nextcloud-honest-review) one of the Nextcloud cool features is "External storage".

![Add Nextcloud external storage]({{ site.url }}/assets/img/posts/2020/2020-06-23-nextcloud-external-storage.jpg)

This means that we can connect different type storages for Nextcloud to read and write to. At first sight, none of these solutions work for me. My home address is behind CG-NAT (Carried Grade NAT) which means that my router is not accessible from the outside. Opening specific ports for SFTP or SMB means nothing to me.

I had two options at disposal:

1. Contacting my internet provider and asking to be moved to different IP address pool, to be outside NAT.
2. To utilize my already powered-on Raspberry 3 somehow.

You guess, I've decided to go with option 2, as it more challenging :). Truth to be said, having dynamic IP address, with publicly accessible router is still painful, as I'd have to synchronize my public address with Nextcloud somehow + I don't like having my router with publicly accessible SFTP or SMB ports.

## Raspberry to the rescue

My home storage is connected to Raspberry via USB anyway. To have it accessible from Nextcloud as well I had to solve a couple of challenges:

1. My home devices are behind dynamic IP address.
2. My IP address is behind NAT.
3. My storage is not directly accessible on network as it is attached to some other device (Raspberry)

Solution for these points is **VPN** connection where my VPS server will act as VPN server, while Raspberry (and it turns out, my mobile device as well) are VPN clients.

What do I gain with it?

- My VPS is on static IP address, with registered DNS record.
- Raspberry acting as VPN client connects to VPS and establish always-on connection.
- Data transfer is encrypted.
- No need to expose my router ports publicly.
- As of now, VPS and my home are practically in local network, which makes maintenance much easier and it has a potential of making my Nextcloud setup completely private by making it inaccessible from outside world and communicating through VPN only.

## VPN

Without going into details on how to setup VPN, I'll reference to awesome, stable, state of the art, open source product [Algo VPN](https://github.com/trailofbits/algo). If you're planning on setting up VPN on your infrastructure, search no more. This is the solution you need. Period.  
Installation is straight-forward and as a result you'll get config files for WireGuard VPN clients, MacOS IPSec configs, even QRCodes to scan with mobile devices with installed WireGuard client app.

On Raspberry, I've installed WireGuard client for Debian.

With keep-alive setup of 30 seconds, I've made sure that the connection is client triggered, always opened.

But this is just the first piece of the puzzle. Making the storage accessible from VPS server is next step. For that to happen, we'll use SMB protocol.

## Sharing NAS on network

- On Raspberry, mount USB device on boot.  
In `/etc/fstab` file, I've added line to auto-mount my USB device, like this:  
`UUID=5E98-D69A	/mnt/hdd	exfat	defaults,auto,users,rw,nofail	0	0`  
You'll notice, I've choosed the directory `/mnt/hdd`.

- Install Samba share server and share the mounted directory

    Setting up Samba server, and sharing mounted device requires a couple of steps.

1. First, install Samba server and setup one directory to share. Usually, that'll be `/home/myuser/sambashare`. You're setting this as part of `/etc/samba/smb.conf` file. Here is my example:
```
[sambashare]
    comment = Samba on Raspberry
    path = /home/pi/sambashare
    read only = no
    browsable = yes
    follow symlinks = yes
    wide links = yes
    unix extensions = no
```
2. Inside of this directory I've created directory called `hdd` and soft-linked it to mounted directory from step one.

![Sambashare USB hdd softlink]({{ site.url }}/assets/img/posts/2020/2020-06-23-sambashare-hdd-softlink.jpg)

## Attaching NAS as Nextcloud external storage via SMB protocol

Now, we have no more obstacles and we can proceed by adding external storage to Nextcloud. We'll choose, SMB protocol and fill the rest of required fields.

- Raspberry private IP address
- Necessary credentials
- Path to the directory on NAS we'd like to show as directory in Nextcloud.

I've decided to add two external storages, more preciselly, two directories from the same storage, as I want to limit access to the rest of NAS directories.

![Nextcloud external storages]({{ site.url }}/assets/img/posts/2020/2020-06-16-nextcloud-external-storage.jpg)

And this is how it looks from within the app:

![Nextcloud external storage as directories]({{ site.url }}/assets/img/posts/2020/2020-06-23-external-storage-as-directories.jpg)

## Mobile access

Nextcloud works in such way that external storage is shown as directory in the app. It's the same for mobile app. I've configured my Auto-Upload feature to synchronize Camera directory from the device with some directory inside `BBData` directory.

![Auto-Upload camera config]({{ site.url }}/assets/img/posts/2020/2020-06-23-auto-upload-camera-config.jpg)

And when I take a photo on my mobile, this is what happens:

- Auto-Upload feature will detect new photo in Camera directory
- It'll upload that photo into `BBData` directory on Nextcloud
- As `BBData` directory is SMB external storage, that photo is actually saved onto location shared by Samba server. Translated path would be `smb://10.19.49.2/sambashare/hdd/bb_data/Phone/Dusko/Camera/photo.jpg`
- Shared directory is on Raspberry located in my home, therefore, VPS server uploads the photo to my home Raspberry.
- As target path is deep inside shared directory, photo is further forwarded onto `/mnt/hdd/` directory, which is a mounted USB storage. (soft-link: `hdd -> /mnt/hdd`)
- Eventually, photo is stored on my NAS and hopefully won't be lost ever.

The point is, none of my files are located on VPS and disk-space Nextcloud takes on VPS is 3-4 GB's (thumbnails are 95% of this).

### Public access lockdown (VPN only)

Since I choosed to allow access to VPN from my mobile devices as well, by installing WireGuard app on mobile device, I'm easily connected to VPS through VPN connection from my mobile device, while the whole sync job works without extra configuration.

![Android WireGuard]({{ site.url }}/assets/img/posts/2020/2020-06-23-android-wireguard.jpg)

If there is a need, I can easily route current DNS record to private address assigned to VPS-VPN network interface. Everything will work without issues, as my devices are always using the same subdomain, but actually targeting the private/local IP addresses.  
Additionally I could allow access from specific devices only to VPS server, making it inaccessible from the outside. This might be as closest to "private cloud" solution I'm aiming for.
