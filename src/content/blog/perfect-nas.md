---
title: "Building My Perfect Home NAS"
description: "From Problem to Conclusion"
pubDate: "Dec 2 2025"
heroImage: "/blog-placeholder-3.jpg"
---

*Please Note: This is a first draft published for feedback. Apologizes for any grammatical and/or spelling mistakes* V2

# The Problem!


While trying to do more with VM's and Docker in my home lab I kept running into issues with Unraid. Don't get me wrong I really do like Unraid. I have used it for years on several diffident servers. I have done a lot with it's Docker subsystem, but I kept running into issues with VM's and Docker that seamed more of a limitation of Unraid then anything else. I needed a more capable hypervisor.

Originally I installed Proxmox on my server and passed through it's USB drive and the entire SATA controller on the motherboard. It worked great with no issues to speak of. With this setup I could run all kinds of VM's and experiment with LXC's as well as have my files.

The problems started when I wanted to pass through the GPU to a VM. This was not a smooth posses. It required lots of research and troubleshooting, but worst of all, lots of rebooting the entire server. Unraid does not like to be rebooted a lot. It does it job like it should and makes sure all the files are still there and in good shape often, especially if it was unexpectedly shutdown. This is great normally, but when you are rebooting constantly after you locked up a VM by followed a decade old guide, it gets really annoying waiting a few hours for it to check the entire array.

I started leaving Unraid to not auto start on boot while working on the GPU pass through issues. This has the added problem of I can't get to my data. Even worse if I really mess something up and Proxmox fails to boot. This is a lab after all, we break things to learn how to fix it.

# The Solution!

And so it was then decided to build a dedicated NAS! I can break Proxmox all I want and the other computer will sit there and run. (Assuming I don't get the itch to push buttons on it)

Here are the requirements I set for the new NAS:
- Low on power usage for 24x7 operation
- Lots of RAM. I decided I wanted to use ZFS and I wanted to make the most out of ARC
- ECC RAM. It's a file server, it should have ECC
- 10Gb networking o board to free up extra PCIe slots
- SATA or SAS on board would be a plus to free up extra PCIe slots
- Some PCIe lanes for NVME drives, an HBA, or other devices
- Not break the bank
- Standard ATX like form factor

Working on getting the right hardware at not ridiculous prices as well as how I would split the responsibilities of the two servers took a good amount of time. This includes hours of researching various file system pros and cons, various NAS OS and features.
Bellow is what I came up with.

## Software

This one was difficult. I was already using Unraid on my old server which was a Lenovo P520 workstation. 

As stated before I wanted to run ZFS. The natural option would be to use TrueNAS. I love free software. I went back and forth on this but decided to stay with Unraid for now. In the future I may do something else with it and move my Unraid license to a backup NAS as Unraid does not care about different HDD sizes as much.

### The Case to Keep Unraid
1. Unraid has decent VM and Docker support that I found works well. That cannot be said with TrueNAS. I have read stories of changes breaking services on TrueNAS often. I still want to run some small services on it that touch the storage a lot.
2. Unraid allows multiple different size drives to be put in a the array and just work, something TrueNas does not do well.
3. I know how to use Unraid well, I am confident I can make a good working NAS with it.
4. Unraid supports ZFS now! Not as flushed out as TrueNAS but I still always have the terminal.
5. Are services I already had running on Unraid which would make the move easy. 

### The Case Against Unraid
1. With Unraid turning on and off the array is required to make almost any changes. This includes networking, Docker, and VM settings. Keeping good up time has been difficult with this. With TureNAS it just keeps going. 
2. With TrueNAS it can connect and duplicate with other devices running TrueNAS. This can kinda be done with Unraid, but I am on a tight budget and do not want to buy another license.

### Decision
I decided to keep Unraid. With its ZFS support as well as all my data and the containers I already had set up, I didn't have to move much of anything. 
I created the new ZFS array from some "new" 6TB drives and kept the two 10TB's in the Unraid array. I am still learning ZFS, there is a possibility of me destroying all my data if I migrated. Until I am more confident, this array will hold backups of my computers as well as any movies and media. Things that if they vanish won't be a big deal to replace or is just a backup of a backup.

## The Hardware 

This project involved hours of researching along with hours and hours scrolling Ebay for used servers and server motherboards. Initially I had my eye on some SuperMicro X10 generation Xeon D embedded systems. They where a little more then I wanted to spend, but they had all the requirements I required. 

Looking at other brands I found some AsRock boards quite a bit cheaper. This is when I ran across the Datto S4 systems. For about $250 you can get 32GB-48GB of ram along with a mATX board with every feature I listed including 8 DIMM slots and support for a massive amount of memory! That was also cheaper then the previous generation Xeon D boards from SuperMicro. Interestingly enough the motherboard by itself on those Datto NAS's where going for about $500. So, I decided to get one!

### CPU & Motherboard: $220 (for entire Datto S4P6 system)
Datto S4P2143 with a Intel Xeon D-2143IT CPU 
- 8 Core, 16 Thread
- 2.2Ghz Base, 3.00 GHz Turbo
- 65W TDP
- 48GB of 2400 DDR4 ECC RDIMM included

The motherboard is essentially a re-branded AsRock Rack D2143D8UM, you can even flash the BMC and BIOS with the firmware from the AsRock board and get rid of all the Datto stuff! I bought it in the original OEM configuration in its 1U chassis, but with the BOIS and BMC already flashed from a fellow homelab'er. 

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

At some point in the future when I may switch to TrueNAS and put Unraid on another backup NAS with all of the spare HDD's I have laying around.

I do plan on revising the fan bracket for the CPU using a 92mm fan. At that point I will publish the model on Printables.com.

I have an idea to make vertical hot swap SATA SSD that its in two 5.25" bays. I found some SATA to SAS adapters that conveniently have screw holes in them! With some clever CAD work I think I can make this.

I am looking at old 2U super micro chassis to move it to, the CSE-826.
- 2U, 12 bay hot swap LFF(3.5") 
- supports up to an EATX motherboard
- It has 7 expatiation slots though those are half height. 
- I can get one used with redundant 920W "Quiet" PSU's and rails for about $250. 
- It would give me plenty of room to expand the storage. 
- I could also make 2.5" SSD mounts from the unused EATX and ATX standoffs for extra storage.
On the other hand...
- It could be really loud, I don't know. 
- I would be very long. Hard to fit in a smaller server rack. My current one is fine right now.

# Conclusion

Costs:

$220 for the server

$120 for the 4 6TB SAS HDD's

$37  for the 512 M.2 SSD

For the machines performance, running a Desktop Linux VM on it was a little rough. On the other hand the Docker containers I planned on keeping on it barely touched the CPU. I certainly have a lot of room to grow with this. 

ZFS with L2ARC and SLOG run great. I ran KDiskMark in a VM, mounting the disks with Virtiofs, really not the best test I know, but it gave me some sort of a result. Take this with a grain of salt...

- L2ARC really did not change that much. The array is empty so it wont get any hits. Ill do a real world benchmark against the Unraid array later
- SLOG almost tripled my write speed! Sequ 1M from 302 M/B to 958 M/B on the benchmark. 
- The Unraid array was 68 M/b on the same test but its read was almost 1000 Mb faster? Faster then the NVRAM drive by itself? There is 100% some stuff going on in the back end that is messing with results, which is a good thing TBH. It just shows I should do some real world tests.
- Just the NVRAM disk didn't score as much as I thought, scoreing the same as my BTRFS SATA SSD array on reads but did score 170 M/B faster in writes. 

More investigation on speeds will need to be done in the real world.

## Sources

