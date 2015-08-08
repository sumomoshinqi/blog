---
layout: post
title: "Add a \"Plugged In\" notification sound for Mac"
date: 2015-06-23 19:37:02 +0800
comments: true
categories: ['Tips']
---

iPhone has a special notification sound when plugged in.  
In fact, Mac can do that too!

Here's how to enable it.

- Open terminal.app
- Enter the following commands

```bash
defaults write com.apple.PowerChime ChimeOnAllHardware -bool true; 
open /System/Library/CoreServices/PowerChime.app &
```
There you go. :)


Well, if you no longer want it.   
Enter

```bash
defaults write com.apple.PowerChime ChimeOnAllHardware -bool false;
killall PowerChime
```
Enjoy!