---
layout: post
title: "[LeetCode]Excel Sheet Column Title"
date: 2015-04-07 21:13:20 +0800
comments: true
categories: ['LeetCode', 'Cpp']
---
Given a positive integer, return its corresponding column title as appear in an Excel sheet.
<!--more-->

For example:
```
    1 -> A
    2 -> B
    3 -> C
    ...
    26 -> Z
    27 -> AA
    28 -> AB 
```

Here is a very short C++ solution using charactor lookup array.

The idea behind this algorithm is coming from the following numeric representation mapping to charactors:
``` 
   A       B             Z  
1+26*0, 2+26*0, ..., 26+26*0  
  AA      AB            AZ    
1+26*1, 2+26*1, ..., 26+26*1   
  BA      BB            BZ     
1+26*2, 2+26*2, ..., 26+26*2   
```

And we use a *tricky* charactor map which starts from 'Z'.

``` Cpp Excel Sheet Column Title
class Solution {
public:
    string convertToTitle(int n) {
        string map = "ZABCDEFGHIJKLMNOPQRSTUVWXY";
        string result;

        while (n) {
        	result = map[n-- % 26] + result;
        	n /= 26;
        }

        return result;
    }
};
```


