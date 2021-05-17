---
layout: single
categories: 
    - LeetCode
author: Yue Yin

toc: true
toc_sticky: true
---

## Monotonic Stack

A monotonic stack is just a normal stack, but we can push/pop the stack in certain way so that values in the stack become sorted (ascending/descending).

To build a ascending stack using an array of number:

```c++
void monotonicStackExample(vector<int> arr) {
    stack<int> stk;
    for (int i = 0; i < arr.size(); ++i) {
        while (!stk.empty() && stk.top() > arr[i]) {
            stk.pop();
        }
        
        stk.push(arr[i]);
    }
}
```

The above code achieves the following property for the stack:

- At the start of iteration, `stk` stores values in ascending order;
- At the end of iterating `arr[i]`, `stk` stores values in ascending order, with `arr[i]` on the top of the stack.

We can make the following observations, at the iteration for `arr[i]`:

- Observation #1: When popping element, say `arr[j]` (where `j < i`) from stack, `arr[i]` is the first element smaller than `arr[j]`; i.e., `arr[i]` is the leftmost element in `arr[j+1:]` that's smaller than `arr[j]`;
- Observation #2: After `arr[j]` is popped from the stack, the top of the stack, say `arr[k]` where `k < j`, is the latest element smaller than `arr[j]`; i.e., `arr[k]` is the rightmost element in `arr[:j-1]` that's smaller than `arr[j]`.

In this way, we get the closest element around `arr[j]` that's smaller than `arr[j]`. Visually, the values look like. 

```
       arr[j]
arr[k]       
              arr[i]
```



## Properties

See if you can understand the following properties. Assume we iterate the array from left to right.

- For ascending stack, when an element `arr[i]` is popped from the stack, you get
    - the rightmost element in `arr[:i-1]` that's smaller than `arr[i]`, and 
    - leftmost element in `arr[i+1:]` that's smaller than `arr[i]`
- For descending stack, when an element `arr[i]` is popped from the stack, you get
    - the rightmost element in `arr[:i-1]` that's larger than `arr[i]`, and 
    - leftmost element in `arr[i+1:]` that's larger than `arr[i]`

Visualization:

```
Ascending stack:
   X
X    X

Descending stack:
X   X
  X
```



## Apply to problems

The properties above help you to several categories of algorithm problems:

"Next greater element": find the closet elment that's greater than the current element. In [Leetcode 496](https://leetcode.com/problems/next-greater-element-i/) and [Leetcode 739. Daily Temperatures](https://leetcode.com/problems/daily-temperatures/), it's the same as finding the leftmost element that's greater than current element ([reference](https://github.com/labuladong/fucking-algorithm/blob/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E7%B3%BB%E5%88%97/%E5%8D%95%E8%B0%83%E6%A0%88.md)). So we use a descending stack. 

"Closest greater/smaller element": a generalized case of the "next greater element" problem. In [Leetcode 42](https://leetcode.com/problems/trapping-rain-water/), for each height, find the closest higher heights on both sides; this allows you to calculate water horizontally ([reference](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/solution/84-by-ikaruga/)). 

