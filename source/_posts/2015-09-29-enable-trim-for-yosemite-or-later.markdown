---
layout: post
title: "Enable TRIM for Yosemite or later"
date: 2015-09-29 23:01:31 +0800
comments: true
categories: ['OS X', 'Tips']
---

So far, there has been several well-known tools such as [Trim Enabler](https://www.cindori.org/software/trimenabler/) for enabling Trim on OS X. 

In the latest update of OS X 10.10.4, Apple silently added a new tool called **trimforce** for such use as well. After suffering from the "prohibitory" sign on booting, I chose to enable Trim with trimforce this time. It worked. :-)

Here's the manual for trimforce
<!--more-->  

![TRIM-manual](/images/posts/TRIMFORCE-manual.png)  

Command to use trimforce is `sudo trimforce enable`, after which you'll see as following.  

![TRIM-screenshot](/images/posts/TRIMFORCE-screenshot.png)

In case someone else is seeing the weird "prohibitory" sign on booting, here's how to fix it.

1. Reboot and press Command+R to enter Recovery Mode.

2. Open Terminal from the menu.

3. Type following commands. Note, you may replace *YOURDISKNAME* with your disk name which can be found from  ```ls /Volumes/```.  

```sh
	rm -rf /Volumes/YOURDISKNAME/System/Library/Extensions/IOAHCIFamily.kext
	cp -r /System/Library/Extensions/IOAHCIFamily.kext /Volumes/YOURDISKNAME/System/Library/Extensions/IOAHCIFamily.kext
	touch /Volumes/YOURDISKNAME/System/Library/Extensions
	kextcache -u /Volumes/YOURDISKNAME
	reboot
```
