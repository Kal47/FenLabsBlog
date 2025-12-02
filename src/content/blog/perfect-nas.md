---
title: "Building My Perfect Home NAS"
description: "From Problem to Conclusion"
pubDate: "Dec 2 2025"
heroImage: "/blog-placeholder-3.jpg"
---

*Please Note: This is a first draft published for feedback. Apologizes for any grammatical and especially spelling mistakes*

# The Problem!

Growing my homelab and wanting to run more and more containers, VM's, as well as wanting to use and learn other technology I simply outgrew the OS I was using, Unraid. Don't get me wrong I really do like Unraid, I have used it for years on a few different builds, but its VM and especially the Docker management do not do, or is very hard to do, some of the projects and setups I wanted.

Originally I installed Proxmox on my it and passed through its USB drive and the entire SATA controller. That worked amazingly with no issues to speak of. With this setup I could run all kinds of VM's and experiment with LXC's. 

The problems started when I wanted to pass through the GPU to a VM. This required lots of troubleshooting and worst of all, lots of rebooting the server. Unraid does not like to be rebooted. It like to make sure all the files are still there and in good shape often especially if it was unexpectedly shutdown, which is fantastic, but when you are rebooting constantly it gets really annoying waiting a few hours for it to check the array. I started leaving it off of auto start on boot while working on the GPU pass through, but then I can't get to my data. Even worse if I really mess something up and Proxmox fails to boot, this is a lab after all. Labs are for screwing up and learning from your mistakes.

# The Solution!

So it was decided to build a dedicated NAS machine! 
Requirements:
- Low on power usge for 24x7 operation
- Lots of RAM. I decided I wanted to use ZFS and I wanted to make the most out of ARC.
- ECC RAM. Its a file server, it should have ECC.
- 10G networking
- Some PCIE lanes for NVME drives, HBA's, or other interesting devices 
- On board SATA or SAS would be a plus
- Not break the bank

# The Quest to build the perfect home NAS!

Well perhaps not "perfect". But for me it is. 
Also include in that hours of researching various file system pros and cons, various NAS OS and features.
Bellow is what I came up with.

## Software

This one was difficult. I was already using Unraid on my old server which was a Lenovo P520 workstation. 

As stated before I wanted to run ZFS. The natural option would be to use TrueNAS. I love free software. I went back and forth on this but decided to stay with Unraid for now. In the future I may do something else with it and move my Unraid license to a backup NAS as Unraid does not care about different HDD sizes as much.

### The Case to Keep Unraid
1. Unraid has decent VM and Docker support that I found works well. That cannot be said with TrueNAS. I have read stories of changes breaking services on TrueNAS often. I still want to run some small services on it that touch the storage a lot.
2. Unraid allows multiple different size drives to be put in a the array and just work, something TrueNas does not do well.
3. I know how to use Unraid well, I am confident I can make a good working NAS with it.
4. Unraid supports ZFS now! Not as flushed out as TrueNAS but I still always have the terminal.

### The Case Against Unraid
1. With Unraid turning on and off the array is required to make almost any changes. This includes networking, Docker, and VM settings. Keeping good up time has been difficult with this. With TureNAS it just keeps going. 
2. With TrueNAS it can connect and duplicate with other devices running TrueNAS. This can kinda be done with Unraid, but I am on a tight budget and do not want to buy another license.

### Decission
I decided to keep Unraid. With its ZFS support as well as its array I created a new ZFS array from the new 6TB drives and kept the 10TB's in the Unraid array. There where also services already running on Unraid which made the move easy. At some point in the future when I may switch to TrueNAS on this NAS and put Unraid on a backup NAS with all of the spare HDD laying around.

## The Hardware 

This project involved hours of researching along with hours and hours scrolling Ebay for used servers and server motherboards. Initially I had my eye on some SuperMicro X10 generation Xeon D embedded systems. They where a little more then I wanted to spend, but they had all the requirements I required. Looking at other brands I found some AsRock boards quite a bit cheaper. This is when I ran across the Datto S4 systems. For about $250 you can get 32GB-48GB of ram along with a mATX board with every feature I listed including 8 DIMM slots and support for a massive amount of memory! That was also cheaper then the previous generation Xeon D boards from SuperMicro. Interestingly enough the motherboard by itself on those Datto NAS's where going for about $500. So, I decided to get one!

### CPU & Motherboard: $220 (for entire Datto S4P6 system)
Datto S4P2143 with a Intel Xeon D-2143IT CPU 
- 8 Core, 16 Thread
- 2.2Ghz Base, 3.00 GHz Turbo
- 65W TDP
- 48GB of 2400 DDR4 ECC RDIMMS included

The motherboard is essentially a re-branded AsRock Rack D2143D8UM, you can even flash the BMC and BIOS with the firmware from the Asrock board and get rid of all the Datto stuff! I bought it in the original OEM configuration in its 1U chassis, but with the BOIS and BMC already flashed from a fellow homelab'er. 

It features:
- Low power ish at 65W. (I have yet to test the real power draw)
- 8 DIMM slots with up to 256GB of DDR4 ECC RDIMMS
- 12 SATA ports
- 16x 3.0 PCIe lanes total split over 2 slots (One 16x or two 8x, also supports bifurcation)
- One M.2 PICe 3.0 x4
- Dual 10GB networking from a changeable mezzanine card. There is a 25GB card that works for this.

### Storage 

From the old NAS
- 1 New  10TB WD Red Pro
- 1 Used 10TB Seagate Exos
- 3 Used 256GB SATA SSD's for Unraid Cache

"New" storage
- $28 4 Used 6TB HGST SAS Drives
- $35 1 Used 512GB M.2 NVME for ZFS L2ARC
- $22 1 Used 8GB RMS-200 NVRAM Accelerator for ZFS SLOG

I have to stop here and talk about this RMS-200. This is such an interesting little device. What it is some DDR3 RAM backed by a big bank of capacitors and a little bit of flash. Normal operation it acts like a normal NVME drive, but at RAM speeds. On power failure it writes all data in its RAM to flash. It also has an advantage of being able to be written to an almost unlimited amount of times. The one I have was  172 PB, yes petabytes, 172,000 TB and still works fine! (except the fan was dead...) This is come in handy when doing sync writes with ZFS which will eat through normal SSD's.

### Case/Chassis

The motherboard came in an entire 1U system, but there where some issues.

1. It was loud! Those little 40mm fans even with some Noctua low noise adapters would still ramp up very high under load. 
2. Drive space. I could only put 4 LFF(3.5") drives in it and 3 SFF(2.5") drives
3. It had a 90 degree PCIe riser that covered the other slot. I could only put one x16 or smaller in it.

What I decided to do was remove the board and put it inside an desktop case. Its old Cooler Master case that originally had a Core 2 Dual system built into it. An odd choice? Hear me out, it has five 5.25" bays on the front. That is a lot of space for hot swap bays.


One problem I ran into is that the IO shield did not fit on the normal PC case. A bigger issue is that the CPU cooler was made for a 1U chassis with the airflow of the chassis itself in mind. So, I built bracket that slides on top of the little cooler that holds 80mm fan. 

Unfortunately, I must have miss typed a dimension as the bracket was about 10mm too short. I found this out after I already moved everything to the new case. I did not have time to modify and print another so I had to make do with that I had and got it attached. Lessons learned. 


## Future 

I do plan on revising the fan bracket for the CPU using a 92mm fan. At that point I will publish the model on Printables.com.

I have an idea to make vertical hot swap SATA SSD that its in two 5.25" bays. I found some SATA to SAS adapters that conveniently have screw holes in them! With some clever CAD work I think I can make this.

I am still looking at old 2U super micro chassis. To perhaps put it in. The CSE-826 is a 2U, 12 bay hot swap LFF(3.5") 


# Conclusion

With this project nearing its close 

$220 for the server
$120 for the 4 6TB SAS HDD's
$37  for the 512 M.2 SSD

For the machines performance, running a Desktop Linux VM on it was a little rough. On the other hand the Docker containers I planned on keeping on it barely touched the CPU. I certainly have a lot of room to grow with this. 

## Sources

