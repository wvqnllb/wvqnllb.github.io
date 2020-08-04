# Leetcode BinarySearch


二分总结和相关题目。

<!--more-->

## binarySearch的两种情况
1. 在范围当中一定可以找到
   1. 结束条件明确，直接break；
   2. 结束条件不明确，收敛到 lo == hi, 即为最终位置 
2. 在范围当中不一定可以找到
   1. 将找到和找不到两种情况以不同方式输出(Java Arrays的实现);
   2. 找到收敛位置之后判断该位置元素是否满足条件 lo > hi；   

类型1在leetcode题目中更加多见，难点经常在于如何将题目转换为二分的结束条件。
类型2感觉是在底层实现中更多一些，通用性更强。

---


## java.Arrays.binarySearch();
类似 2.1
```java
    private static int binarySearch0(int[] a, int fromIndex, int toIndex,
                                     int key) {
        int low = fromIndex;
        int high = toIndex - 1;

        while (low <= high) {
            int mid = (low + high) >>> 1;
            int midVal = a[mid];

            if (midVal < key)
                low = mid + 1;
            else if (midVal > key)
                high = mid - 1;
            else
                return mid; // key found
        }
        return -(low + 1);  // key not found.
    }
```

---
## 自己实现的通用方法
类似 2.2
```java
    public static int binarytsearch(int[] vec, int targetNumber, int lo, int hi) {
        while(lo <= hi) {
            int mi = (lo + hi) / 2;
            if (targetNumber < vec[mi]) {
                hi = mi - 1;
            } else {
                lo = mi + 1;
            }
        }
        return --lo;
    }
```

---

## Leecode410 分割数组的最大值
[题目链接](https://leetcode-cn.com/problems/split-array-largest-sum/)

> **二分题解：最大最小问题** 
> 
> 贪心地模拟分割的过程，从前到后遍历数组，用 sum 表示当前分割子数组的和，cnt 表示已经分割出的子数组的数量（包括当前子数组），那么每当 sum 加上当前值超过了 x，我们就把当前取的值作为新的一段分割子数组的开头，并将 cnt 加 11。遍历结束后验证是否 cnt 不超过 m。
>
>接下来确定ans范围，因为m在范围[1,n]内，因此ans_right为数组和，ans_left为数组中最大的元素值，并且在该范围内一定可以找到最终ans。

```java
class Solution {
    public int splitArray(int[] nums, int m) {
        int left = 0, right = 0;
        for (int i = 0; i < nums.length; i++) {
            right += nums[i];
            if (left < nums[i]) {
                left = nums[i];
            }
        }
        while (left < right) {
            int mid = (right - left) / 2 + left;
            if (check(nums, mid, m)) {
                right = mid;
            } else {
                left = mid + 1;
            }
        }
        return left;
    }

    public boolean check(int[] nums, int x, int m) {
        int sum = 0;
        int cnt = 1;
        for (int i = 0; i < nums.length; i++) {
            if (sum + nums[i] > x) {
                cnt++;
                sum = nums[i];
            } else {
                sum += nums[i];
            }
        }
        return cnt <= m;
    }
}

作者：LeetCode-Solution
```





