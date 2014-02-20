---
layout: post
title: "Printing issues in gentoo linux"
date: 2014-02-20 01:04:45 -0800
comments: true
categories: 
---
It's been a while since I last printed something from my linux box and when the need most recently came up, a number of issues arose. My printer is an old Samsung ML-1710 which uses the splix driver, obtained here http://www.openprinting.org/driver/splix/ I grabbed the 2.0.0 RPM for LSB 3.2 version of the package. I then converted the rpm to tgz using alien:
```
sudo alien -t openprinting-splix-2.0.0-2lsb3.2.x86_64.rpm
tar xfz openprinting-splix-2.0.0.tgz
```
and started poking around in the directory contained within the tarball. Its content is meant to be placed in /opt so I copied the OpenPrinting-SpliX subdirectory of the tarball to /opt
```
sudo cp -r OpenPrinting-SpliX /opt/
```

and now I ran into my first 2 problems:
```
$ lpstat -p -d
printer Samsung_ML-1710 is idle.  enabled since Thu Feb 20 00:21:43 2014
        File "/opt/OpenPrinting-SpliX/cups/lib/filter/rastertoqpdl" not available: No such file or directory
lpstat: error - PRINTER environment variable names non-existent destination "samsung".
```
My PRINTER environment variable was not set correctly, that was easily fixed by 
```
export PRINTER=Samsung_ML-1710
```
the issue with rastertoqpdl was not so obvious. The rastertoqpdl binary is linked in an interesting way:
```
$ readelf -a rastertoqpdl  | grep -i interp
  [ 1] .interp           PROGBITS         0000000000400200  00000200
  INTERP         0x0000000000000200 0x0000000000400200 0x0000000000400200
      [Requesting program interpreter: /lib64/ld-lsb-x86-64.so.3]
   01     .interp 
   02     .interp .note.ABI-tag .hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt .init .plt .text .fini .rodata .eh_frame_hdr .eh_frame .gcc_except_table 
```
I did not have ld-lsb-x86-64.so.3 available so:
```
sudo ln -s /lib64/ld-linux-x86-64.so.2 /lib64/ld-lsb-x86-64.so.3
```
Now a different issue was revealed:
```
$ lpstat -p -d
printer Samsung_ML-1710 is idle.  enabled since Thu Feb 20 00:49:21 2014
        File "/opt/OpenPrinting-SpliX/cups/lib/filter/rastertoqpdl" has insecure permissions (0100755/uid=1000/gid=0).
system default destination: Samsung_ML-1710
```
Apparently at some point cups was updated so that files like rastertoqpdl could not have write permissions, so:
```
sudo chmod -w rastertoqpdl
sudo /etc/init.d/cupsd restart
```

and now finally:
```
$ lpstat -p -d
printer Samsung_ML-1710 is idle.  enabled since Thu Feb 20 00:56:43 2014
        Rendering completed
system default destination: Samsung_ML-1710


$ lpr my_file.ps
```
