---
layout: post
title: "Fix annoying perl locale warnings"
date: 2015-08-09 22:22:07 +0800
comments: true
categories: ['Linux']
---
When I was setting up Postgres on a new Linux VM server last Friday, a perl warning was keeping showing up. 
<!--more-->

```text
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = (unset),
	LC_ALL = (unset),
	LC_PAPER = "zh_CN.UTF-8",
	LC_ADDRESS = "zh_CN.UTF-8",
	LC_MONETARY = "zh_CN.UTF-8",
	LC_NUMERIC = "zh_CN.UTF-8",
	LC_TELEPHONE = "zh_CN.UTF-8",
	LC_IDENTIFICATION = "zh_CN.UTF-8",
	LC_MEASUREMENT = "zh_CN.UTF-8",
	LC_TIME = "zh_CN.UTF-8",
	LC_NAME = "zh_CN.UTF-8",
	LANG = "en_US.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
```

Well it's annoying. XD

Alright let's fix this.

```sh
sudo locale-gen en_US en_US.UTF-8
[sudo] password for administrator: 
Generating locales...
  en_US.ISO-8859-1... done
  en_US.UTF-8... up-to-date
Generation complete.
```
Then

```
sudo dpkg-reconfigure locales
Generating locales...
  en_AG.UTF-8... done
  en_AU.UTF-8... done
  en_BW.UTF-8... done
  en_CA.UTF-8... done
  en_DK.UTF-8... done
  en_GB.UTF-8... done
  en_HK.UTF-8... done
  en_IE.UTF-8... done
  en_IN.UTF-8... done
  en_NG.UTF-8... done
  en_NZ.UTF-8... done
  en_PH.UTF-8... done
  en_SG.UTF-8... done
  en_US.ISO-8859-1... up-to-date
  en_US.UTF-8... up-to-date
  en_ZA.UTF-8... done
  en_ZM.UTF-8... done
  en_ZW.UTF-8... done
  zh_CN.UTF-8... up-to-date
Generation complete.
```
OK. Done. I guess there was a way to simply ignore these settings. Whatever.