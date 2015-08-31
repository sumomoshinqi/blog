---
layout: post
title: "htop - A better process viewer"
date: 2015-08-31 12:59:29 +0800
comments: true
categories: ['Linux']
---
<!-- more -->
Here's a very good [article](https://delightlylinux.wordpress.com/2014/03/24/htop-a-better-process-viewer-then-top/) about the interactive process viewer [**htop**](http://hisham.hm/htop/).

I even replace the original **top** with **htop** after trying it. For **fishshell** users, you can do this by adding a new function at the end of your `~/.config/fish/config.fish`:

```sh
	function top
		command htop
	end
```
![htop-screenshot](/images/posts/htop-screenshot.png)