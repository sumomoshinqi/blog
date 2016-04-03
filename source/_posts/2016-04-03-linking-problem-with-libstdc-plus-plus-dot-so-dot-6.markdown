---
layout: post
title: "Linking problem with libstdc++.so.6"
date: 2016-04-03 13:53:58 +0800
comments: true
categories: "Linux"
---

I encountered a problem while compiling a C++ project under CLion.
There's an error about undefined reference as shown below.
![link](/images/posts/2016-04-03-link.png)
I was on Ubuntu 14.04 and using Clang. But what is "_ZNSs4_Rep10_M_destroyERKSaIcE@@GLIBCXX_3.4"?

**SOLUTION**  
Add the following parameters:  
**-L /usr/lib64 -lstdc++**  

My CMakeLists.txt has this line:
```
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x -L /usr/lib64 -lstdc++")
```

This will force link with stdc++. (g++ will automatically do this but gcc won't, not sure about clang)

I thought I was missing some lib files at first.
```
➜  ~ ldd /usr/lib/libstdc++.so.6
	linux-gate.so.1 =>  (0xf77ab000)
	libm.so.6 => /lib/i386-linux-gnu/libm.so.6 (0xf7637000)
	libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xf7489000)
	/lib/ld-linux.so.2 (0xf77ac000)
	libgcc_s.so.1 => /lib/i386-linux-gnu/libgcc_s.so.1 (0xf746b000)
```
So it's still the linking matter and it may due to compiler's default options.