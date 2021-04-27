---
layout: single
categories: 
    - LeetCode
author: Yue Yin

toc: true
toc_sticky: true
---



This post is about the terminate conditions when solving algorithm problems.



## Binary Search

The idea of binary search is simple, but there are two variants of implementation.

### Variant 1

```
while (l < r) {
	int m = l + (r - l) / 2;
	if (...) {
		l = m-1;
	} else {
		r = m;	
	}
}
```

First, you should never use `l = m`. When `l + 1 >= m`, this causes infinite loop. Thus, we use `l = m-1` and `r = m`. However, even with `r = m`, you should avoid having `l == r`, which cause infinite lsoop again. Conclusion: when using `r=m`, use `l < r` to avoid infinite loop.

### Variant 2

```
while (l <= r) {
	int m = l + (r - l) / 2;
	if (...) {
		l = m-1;
	} else {
		r = m+1;	
	}
}
```

For `l = m-1` and `r = m+1`, no infinite loop is possible, so you can use `l <= r`. 



## Two pointers

Should you use `l < r` or `l <= r`? Answer: if you want `while` to execute for `l == r`, use `l <= r`. Otherwise, use `l < r`. 

```
while (l < r) {
   // ...
}
```

