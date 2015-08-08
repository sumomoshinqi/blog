---
layout: post
title: "[LeetCode]Compare Version Numbers"
date: 2015-03-20 19:27:39 +0800
comments: true
categories: ['LeetCode', 'C']
---
Compare two version numbers version1 and version2.    
If version1 > version2 return 1, if version1 < version2 return -1, otherwise return 0.    

You may assume that the version strings are non-empty and contain only digits and the . character.     
The `.` character does not represent a decimal point and is used to separate number sequences.    
For instance, `2.5` is not "two and a half" or "half way to version three", it is the fifth second-level revision of the second first-level revision.

Here is an example of version numbers ordering:   
`0.1 < 1.1 < 1.2 < 13.37`
<!--more-->


``` C compareVersion
int compareVersion(char * version1, char * version2){
    int v1 = 0, v2 = 0;
    int p1 = 0, p2 = 0;
    
    while (p1 < strlen(version1) || p2 < strlen(version2)) {
        v1 = 0;
        while (p1 < strlen(version1)) {
            if (version1[p1] == '.') {
                ++p1;
                break;
            }
            v1 = 10 * v1 + (int)(version1[p1] - '0');
            ++p1;
        }
        
        v2 = 0;
        while (p2 < strlen(version2)) {
            if (version2[p2] == '.') {
                ++p2;
                break;
            }
            v2 = 10 * v2 + (int)(version2[p2] - '0');
            ++p2;
        }
        
        if (v1 < v2) {
            return -1;
        }
        else if (v1 > v2) {
            return 1;
        }
    }
    
    return 0;
}  
```
