---
layout: post
title: "Customize MongoDB maxConn"
date: 2015-08-08 23:29:05 +0800
comments: true
categories: ['MongoDB', 'Linux', 'Jmeter']
---

Sometimes we need MongoDB support very large number of concurrent connections. To do that, following configurations should be modified.
<!--more-->

###1. Kill current mongod processes(es)
	
```sh
	ps ax | grep mongod    # Find current mongod processes
	kill -9 PID            # Kill them
```

###2. Unlock MongoDB max connection restriction

Edit system configuration file **limits.conf**

```sh
	sudo vim /etc/security/limits.conf  # Configuration file for the pam_limits module
```

> For more information see [limits.conf](http://linux.die.net/man/5/limits.conf)  

Add following lines to change system resource limits
Here **\*** means for all users, **noproc** means max user processes, **nofile** means max open files

```text
	root - core unlimited
	*    - core unlimited
	* soft noproc 65535 
	* hard noproc 65535 
	* soft nofile 409600 
	* hard nofile 409600
``` 
	
Reboot to apply all these changes

> Actually reboot [is not a must](http://unix.stackexchange.com/questions/108603/do-changes-in-etc-security-limits-conf-require-a-reboot), yet it's the easiest way to apply all changes.

```sh
	sudo reboot
```

Now check ulimit settings

```sh
	ulimit -a
```
You shall see your changes here

```
	core file size          (blocks, -c) unlimited
	data seg size           (kbytes, -d) unlimited
	scheduling priority             (-e) 0
	file size               (blocks, -f) unlimited
	pending signals                 (-i) 31420
	max locked memory       (kbytes, -l) 64
	max memory size         (kbytes, -m) unlimited
	open files                      (-n) 409600
	pipe size            (512 bytes, -p) 8
	POSIX message queues     (bytes, -q) 819200
	real-time priority              (-r) 0
	stack size              (kbytes, -s) 8192
	cpu time               (seconds, -t) unlimited
	max user processes              (-u) 31420
	virtual memory          (kbytes, -v) unlimited
	file locks                      (-x) unlimited
```

###3. Restart **mongod** with your own **maxConns**

```sh
	sudo mongod --maxConns=20000
```

Probably you'll see this error

```
	exception in initAndListen std::exception: locale::facet::_S_create_c_locale name not valid, terminating
```

To fix it, set **LC_ALL**
```sh
	export LC_ALL="en_US.UTF-8"
```
	
Restart mongod in background

```sh
	sudo mongod --maxConns=20000 &
```

Enter mongo shell 

```sh
    mongd
```

and check current **maxConns**

```sh
	> db.serverStatus().connections
	{ "current" : 1, "available" : 19999, "totalCreated" : NumberLong(1) }
```
 
 
----
For Jmeter users.
1. Run *Jmeter PerfMon agent* on the server to monitor server system status
You need first upload **ServerAgent-X.X.X.zip** to the server e.g. using **sftp**
Then unzip it anywhere you like :-)
It now looks something like

```sh
	CMDRunner.jar  lib/  LICENSE  ServerAgent-2.2.1.zip  ServerAgent.jar  startAgent.bat  startAgent.sh
```

Run following command to keep the agent running in background

```sh
./startAgent.sh --udp-port 0 --tcp-port 4444 &
```

Now add listeners like **jp@gc-PrfMon Metrics Colletor** to your thread groups, configure the port your'll listen to and run your tests.