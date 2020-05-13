---
layout: wiki
title: 每日一算
categories: Algorithm
description: 大神博客的优秀文档，建议多看看。
keywords: Java， Android
---

# 每日一算

每日一题，求恒不求多。

## 合并两个有序数组

<font size=4>**题目描述**</font>

给你两个有序整数数组 nums1 和 nums2，请你将 nums2 合并到 nums1 中*，*使 nums1 成为一个有序数组。

**说明：**
* 初始化 nums1 和 nums2 的元素数量分别为 m 和 n 。
* 你可以假设 nums1 有足够的空间（空间大小大于或等于 m + n）来保存 nums2 中的元素。

**示例：**

输入:
nums1 = [1,2,3,0,0,0], m = 3
nums2 = [2,5,6],       n = 3

输出: [1,2,2,3,5,6]

``` java

    public static void main(String[] args) {
        int[] nums1 = new int[]{1, 4, 5, 0, 0, 0};
        int[] nums2 = new int[]{2, 5, 6};
        merge(nums1, nums1.length - nums2.length, nums2, nums2.length);
        System.out.println(Arrays.toString(nums1));
    }

    /** 双指针，从后向前比较**/
    private static void merge(int[] nums1, int m, int[] nums2, int n) {
        int pointA = nums1.length - nums2.length -1;
        int pointB = nums2.length - 1;

        while (pointA > -1 && pointB > -1) {
            if (nums1[pointA] < nums2[pointB]) {
                nums1[pointA + pointB + 1] = nums2[pointB];
                pointB--;
            } else {
                nums1[pointA + pointB + 1] = nums1[pointA];
                pointA--;
            }
        }
    }
```

Date : 2020.05.12  
From : LeetCode 图解 | 88. 合并两个有序数组  

## 搜索旋转排序数组

<font size=4>**题目描述**</font>

假设按照升序排序的数组在预先未知的某个点上进行了旋转。

( 例如，数组 [0,1,2,4,5,6,7] 可能变为 [4,5,6,7,0,1,2] )。

搜索一个给定的目标值，如果数组中存在这个目标值，则返回它的索引，否则返回 -1 。


**说明：**
* 你可以假设数组中不存在重复的元素。
* 你的算法时间复杂度必须是 O(log n) 级别。

**示例：**

* 示例1  
输入: nums = [4,5,6,7,0,1,2], target = 0  
输出: 4

* 示例2  
输入: nums = [4,5,6,7,0,1,2], target = 3  
输出: -1

**分析：**
* 时间复杂度必须是O(log n)基本上就确定了需要使用二分查找。
* 需要额外特别注意边界上的判断，是否取等，哪里取等这些。
* 二分如果使用递归的话可能会造成栈溢出。

```java

    public static void main(String[] args) {
        int[] nums = new int[]{5,1,3};
        int target = 3;
        System.out.println("search : " + search(nums, target));
    }

    public static int search(int[] nums, int target) {
        //  还是递归思路，采用二分查找，但是是分条件进行二分查找
        if (nums == null || nums.length == 0) {
            return -1;
        }

        int left = 0;
        int right = nums.length - 1;

        while (left <= right) {
            int middle = (left + right) /2;
            if (nums[middle] == target) {
                return middle;
            }

            if (nums[middle] >= nums[left]) {
                if (target >= nums[left] && target < nums[middle]) {
                    right = middle -1;
                } else {
                    left = middle + 1;
                }
            } else {
                if (target > nums[middle] && target <= nums[right]) {
                    left = middle + 1;
                } else {
                    right = middle -1;
                }
            }
        }

        return -1;
    }
```

Date : 2020.05.13  
From : LeetCode | 探索字节跳动. 搜索旋转排序数组  


