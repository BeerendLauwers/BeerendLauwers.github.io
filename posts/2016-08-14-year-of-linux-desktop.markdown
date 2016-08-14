---
title: 2016 - Year of the Linux desktop?
---

## The prelude

Developing for [Hask Anything](http://haskanything.com/) on Windows has become ever more painful: dependencies on `unix`-y stuff combined with [MSYS](http://www.mingw.org/wiki/MSYS) tended to result in weirdly broken packages and programs, often at runtime. 

I tended to power through these and try to resolve the problem and document it on this site, but with Hask Anything having launched, I could no longer find the time to chase these exotic issues: I had features to write, horribly overseen bugs[^bugs] to squash!

So, I got another SSD dedicated for Ubuntu 16.04 and used Rufus to get a UEFI-ready USB stick ready.

## Mo' Nvidia, Mo' problems

Installing Linux, for most people, is *probably* relatively effortless: plug in a USB stick, select the boot option after restarting, and click through the installation wizard.

I, however, had a pretty perfect storm of hardware for it to go wrong:

1. Windows 10 was already installed. In UEFI mode.
2. On a Skylake chipset.
3. With an Nvidia graphics card.

The first two had me going on a wild goose chase when trying to solve my problems, but they probably solely originated from the fact that I had a graphics card from Nvidia.

## Boot #1

I plugged in the USB stick and selected it in the boot options. The Ubuntu background loaded after a few minutes, but it was empty.
Well, there was a cursor, but that was it. No right-click menu, no sidebar, nothing.

After some time, the following message popped up:

> Ubuntu 16.04 has experienced an internal error.

Oh dear. Opening up the details, I deciphered that the problem lied with `compiz`, whatever that was.
Some googling told me that [Unity wasn't loading](http://askubuntu.com/questions/17381/unity-doesnt-load-no-launcher-no-dash-appears), and I had to do a bunch of stuff on the command line.

Only problem was, I *didn't have a command line*.

So, off I went to figure out how I could get to a command line.
Which led me to [this question](http://askubuntu.com/questions/162535/why-does-switching-to-the-tty-give-me-a-blank-screen).

Small excerpt of the proposed solution:

> Open the terminal with Ctrl+Alt+T.
> 
> Paste the below, and enter your password when asked:
>
>     sudo sed -i -e 's/#GRUB_TERMINAL/GRUB_TERMINAL/g' /etc/default/grub

I couldn't call a terminal with that key combination, and CTRL+ALT+F1 gave me a black screen. It was clear that the Nvidia card was causing issues.

So, I took it out temporarily and installed Ubuntu with my display hooked up to the motherboard's output.
Success! I could see the Ubuntu desktop.

Back in went the card, and back in came the problems: only the Ubuntu background and a lone cursor. Oh, and the terminal screen was still black.

## Boot #5

Someone suggested an alternative: editing Grub's boot options to include `nomodeset`.

I tried to get into Ubuntu's booting options.
How to do that? [No one knows for certain.](http://askubuntu.com/questions/16042/how-to-get-to-the-grub-menu-at-boot-time)

The Escape key didn't work, and neither did Shift. Mashing space did.

Then, it was just a matter or editing the configuration for one of the boot options and booting with the new config.

## Boot #9

The blank background was still there[^taunt], but at least CTRL+ALT+F1 got me a *visible* command line.
It was time to [install the proper drivers](http://askubuntu.com/questions/481414/install-nvidia-driver-instead-nouveau).

This process involved blacklisting a bunch of drivers and then purging all the nvidia-related drivers on there, stopping the display manager (`lightdm` in my case), and installing the Nvidia drivers you downloaded earlier.

Wait. *Crap*. I didn't have a working internet connection on Ubuntu yet, I needed proprietary drivers for the Wifi dongle I was using.

Ok, so *obviously*, I couldn't continue this way. An accidental reboot later, I had lost the visible command line once more, even with `nomodeset`. 

*Shit*.

## Boot #10

What options remained? Pretty much none.
I scrolled through the tabs I had open:

> Open the terminal with Ctrl+Alt+T.
> 
> Paste the below, and enter your password when asked:
>
>     sudo sed -i -e 's/#GRUB_TERMINAL/GRUB_TERMINAL/g' /etc/default/grub

Hmmm... Could I just enter this thing blindly?
Doesn't hurt to try. Blindly enter username, password, that command and password again, nothing to it. `reboot` for good measure.

## Boot #11

The `reboot` command worked, and I had a visible command line again. Great!

I downloaded the Nvidia drivers from their website, put them on a USB stick, mounted it manually and ran the installer.

## Boot #12

**Success!** The Ubuntu desktop greeted me in *glorious* 4K. I set both screens to 2K for the moment and went to bed.

## Boot #13

The next day, I booted it up again and the resolution had reverted to 4K again. No biggie, just set it up again. Time to install Git, Stack, Haskell and all that.

Ah, no internet. Forgot. Just needed to get a driver for this dongle and install it.

Unfortunately, [Google says No](https://ubuntuforums.org/showthread.php?t=2258715). 

```bash
sudo apt-get install build-essential git
git clone https://github.com/gnab/rtl8812au.git
cd rtl8812au
make
sudo make install
sudo modprobe 8812au

```

The git repo, okay, I can get that via USB. But the `build-essential` stuff? How do I get that?
Almighty Google tells me, [like so](http://askubuntu.com/questions/146425/how-can-i-install-and-download-drivers-without-internet):

> You can manually download any ubuntu package from http://packages.ubuntu.com, copy them to the linux drive, then use dpkg to install them. 

Okay, great. Let's download it and put it on the USB stick.
The Ubuntu computer dimmed its screen, so let's wake it up aaaand... the resolution has gone to something like 1024x768.

No, not exactly. The resolution is 4K. *Again*. 
But it's only showing a small part of the screen. 
The cursor still thinks we're 4K, so I have to play "find where the cursor is clicking" for a minute or two to get to the display screen and set it to 2K again.

After pressing "Apply", the screen flickers as usual... and just displays "no input". No. nonono. I am done. *Done*.

## Boot #14

I booted up *Windows*, installed Ubuntu 16.04 in a virtual machine, and called it a day. (I'm writing this post on it as we speak. Love it.)

## Conclusion

Every year, more setups become supported by Linux. But every year, small things like this keep popping up and preventing people from even *getting to the Ubuntu installer*.

I mean, last time I tried to install Ubuntu on a laptop, it kept segfaulting, so I *guess* this is an improvement.
Still, it's surprising how much I had to struggle with it just to get it up and running.

So, to answer the question posed in the title: "Maybe next year".
 

[^bugs]: *Seriously*, how did I forget to point the web content submission interface [*to the real git repository?*](https://github.com/beerendlauwers/HaskAnything/commit/9a6ac37683830d799dd771293609d19e6171a3e8)

[^taunt]: *Taunting me.*
