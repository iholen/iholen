---
layout: post
title:  "插入、希尔、归并排序"
date:   2020-03-28 14:25:05 +0800
categories: algorithm
author: iholen
introduction: 插入、希尔、归并
---
### 插入排序

```java
/**
 * 插入排序
 * @param nums
 */
public void insertSort(int[] nums) {
    // 从数组第二个元素开始遍历
    for (int i = 1; i < nums.length; i++) {
        // 记录待插入元素值
        int current = nums[i];
        int j = i - 1;
        for (; j >= 0; j--) { // 倒序遍历已排序的数组
            if (current < nums[j]) { // 已排序数组的元素 大于 待插入元素
                nums[j + 1] = nums[j]; // 将已排序的数组 元素后移
            } else {
                break;
            }
        }
        // 将待插入元素放入查找到的位置
        nums[j + 1] = current;
    }
}

```
### 希尔排序

```java
/**
 * 希尔排序
 * @param nums
 */
public void shellSort(int[] nums) {
    // 设置步长
    int step = nums.length >> 1;
    while(step > 0) {
        // 从数组第二个元素开始遍历
        for (int i = step; i < nums.length; i++) {
            // 记录待插入元素值
            int current = nums[i];
            int j = i - step;
            for (; j >= 0; j -= step) { // 倒序遍历已排序的数组
                if (current < nums[j]) { // 已排序数组的元素 大于 待插入元素
                    nums[j + step] = nums[j]; // 将已排序的数组 元素后移
                } else {
                    break;
                }
            }
            // 将待插入元素放入查找到的位置
            nums[j + step] = current;
        }
        step >>= 1;
    }
}

```
### 归并排序

```java
/**
 * 临时数组
 */
int[] tempArr;
/**
 * 归并排序
 * @param nums
 */
public void mergeSort(int[] nums) {
    tempArr = new int[nums.length];
    mergeSort(nums, 0, nums.length - 1);
}

/**
 * 拆分完后合并
 * @param nums
 * @param left
 * @param right
 */
public void mergeSort(int[] nums, int left, int right) {
    if (left < right) {
        int mid = (right + left) >> 1;
        mergeSort(nums, left, mid);
        mergeSort(nums, mid + 1, right);
        merge(nums, left, mid, right);
    }
}

/**
 * 合并
 * @param nums
 * @param left
 * @param mid
 * @param right
 */
public void merge(int[] nums, int left, int mid, int right) {
    // 临时数组下标
    int tempIndex = left;
    // 左边数组下标
    int i = left;
    // 右边数组下标
    int j = mid + 1;
    while (i <= mid && j <= right) {
        if (nums[i] < nums[j]) {
            tempArr[tempIndex++] = nums[i++];
        } else {
            tempArr[tempIndex++] = nums[j++];
        }
    }
    // 将左边数组剩余元素移入临时数组
    while(i <= mid) {
        tempArr[tempIndex++] = nums[i++];
    }
    // 将临时数组中的数据移到原数组中
    for (int k = left; k < tempIndex; k++) {
        nums[k] = tempArr[k];
    }
}  
```
---
### 三者对比

| 排序名称 | 时间复杂度 | 是否稳定 | 额外空间开销 |
|:----|:----|:----|:----|
| 插入排序 | O(n^2) | 稳定 | O(1) |
| 希尔排序 | O(n^2) | 不稳定 | O(1) |
| 归并排序 | O(n^2) | 稳定 | O(n) |
