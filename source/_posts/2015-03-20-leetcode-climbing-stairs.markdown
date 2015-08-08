---
layout: post
title: "[LeetCode]Climbing Stairs"
date: 2015-03-20 20:01:00 +0800
comments: true
categories: ['LeetCode', 'C']
---
You are climbing a stair case. It takes n steps to reach to the top.

Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top?  
<!--more-->


``` C Climbing Stairs
int climbStairs(int n) {
    int stepOne = 1;
    int stepTwo = 0;
    int ret = 0;
    int i;
    
    for (i = 0; i < n; i++) {
        ret = stepOne + stepTwo;
        stepTwo = stepOne;
        stepOne = ret;
    }
    
    return ret;
}
```