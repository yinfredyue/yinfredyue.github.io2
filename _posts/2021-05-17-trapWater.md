---
layout: single
categories: 
    - LeetCode
author: Yue Yin

toc: true
toc_sticky: true
---

# Traping rain water

Problem: [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)

## Solution1: Vertical Bar

At each slot `i`, the amount of water that can be trapped = `min(leftHighestBar, rightHighestBar) - height[i]`. This approach is essentially calculting vertical bars (see a, b, c, d ,e in graph).

```
X           X
X X       X X
X X X   X X X
X X X X X X X

X a b c d e X
X X b c d X X
X X X c X X X
X X X X X X X
```

It's easy to come up with DP solutions. Time: O(n). Space: O(n)



## Solution2: Horizontal Bar

Can we calculate water horizontally? (See a, b, c in graph)

```
X           X
X X       X X
X X X   X X X
X X X X X X X

X c c c c c X
X X b b b X X
X X X a X X X
X X X X X X X
```

We can use a monotonic stack for this purpose: for each height, find the closest higher heights to the left and right. See detailed explanation [here](https://yinfredyue.github.io/leetcode/MonotonicStack/).



## Solution3: Two pointers

### Variant 1

```c++
class Solution {
public:
    int trap(vector<int>& H) {
        int n = H.size();
        int left = 0, right = n-1, leftMax = 0, rightMax = 0, res = 0;
        while (left <= right) {
            if (leftMax < rightMax) {
                if (leftMax > H[left]) {
                    res += leftMax - H[left];
                }
                leftMax = max(leftMax, H[left++]);
            } else {
                if (rightMax > H[right]) {
                    res += rightMax - H[right];
                }
                rightMax = max(rightMax, H[right--]);
            }
        }
        return res;
    }
};
```

Very clear explanation [here](https://leetcode-cn.com/problems/trapping-rain-water/solution/jie-yu-shui-by-leetcode/327718/). The idea is to keep track of `leftMax` in `H[:left-1]` and `rightMax` in `H[right+1:]`. For `H[left]`, `leftMax` correctly tracks the maximum value in `H[:left-1]`, whereas `rightMax` only tracks max value in `H[right+1:]`, leaving `H[left:right]` unknown. Can we calculate the water for `H[left]` in this case? The asnwer is yes! When `leftMax < rightMax`, water trapped at `H[left]` will be determined by `leftMax`, because maximum value in `H[left:]` will be no smaller than `rightMax`. Thus, we can calculate the water at `H[left]` by `max(0, leftMax - H[left])`. 

In notation: 

```
When leftMax < rightMax:
    max{ H[left+1:] } >= max{ H[right+1:] } = rightMax > leftMax = max{ H[:left-1] }
So
    min(max{ H[left+1:] }, max{ H[:left-1] }) = max{ H[:left-1] } = leftMax
```

The idea is simlar: we can calculate water at `H[right]` when `leftMax > rightMax`.



### Variant 2

```c++
class Solution {
public:
    int trap(vector<int>& H) {
        int n = H.size();
        int left = 0, right = n-1, leftMax = 0, rightMax = 0, res = 0;
        while (left <= right) {
            if (H[left] < H[right]) { // *
                if (leftMax > H[left]) {
                    res += leftMax - H[left];
                }
                leftMax = max(leftMax, H[left++]);
            } else {
                if (rightMax > H[right]) {
                    res += rightMax - H[right];
                }
                rightMax = max(rightMax, H[right--]);
            }
        }
        return res;
    }
};
```

The only difference from previous approach is `H[left] < H[right]`. Why we can do `res += leftMax - H[left]` when `H[left] < H[right]`? What if `leftMax < rightMax`?

The reason is that `H[left] < H[right]` implies `leftMax < rightMax`. See the following proof.

Proof by contradiction: https://leetcode-cn.com/problems/trapping-rain-water/solution/jie-yu-shui-by-leetcode/868621. Statement: if `H[left] < H[right]`, then `H[right]` is the maximum value in all `H[:left]` and `H[right:]`. Proof: (1) suppose the maximum element is not `H[right]`, but `H[m]` where `m > right`. The face that `right` decreases from `m` to `m-1` means that there exists a value in `H[:left]` that's greater than `H[m]`, which means `H[m]` is not the maximum element. Thus, we reach a contradiction. (2) suppose the maximum element is not `H[right]`, but `H[m]` where `m < left` (clearly `H[left]` cannot be the maximum since it's less than `H[right]`). The fact that `left` increase from `m` to `m+1` means that there's a value in `H[right:]` that is greater than `H[m]`. Thus, we reach a contradiction. Based on (1) and (2), we know that when `H[left] < H[right]`, `H[right]` is the maximum value in all travsered values. In other words, `leftMax < rightMax = H[right]` holds.

Proof by induction (This proof is inspired by [this](https://leetcode-cn.com/problems/trapping-rain-water/solution/xiang-xi-tong-su-de-si-lu-fen-xi-duo-jie-fa-by-w-8/131029).). Statement to prove: when `H[left] < H[right]`, `H[right]` is the maximum of all traversed; when `H[left] > H[right]`, `H[left]` is the maximum of all traversed. Pre-condition: when `left = 0`, `right = n-1`, the statement holds. Induction step: suppose the statement holds for `left` and `right`. You can easily verify that for all four possible situtations, the statements still hold:

- H[left] < H[right], and H[left+1] < H[right]
- H[left] < H[right], and H[left+1] > H[right]
- H[left] > H[right], and H[left+1] < H[right]
- H[left] > H[right], and H[left+1] < H[right]

The remaining idea is similar to variant1. 

In summary, the idea is that the amount of water trapped is determined by the *lowest* of all boundaries, i.e., when we know the lowest (not all) boundary, we can calculate the water. In 2D, there are only two boundaries (left and right).

From the perspective of `H[left]` and `H[right]`:  `leftMax` represents accurate left boundary for `H[left]`, inaccurate left boundary for `H[right]`; `rightMax` represents inaccurate rigth boundary for `H[left]`, and accurate right boundary for `H[right]`. When `leftMax < rightMax`, we are sure that left boundary is the lowest boundary for `H[left]`, when `leftMax > rightMax`, we are sure that the right boundary is the lowest boundary for `H[right]`. 



Problem: [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water-ii/).

Follow the same idea as above: https://leetcode.com/problems/trapping-rain-water-ii/discuss/89495/How-to-get-the-solution-to-2-D-%22Trapping-Rain-Water%22-problem-from-1-D-case. Initially we pushed all boundaries to the priority queue, and popping from the lowest boundary. Thus, when a slot is popped from the priority_queue, it knows that *the lowest of its boundaries* is determined, with the value of `currMax`. So, the water at this slot can be calculated. Visualization: https://leetcode.com/problems/trapping-rain-water-ii/discuss/89472/Visualization-No-Code

