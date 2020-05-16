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

**题目描述**

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

**题目描述**

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

## 最长连续递增序列

**题目描述**

给定一个未经排序的整数数组，找到最长且连续的的递增序列。


**说明：**  
无

**示例：**

* 示例1  
输入: [1,3,5,4,7]  
输出: 3  
解释: 最长连续递增序列是 [1,3,5], 长度为3。 
尽管 [1,3,5,7] 也是升序的子序列, 但它不是连续的，因为5和7在原数组里被4隔开。

* 示例2  
输入: [2,2,2,2,2]  
输出: 1  
解释: 最长连续递增序列是 [2], 长度为1。

**分析：**
* 依次遍历数组，记录最大连续的递增序列长度。当序列不再递增时，将计算长度置0重新计算。然后取最大的计算长度返回即可。

```java
    public static void main(String[] args) {
        int[] array = new int[]{1};
        System.out.println("" + findLengthOfLCIS(array));
    }

    public static int findLengthOfLCIS(int[] nums) {
        if (nums == null) {
            return 0;
        }
        if (nums.length < 2) {
            return nums.length;
        }

        int max = 0;
        int start = 1;

        for (int i = 0; i < nums.length -1; i++) {
            if (nums[i] < nums[i + 1]) {
                start++;
                if (i == nums.length - 2) {
                    max = Math.max(max, start);
                }
            } else {
                max = Math.max(max, start);
                start = 1;
            }
        }

        return max;
    }
```

Date : 2020.05.14  
From : LeetCode | 674.最长连续递增序列 


##  数组中的第K个最大元素

**题目描述**

在未排序的数组中找到第 k 个最大的元素。请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。


**说明：**  
无

**示例：**

* 示例1  
输入: [3,2,1,5,6,4] 和 k = 2  
输出: 5

* 示例2  
输入: [3,2,3,1,2,4,5,5,6] 和 k = 4  
输出: 4

**分析：**
* 第一想法是将数组排序，然后输出第k个元素。但是可以通过空间换时间更快得到。然后其次的想法是建立长度为k的数组array，将nums数组中的数据依次加入数据，并且每次都将array排序。添加过程中，当array数组长度到达k时，就用nums中大的元素替换array[0]\(因为array数组始终有序),最终输出array[0]即可。但是碰到一个问题初始化array数组时，数组元素默认为0，当nums数组中含有负数则会出现错误，后看答案看到了`PriorityQueue`结构体与我的做法不谋而合并且解决了数组默认元素为0的问题。

```java
    public static void main(String[] args) {
        int[] array = new int[]{1,2,3};
        int k = 3;
        System.out.println("findKthLargest : " + findKthLargest(array, k));
    }

    public static int findLengthOfLCIS(int[] nums) {
        PriorityQueue<Integer> priorityQueue = new PriorityQueue<>();

        for (int i =0; i < nums.length; i++) {
            priorityQueue.add(nums[i]);
            if (priorityQueue.size() > k) {
                priorityQueue.poll();
            }
        }
        return priorityQueue.poll();
    }
```

Date : 2020.05.15  
From : LeetCode | 探索字节跳动.数组中的第K个最大元素


##  最长连续序列

**题目描述**

给定一个未排序的整数数组，找出最长连续序列的长度。  
要求算法的时间复杂度为 O(n)。


**说明：**  
无

**示例：**

* 示例1  
输入: [100, 4, 200, 1, 3, 2]  
输出: 4  
解释: 最长连续序列是 [1, 2, 3, 4]。它的长度为 4。

**分析：**
* 我们维护一个hashMap，以当前的数值作为key，以该数值所在的最长连续序列的长度作为value，遍历一次数组，假设当前遍历到的数组中的值是n，我们做如下判断：

如果hashMap的key中存在n，则continue

如果hashMap的key中存在n-1，则hashMap中n对应的总长度是map.get(n-1)+1

如果hashMap的key中存在n+1, 则hashMap中n对应的总长度是map.get(n+1)+1

如果hashMap的key中既存在n+1也存在n-1, 则hashMap中n对应的总长度是map.get(n-1)+1+map.get(n+1)

这么做最大的好处在于我们不需要像题解那样用while遍历一遍去更改之前或之后的key所对应的总长度，现在我们只需要关心一个连续串的两端，只修改两个端点上的值，因为当前连续串的长度已经记录下来了，所以O(1)时间就能找到这个串的两端。如果用while遍历修改一遍的话在极端情况下时间复杂度会很大，但是用这个方法，除了大家在评论里说的，数据量大的时候哈希表查询耗时不是O(1)之外，其他就都没问题了。（大佬的分析）

```java
    public int longestConsecutive(int[] nums) {
        int max = 0;
        Map<Integer, Integer> map = new HashMap<>();

        for (int i =0; i < nums.length; i++) {
            int num = nums[i];
            if (map.containsKey(num)) {
                continue;
            }

            if (map.containsKey(num - 1) && !map.containsKey(num + 1)) {
                int length = map.get(num - 1) + 1;
                map.put(num, length);
                map.put(num - 1, length);
                max = Math.max(max, length);
            } else if (!map.containsKey(num - 1) && map.containsKey(num + 1)) {
                int length = map.get(num + 1) + 1;
                map.put(num, length);
                map.put(num + 1, length);
                max = Math.max(max, length);
            } else if (map.containsKey(num -1) && map.containsKey( num + 1)) {
                int length = map.get(num -1) + 1 + map.get(num + 1);
                map.put(num, length);
                map.put(num -map.get(num - 1), length);
                map.put(num +map.get(num + 1), length);
                max = Math.max(max, length);
            } else {
                map.put(num ,1);
                max = Math.max(max, 1);
            }
        }

        return max;
    }
```

Date : 2020.05.16  
From : LeetCode | 探索字节跳动.最长连续序列

