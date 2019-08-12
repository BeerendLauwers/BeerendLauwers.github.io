# Unraid

I've recently completed a build of a four-GPU Unraid box designed for quick, zero-setup LAN parties where each GPU was given to a Windows 10 VM along with some USB peripherals, which meant that four people could play on a single machine.
Essentially, [Linus Tech Tips' "7 gamers, 1 PC"](https://www.youtube.com/watch?v=LXOaCkbt4lI) project reduced in cost and scope (and with increased stability).
This was definitely not a quick project and I encountered many roadblocks along the way,
so I'd like to document them here.

## Hardware

At first, I wanted to have a rack-mounted solution, because in the original vision, each Windows 10 VM would have its own GPU with a single dummy HDMI dongle inserted in it.
These dongle inform the GPU they're actually a connection to a high-resolution screen and one is necessary to activate most GPUs.

Gaming would be done via streaming, either via a remote desktop solution like [Parsec](https://ui.parsecgaming.com/) (although that didn't exist when I started the project) or Steam in-home streaming.

In the worst-case scenario, I could always attach a monitor, mouse and keyboard for each VM and use it like that.
(Audio is passed via the HDMI/DP cable, so headphones plugged into the monitor work fine.)

### Initial revision

So, the first hardware revision was this:

| Part        | Item                                                        |
|-------------|-------------------------------------------------------------|
| Motherboard | Z10PE-D8 WS                                                 |
| Case        | Chenbro RM41300-FS81 8 Slot Gpu Version                     |
| PSU         | Be Quiet Dark Rock Pro 11 (1200w)                           |
| CPU         | Intel Xeon E5 2683 V3 (14 core, burst 3.0GHz)               |
| Cooler      | Noctua NH-U9DX i4                                           |
| RAM         | Kingston ValueRAM DDR4 (KVR21R15D4/32) (128GB)              |
| GPU 1       | Asus STRIX-RX480-O8G-GAMING                                 |
| GPU 2       | Asus STRIX-RX480-O8G-GAMING                                 |
| GPU 3       | Sapphire Pulse Radeon RX 580 8GD5                           |
| GPU 4       | Sapphire Pulse Radeon RX 580 8GD5                           |
| Storage     | todo                                                        |
| Storage     | Samsung 850 EVO 250 GB x4                                   |
| Storage     | Sedna – Mounting Adapter for 2.5in HDD/SSD to Hard Disk Bay |
| Fans        | Scythe Slip Stream Sl Case Fan 2,000 rpm x2                 |
| Fans        | Lian Li BZ-502B Front Cooling Kit                           |

**Motherboard:** Finding a good motherboard at a decent price point was hard - unless you went for the *really** expensive boards, it appeared that stability and QA issues were rampant on some of the earlier 2011-3 boards.
In the end, I went for the board Linus used in his video.

**Case:** Chenbro makes good cases for a great price point. 
It was also one of the few cases that could handle the E-ATX format of the Z10PE-D8 WS.
This particular case also has two fan slots at the top blowing directly on the GPU slots.

**PSU:** *Be Quiet!** has always been a favourite of mine for PSUs and fans.
They're high quality and not too expensive and very quiet indeed.

**CPU:** There's no particular reason I got this CPU other than that it had sufficient cores to distribute among the VMs, and that the OEM version was cheap on eBay at the time.

**Cooler:** Noctua is also a high-quality brand that makes quiet coolers.
This one could handle the high TDP of the CPU *and* fit in the case (but just barely).

**RAM:** Unraid recommends ECC because it's the only thing preventing bit rot, as it cannot make use of `btrfs**' RAID-like capabilities to correct bit rot, but only detect it when it occurs.
Unfortunately, RAM was quite expensive at the time, and because I wasn't going to actually use Unraid for its file storage, I went with this.

**GPU 1 and 2:** At the time, these were the fastest GPUs on the market for under €250 (the GPU mining wars had just begun).
I purchased them via Amazon Warehouse Deals, although I did have to return two of them because they were genuinely faulty.

**GPU 3 and 4:** A few months later, I got two of these for a very fair price, so in they went.
I later did return one of them after it turned out that four GPUs in this case wasn't going to work.

**Storage:** I had one Crucial SSD lying around that I used as the main parity disk and got four Samsung 850 EVOs for the VMs.

**Fans:** There was already a fan at the front of the case, but I needed very slim fans to go over the GPUs,
so I got these Scythe Slip Stream S1 fans that are very slim indeed.

I also had a Lian Li BZ-502B kit that takes up three 5.25" slots that went in the front of the case.

### First roadblocks

- The CPU cooler I had bought barely fit in the 4U case, so I had to ditch one of the slim fans.

- During testing of the GPUs, I discovered that half of the PCIe slots of that motherboard aren't even available unless you have *two* CPUs sitting in there. 
So, I had to buy another CPU and cooler. 
The CPU had to be the same, so I ended up with a ridiculous 28-core monster of a machine already.

- With all of the GPUs plugged in to test them and all the SSD put in the cage, there were a *lot* of cables strewn around.
I was afraid of cooling issues if I was going to have four two-slot GPUs.

- Unraid was giving me trouble with the Windows VMs - after booting, a VM would be able to access its GPU, but only once.
I did not know the source of the problem at the time, but it made starting a VM (and it working) very unreliable, even seemingly random.
This was the main reason I started thinking of moving to another case that could be more easily moved about and used with directly-attached peripherals.

- The Chenbro case was awkward to move about and it sometimes suffered from random issues like RAM getting unseated. 
This may have been a one-off event, but those are the kinds of things I definitely did not want to spend time on if I took this with me to a LAN event.

### Second hardware revision

So, time for a new case:

| Part        | Item                                                        |
|-------------|-------------------------------------------------------------|
| Case        | Corsair Carbide Series Air 540                              |

Unfortunately, the E-ATX motherboard I had bought didn't actually fit inside the case.
Apparently, E-ATX motherboard specs can vary wildly, and mine was one of the largest around.
This meant I would have had more than a third of the board just hanging freely.
If I remember correctly, I could only put one screw at the top of the board, 
all of them on the left side, and a few at the bottom, with none in the center or the right side.

Oh well, off it went to Amazon again.

### Third hardware revision

| Part        | Item                                                        |
|-------------|-------------------------------------------------------------|
| Case        | Cooler Master MasterCase H500P                              |
| Fans        | Noctua NF-A14 x6                                            |
| GPU         | ASUS Dual RX580 8GB                                         |
| Cables      | Noctua NA-SEC1 Chromax                                      |

Here, the motherboard fit after I removed one of the cable gutters.
I still couldn't populate all the screw holes, 
but at least I could put a few on all sides and one in the center, 
so I called that a success and continued onwards.

This case had two 200mm RGB fans at the front and space for three 140mm fans at the top, 
so I replaced the 200mm fans with six Noctua NF-A14 140mm fans that can pump in a whole bunch of air.

The cable gutters were also very useful to manage cables and optimize air flow.

### Second set of roadblocks

- After installing the fourth GPU, the system would no longer boot, instead displaying a "Insufficient PCI Resources Detected" message.
I thought I was well and truly screwed, but apparently it's just a [setting in the BIOS](https://www.supermicro.com/support/faqs/faq.cfm?faq=20284) for most motherboards. Mine was called "Above 4G Decoding" as well, which I enabled.

- The VM issues persisted, but luckily I found the [root cause through this post](https://forum.level1techs.com/t/linux-host-windows-guest-gpu-passthrough-reinitialization-fix/121097): hibernation and Windows Update restarts were causing problems.
So, I disabled hibernation.
Windows updates are relatively rare, and I can copy over the VM images if it really takes some times for one of them.
I'll probably add the script the original poster mentioned for ease of use.

### Fourth hardware revision

| Part        | Item                                                        |
|-------------|-------------------------------------------------------------|
| GPU         | Sapphire GPro 8200 x2                                       |

The third GPU's exhaust port at the back was blocked off, and there wasn't a lot of space beneath it to get rid of hot air, so I got a Sapphire GPro 8200 to replace it with. 
These cards aren't the strongest, but they're 1-slot blower cards. 
The added benefit was that I could use the available PCIe slot for the PCIe extension cable without blocking the fan of a card, because the Sapphire GPro 8200 only has one fan at the back end of the card.

I later added another card of the same type, replacing one of the STRIX cards.
This meant that there was always at least one PCI-e slot of space available for air dumped by the open-air GPUs.

### Third roadblock

The VMs did not seem to come up reliably when using the HDMI dummy dongles.
When they did come up, as soon as I started streaming the screen, be it with Teamviewer, NoMachine or Parsec, the VM froze.

Booting up a VM with an actual screen connected to it did not suffer from the VM freezing when streaming the screen.

Thinking I could fool the system, I booted up a VM with a real screen connected to it, then swapped it out for a HDMI dongle, but no dice: the VM just froze once more.
This headless setup might have worked with different HDMI dongles, but I was kind of tired of working around the problem, so I went the back-up route: hooking up screens and peripherals to the VMs directly and get it to work like that.

## Unraid configuration

Let's talk about the Unraid configuration.

### VM storage choices

I used the "Unassigned Devices" plugin that allowed me to manage drives that are outside of the Unraid array.

Each drive would hold a `.vdisk` image of a single VM. 
I went this route to make it easier to make backups of VMs and put them back in a single spot.

Additionally, this would allow me to set up a single VM with the programs and games I needed, and then copy over the VM image to the other drives.
(This did not always work out. VMs with similar or identical GPUs survived the process, but the VMs that I assigned to the Sapphire GPro 8200's did not like it one bit, crashing or refusing to boot. I'd say if you're using nearly-identical GPUs for all of your VMs, it's definitely worth a shot.)

Unraid needs a single drive to be happy, and I thought I could put common shared files there.
That's what the OCZ Saber is for.
I'll probably replace it with a cheap 2TB SSD that I'll use to save the VM image backups on, because `rsync`ing a 250GB file over a gigabit Ethernet cable still takes quite a while.

Here's the overview:

![Unraid overview](/images/unraid/unraid-overview.PNG)

### VM configuration

Each PC gets 8 CPUs, 16 gigs of RAM, a dedicated GPU and a large `vdisk.img` via VirtIO on a dedicated SSD.

VMs seemed to be stable with an OVMF BIOS, so that's what I went with.
Q35 is needed because it's a modern emulation of a chipset that properly supports PCI-E out of the box (as well as a whole other host of improvements over I440FX).

As you'll be able to see in the screenshots, I played fast and loose with copying over the `vdisk.img` files to other drives, hence the weirdly nested `PC2-UD/PC1-UD/PC2-UD/PC1-UD`. 
Never really cleaned up those paths, all that matters is that each VM gets a dedicated SSD.

For the VirtIO drivers, I just used the most recent ones available, which were `virtio-win-0.1.141`.

The OS for all of them is an unactivated trial version of Windows 10.
There's an "activate your Windows 10" message at the bottom, but that's easily ignored.
Of course, you can purchase licenses if you're going to use your VM often.

Because the VMs are all on the same network, it's possible to make a local LAN server and play without internet, as well.
(You might have to do some static IP assignment, of course.)

Here are the VM configs:

![Unraid PC1 Config](/images/unraid/unraid-pc1.PNG)

![Unraid PC2 Config](/images/unraid/unraid-pc2.PNG)

![Unraid PC3 Config](/images/unraid/unraid-pc3.PNG)

![Unraid PC4 Config](/images/unraid/unraid-pc4.PNG)

## Rubber hits the road

### Setup

I hooked up four screens and mouse/keyboard combo's to the machine.
I had to use a USB hub for the last machine's keyboard, but that gave no issues.

What I learned here is that it's important to number your PC ports with some tape or the like so you can easily replicate the setup,
as Unraid remembers the USB devices that were assigned to a particular VM.

Additionally, it's very useful to use different keyboards and mice to prevent conflicts with similar USB identifiers: two mice of the exact same make and model assigned to two different VMs will cause Unraid to complain when starting the second VM.

Again, labeling the peripherals and screens helps a lot with replicating a setup.

### Performance

I ran a GPU stress test (Valley Benchmark) on all four VMs at 1080p to test system stability.
With OpenHardwareMonitor, I checked the GPU temperatures, which stayed nicely below 60 degrees Celsius.
After leaving it running for an hour or so, I felt comfortable to put it to use.

### Gaming

After inviting some friends for a game night, I set up the configuration again and added four headsets that were connected via the monitors.
Luckily, no latency or crackling was experienced.

Performance was admirable: we were able to play a few games at 1080p that took some slight grunt, like Alien Swarm: Reactive Drop, Warhammer End Times - Vermintide and Zombie Army Trilogy.
(Unfortunately, Earth Defense Force 4.1 refused to run because I had apparently used a Windows 10 Pro ***N*** image, which appears to be missing a bunch of stuff EDF 4.1 depends on.)

### Tragedy strikes

Unbeknownst to me, all four VMs were downloading Windows 10 updates.
Suddenly, all of them unexpectedly shut down to update at the exact same time, which was quite a sight to see.

Unfortunately, boot errors showed up on all three of them, and attempting a system restore did not help.

Luckily, a backup plan had been prepared where we could play 4-player couch co-op games with game controllers on another computer.

## Getting Windows 10 updates to work on the VMs

After searching around online, the trick turned out to be twofold:

- Remove the GPU from the VM config, defaulting back to VNC Remote.
- Set each VM to CPU core 0.
- One by one, let the VM perform the Windows update and power it down again.

One of the VMs turned out to be damaged beyond repair, so that one will be replaced with a backup (or image copy of an updated VM).

## Conclusion

All in all, I would call this multi-year experiment a success.
System stability is great, game performance is admirable, and the ability to swap out and copy over VM images makes setting up new games a lot less of a hassle.

If you have any questions or suggestions, feel free to open up an issue for this blog over at Github: https://github.com/beerendlauwers/beerendlauwers.github.io
