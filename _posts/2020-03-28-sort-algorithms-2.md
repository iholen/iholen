---
layout: post
title:  "排序算法之选择、冒泡、快速排序"
date:   2020-03-28 14:25:05 +0800
categories: algorithm
author: heoller
introduction: 选择、冒泡、快速排序实现和对比
---
### 选择排序

```java
/**
 * 选择排序
 * @param nums
 */
public void selectSort(int[] nums) {
    int len = nums.length;
    // 记录最小值的数组下标
    int minIndex;
    for (int i = 0; i < len - 1; i++) {
        minIndex = i;
        for (int j = i + 1; j < len; j++) {
            // 如果当前元素小于标记的最小值，记录新下标
            if (nums[minIndex] > nums[j])
                minIndex = j;
        }
        // 做交换
        swap(nums, i, minIndex);
    }
}

/**
 * 交换
 * @param nums
 * @param i
 * @param j
 */
public void swap(int[] nums, int i, int j) {
    if (i == j) return;
    nums[i] += nums[j];
    nums[j] = nums[i] - nums[j];
    nums[i] = nums[i] - nums[j];
}

```
### 冒泡排序

```java
/**
 * 冒泡排序
 * 添加了switchFlag优化
 * @param nums
 */
public void bubbleSort(int[] nums) {
    int len = nums.length;
    // 标记数组是否交换过，没有交换过就表示数组本来就有序
    boolean switchFlag;
    // 循环次数为n-1时就可以将数组排好序
    for (int i = 0; i < len - 1; i++) {
        switchFlag = false;
        // 每循环一次，就是排好一个元素，所以减去i
        for (int j = 0; j < len - 1 - i; j++) {
            if (nums[j] > nums[j + 1]) {
                swap(nums, j, j + 1);
                // 标记 有交换过
                switchFlag = true;
            }
        }
        // 如果没有交换过，直接终止循环
        if (!switchFlag)
            break;;
    }
}

/**
 * 交换
 * @param nums
 * @param i
 * @param j
 */
public void swap(int[] nums, int i, int j) {
    if (i == j) return;
    nums[i] += nums[j];
    nums[j] = nums[i] - nums[j];
    nums[i] = nums[i] - nums[j];
}

```
### 快速排序

```java
/**
 * 快速排序
 * @param nums
 */
public void quickSort(int[] nums) {
    quickSort(nums, 0, nums.length - 1);
}
/**
 * 分段排序
 * @param nums
 * @param left
 * @param right
 */
public void quickSort(int[] nums, int left, int right) {
    if (left >= right)
        return;
    int i = left;
    int j = right;
    int refIndex = left;
    // 终止条件
    while (i < j) {
        // 从右边找到小于基准数的下标
        while (i < j && nums[j] >= nums[refIndex]) {
            j--;
        }
        // 将找到的那个数移到基准数左边(交换位置)
        swap(nums, refIndex, j);
        // 修改基准数下标
        refIndex = j;
        // 再从左边找到大于基准数的下标
        while (i < j && nums[i] <= nums[refIndex]) {
            i++;
        }
        // 将找到的那个数移到基准数右边(交换位置)
        swap(nums, refIndex, i);
        // 修改基准数下标
        refIndex = i;
    }
    // 递归，继续排左边
    quickSort(nums, left, i - 1);
    // 递归，继续排右边
    quickSort(nums, i + 1, right);
}
/**
 * 交换
 * @param nums
 * @param i
 * @param j
 */
public void swap(int[] nums, int i, int j) {
    if (i == j) return;
    nums[i] += nums[j];
    nums[j] = nums[i] - nums[j];
    nums[i] = nums[i] - nums[j];
}
```
---
### 三者对比

| 排序名称 | 时间复杂度 | 是否稳定 | 额外空间开销 |
|:----|:----|:----|:----|
| 选择排序 | O(n^2) | 不稳定 | O(1) |
| 冒泡排序 | O(n^2) | 稳定 | O(1) |
| 快速排序 | O(nlogn) | 不稳定 | O(1) |
