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

## Unraid configuration