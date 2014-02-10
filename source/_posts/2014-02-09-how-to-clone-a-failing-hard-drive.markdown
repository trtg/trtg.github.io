---
layout: post
title: "How to clone a failing hard drive"
date: 2014-02-09 15:22:37 -0800
comments: true
categories: 
---
I recently learned a little about data recovery the hard way. The other day when I tried to ssh to my server at home, it was unresponsive. I assumed there had been a power outage. When I got home, that sadly proved not to be the case. My 2TB drive had been remounted readonly and smartctl indicated that it was failing. I was lucky in that the drive was still bootable. I immediately bought a replacement drive of the same capacity and set to work. My first attempt to salvage things was to boot with a gentoo liveusb and attempt to clone the drive with 
```
dd if=/dev/sda of=/dev/sdb conv=notrunc,noerror  
```
At first this seemed quite promising and iotop showed that data was moving between the drives at around 200MB/s. My enthusiasm quickly died down, however, when the process reached around 384GB and spit out some input/output errors. I let the process complete but assumed the worst. Once the clone had completed, albeit rather fitfully, I tried to boot from the clone and was hit with a kernel panic. For my second (and this time successful) attempt I switched to using ddrescue. For this command it is critical to make sure that you specify the arguments in the correct order as flags are not used to designate input and output drives. Using ddrescue is a multi-step process- first a fast pass skipping all errors, then potentially several slower passes to rescue problematic sections of the disk. 

Note that this first command will take a very long time for a large drive (depending on how many errors are present). In my case with a 2TB drive, it took around 7 hours.
```
ddrescue source destination -n rescue.log
```
The -n will proceed past any errors and then we'll revisit them with the next command.

```
ddrescue source destination -n -r1 rescue.log
```
the -r specifies the number of times to attempt to rescue a block

Then if you run into errors, consider adding -R to read the disk in reverse
```
ddrescue source destination -n -r1 rescue.log
```

Hopefully throughout this process the error count will not go up and the error size will drop. Once satisfied with the results, check the integrity of the filesystem using the fsck appropriate for your system. In my case:

```
e2fsck -f /dev/sda1
e2fsck -f /dev/sda3
e2fsck -f /dev/sda4
```

And then finally the moment of truth- try booting with the cloned drive (if that is your goal, some folks may wish to just mount the disk and copy data elsewhere). In my case the system did successfully boot at this point.
