---
layout: post
title: "How to get pepper flash working perfectly in chromium with dual screen support"
date: 2014-02-10 01:17:21 -0800
comments: true
categories: 
---
Getting flash working just right in chromium is fairly tricky due to a number of issues that one may encounter along the way:

* Getting adblock to work on youtube videos ads 
* Getting the pepper (PPAPI) version of flash to even appear in chromium as a choice
* Preventing flash from minimizing when fullscreen on one monitor while focus is on the other monitor
* Preventing flash windows from going all black when you switch tabs  

<br>
<u>Getting adblock to work on youtube video ads</u>  

The adblock extension only works with PPAPI flash, so if you are using older NPAPI flash you will probably notice that youtube video ads are still showing up even when adblock is enabled. So to get adblock working you just need to install and enable PPAPI flash.

<u>Making PPAPI/pepper flash visible in chromium in gentoo</u>  
Now that we know we need the PPAPI version of flash we simply install it:

```
emerge chrome-binary-plugins
```
At least for me, however, simply emerging this package was not enough. The recommended next step is to edit /etc/chromium/default to set the CHROMIUM_FLAGS variable with the path to the flash plugin. In my case, this variable seems to never get used, so instead I created a bash alias for chromium that explicitly specifies the plugin path:

```
alias chromium="/usr/bin/chromium --embed-flash-fullscreen --ppapi-flash-path=/usr/lib64/chromium-browser/PepperFlash/libpepflashplayer.so --ppapi-flash-version=12.0.0.44 --disable-metrics --disable-metrics-reporting --purge-memory-button"
```

Note that I also pass a flag --embed-flash-fullscreen  That flag allows flash to run in fullscreen in a one monitor while doing something else in another. This flag is apparently only going to be necessary temporarily while some part of chromium is reworked.

Finally the last issue one may encounter is that if you have a flash video open in the current tab, switch to another tab and then switch back, your video may appear all black while the audio keeps playing. To fix this:
in chrome://flags
enable "Override software rendering list"
 and disable "Threaded compositing"
