---
layout: post
title: "[LeetCode]Majority Element"
date: 2015-03-20 19:57:22 +0800
comments: true
categories: ['LeetCode', 'C']
---
Given an array of size n, find the majority element. The majority element is the element that appears more than *⌊ n/2 ⌋* times.

You may assume that the array is non-empty and the majority element always exist in the array.  

<!--more-->


``` C Majority Element
intcomp(const void * a,const void * b)
{
    return * (int *)a - * (int *)b;
}
int majorityElement(int num[], int n) {
    qsort(num, n, sizeof(int), intcomp);
    return num[n/2];
}
```