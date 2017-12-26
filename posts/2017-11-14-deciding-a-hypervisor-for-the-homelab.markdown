---
title: Deciding upon a hypervisor for the home lab
---

I have a HPE DL120 Gen9 that I wanted to use for home services: 
Plex, audio streaming services, JIRA, and so on.

I wanted something low maintenance that I could just set up once
and then leave running.
I also wanted it to have Docker support to learn more about it.

## Proxmox

My first attempt was Proxmox, which is free and seemed to be
easy to use.

Unfortunately, I couldn't get it to install properly on the DL120; 
the installer failed to boot under UEFI, so I switched to "Legacy Mode" (AKA BIOS) 
and attempted to install it again.

This time, the installer showed up.
After installing, however, the server was unable to boot from the partition 
to which Proxmox had been installed. 
I tried to fix the bootloader and some other stuff, but I gave up after a few days.

## ESXI

HPE provides a customized image of the ESXI hypervisor that works out of the box.
No Docker support, sure, but I could just spin up a few barebones VMs and install Docker, right? 

Well, not quite.

ESXI 6.5 had just moved over to a web-based management UI. 
And it was slow as dogshit. (I only realized later there was a fat client for Windows.) 
The UI would time out periodically or become unresponsive.

But what was worse, was that VM installations tended to lock up. 
When that happened, I had to delete the VM entirely, as it no longer booted into the installer.

At first I thought this was just a problem in Ubuntu Server 16.04, 
but none of the fixes or suggestions seemed to work for me.

Then I discovered I was using outdated drivers and outdated ESXI components, 
so after a big driver update (thank you HPE SPP all-in-one packages, although you don't make them easy to find without a server subscription) 
and an update of ESXI, the UI was a lot more responsive and installs no longer locked up.

Unfortunately, after rebooting the server, many of the Ubuntu VMs started to lock up while running 
and become unresponsive. 

[Sebastian Foss's article](http://www.nxhut.com/2016/11/fix-slow-disk-performance-vmwahci.html) suggested that 
perhaps this performance degradation was caused by a very new AHCI driver introduced in ESXI 6.5. 
The solution: fall back to the old, battle-tested driver.

After doing that and rebooting, ESXI became entirely unstable for me, randomly locking up.

At that time, I had already spent several weeks spread over a year to get this thing off the ground. 
Not what I had in mind when I thought of "low maintenance".

## Unraid

Over time I had heard of Unraid, a paid and closed-source, user-friendly hypervisor aimed at 
getting up and running quickly. 
Originally, it was developed for non-RAID-based redundancy, 
but it has grown to be an excellent base for a homelab.

So, I took the plunge and downloaded Unraid on a USB stick, 
which contains the config file that is loaded into RAM along with the rest of the hypervisor on boot. 
Much to my surprise, it worked on the first time!

Both SSDs were detected correctly, and I could get to creating a few test VMs in only a few minutes.

While Unraid does has Docker support, it needs some XML to wrap a Dockerfile with to get it to work with Unraid's UI.
 So I set up a few VMs instead.

I haven't had any issues with it as of yet, and it's quite responsive. 
The only gripe I have with it is that the built-in VNC solution is very slow, 
and finding a decent alternative for a GUI turned out to be a lot more hassle. 

In the end I ended up choosing NoMachine, which is the only continuation of the next generation X protocol (NX). 
It's pretty performant compared to something like TigerVNC.

So in my opinion, Unraid's promise of low maintenance and ease of use has most certainly been fulfilled!
