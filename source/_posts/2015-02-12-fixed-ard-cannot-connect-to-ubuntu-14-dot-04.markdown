---
layout: post
title: "Fixed: ARD cannot connect to Ubuntu 14.04"
date: 2015-02-12 16:53:28 +0800
comments: true
categories: ['VNC', 'tips', 'OS X']
---

I've been so far using [RealVNC](http://www.realvnc.com) as VNC client. It's perfect except one thing - the icon.  
The original icon appears really big when standing on dock. I even resized it to make the icon look *normal*.  

![dock](/images/posts/2015-02-12-dock.png)  
<!--more-->
But what about the OS X's built-in *Screen Sharing*? Well, here comes the problems.  
> Connection failed to “XXX's remote desktop on XXX”. The software on the remote computer appears to be incompatible with this version of Screen Sharing.  

This message was showing up again and again when I tried to connect to a server running Ubuntu 14.04.  
I've set up everything as usual (port, encryption, firewall ...). Finally I made it work.    


In fact this issue was caused by a [BUG](https://bugs.launchpad.net/ubuntu/+source/vino/+bug/1281250) introduced into [Vino](https://wiki.gnome.org/Projects/Vino) (Default VNC server for GNOME).  

####Here's the solution:  

- Make sure `Desktop Sharing` has been set up properly.           
- Install `gconf-tools`. e.g. 'sudo apt-get install dconf-tools'  
- Run `dconf-Editor`.                                             
- Expand 'org'                                                    
- Expand 'gnome'                                                  
- Expand 'Desktop'                                                
- Select 'Remote Access'                                          
- Uncheck 'Require Encrption'                                     

![Desktop-Sharing-setting](/images/posts/2015-02-12-Ubuntu-setting.png)  

![gconf-Editor](/images/posts/2015-02-12-gconf-Editor.png)  

Now you can connect to the server using built-in *Screen Sharing* or *Apple Remote Desktop*.  

![Screen Sharing](/images/posts/2015-02-12-Screen-Sharing.png)  

![Apple Remote Desktop](/images/posts/2015-02-12-ARD.png)  