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
我们维护一个hashMap，以当前的数值作为key，以该数值所在的最长连续序列的长度作为value，遍历一次数组，假设当前遍历到的数组中的值是n，我们做如下判断：  

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


##  合并两个有序链表

**题目描述**

将两个升序链表合并为一个新的升序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。   


**说明：**  
无

**示例：**

* 示例1  
输入：1->2->4, 1->3->4  
输出：1->1->2->3->4->4

**分析：**
* 首先，我们设定一个哨兵节点 prehead ，这可以在最后让我们比较容易地返回合并后的链表。我们维护一个 prev 指针，我们需要做的是调整它的 next 指针。然后，我们重复以下过程，直到 l1 或者 l2 指向了 null ：如果 l1 当前节点的值小于等于 l2 ，我们就把 l1 当前的节点接在 prev 节点的后面同时将 l1 指针往后移一位。否则，我们对 l2 做同样的操作。不管我们将哪一个元素接在了后面，我们都需要把 prev 向后移一位。  
在循环终止的时候， l1 和 l2 至多有一个是非空的。由于输入的两个链表都是有序的，所以不管哪个链表是非空的，它包含的所有元素都比前面已经合并链表中的所有元素都要大。这意味着我们只需要简单地将非空链表接在合并链表的后面，并返回合并链表即可。（大佬的分析）

```java

     public static class ListNode {
         int val;
         ListNode next;
         ListNode(int x) { val = x; }
    }

    // 递归，需考虑栈的深度是否会溢出
    public static ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        } else if (l2 == null) {
            return l1;
        }

        if (l1.val <= l2.val) {
            l1.next = mergeTwoLists(l1.next, l2);
            return l1;
        } else {
            l2.next = mergeTwoLists(l1, l2.next);
            return l2;
        }
    }

    public static ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode prehead = new ListNode(-1);
        ListNode prev = prehead;
        while (l1 != null && l2 != null) {
            if (l1.val <= l2.val) {
                prev.next = l1;
                l1 = l1.next;
            } else {
                prev.next = l2;
                l2 = l2.next;
            }
            prev = prev.next;
        }

        prev.next = l1 == null ? l2 : l1;
        return prehead.next;
    }
```

Date : 2020.05.17  
From : LeetCode | 探索字节跳动.合并两个有序链表


##  第k个排列

**题目描述**

给出集合 [1,2,3,…,n]，其所有元素共有 n! 种排列。  
按大小顺序列出所有排列情况，并一一标记，当 n = 3 时, 所有排列如下：

"123"  
"132"  
"213"  
"231"  
"312"  
"321"  
给定 n 和 k，返回第 k 个排列。


**说明：**  
* 给定 n 的范围是 [1, 9]。
* 给定 k 的范围是[1,  n!]。

**示例：**

* 示例1  
输入: n = 3, k = 3  
输出: "213"

* 示例2  
输入: n = 4, k = 9  
输出: "2314"

**分析：**
  通过使用k / (n-1)! 得到除数和mod余数。通过判断除数和mod余数确定当前list中的数值，添加之后remove list中的该值。然后n = n-1; k = mod，继续之前的逻辑。

```java

    public String getPermutation(int n, int k) {
        StringBuilder builder = new StringBuilder();
        List<Integer> list = new ArrayList<>();
        int temp = k;

        for (int i = 1; i <= n; i++) {
            list.add(i);
        }

        for (int i = n; i >= 1; i--) {
            if (list.size() == 1) {
                builder.append(list.get(0));
                break;
            }
            int j = i -1;
            int fatal = 1;
            while (j > 0) {
                fatal = fatal * j;
                j--;
            }
            int result = temp / fatal;
            int mod = temp - result * fatal;

            if (mod == 0) {
                builder.append(list.get(result -1));
                list.remove(list.get(result -1));
                int length = list.size();
                for (j = length; j>0;j--) {
                    builder.append(list.get(list.size() -1));
                    list.remove(list.size() -1);
                }
                break;
            } else {
                builder.append(list.get(result));
                list.remove(list.get(result));
            }
            temp = mod;
        }

        return builder.toString();
    }
```

Date : 2020.05.18  
From : LeetCode | 探索字节跳动.第k个排列



##  朋友圈  

**题目描述**

班上有 N 名学生。其中有些人是朋友，有些则不是。他们的友谊具有是传递性。如果已知 A 是 B 的朋友，B 是 C 的朋友，那么我们可以认为 A 也是 C 的朋友。所谓的朋友圈，是指所有朋友的集合。

给定一个 N * N 的矩阵 M，表示班级中学生之间的朋友关系。如果M[i][j] = 1，表示已知第 i 个和 j 个学生互为朋友关系，否则为不知道。你必须输出所有学生中的已知的朋友圈总数。


**说明：**  
* 给定 n 的范围是 [1, 9]。
* 给定 k 的范围是[1,  n!]。

**示例：**

* 示例1  
输入: 
[[1,1,0],  
 [1,1,0],  
 [0,0,1]]  
输出: 2   
说明：已知学生0和学生1互为朋友，他们在一个朋友圈。
第2个学生自己在一个朋友圈。所以返回2。

* 示例2  
输入:   
[[1,1,0],  
 [1,1,1],  
 [0,1,1]]  
输出: 1  
说明：已知学生0和学生1互为朋友，学生1和学生2互为朋友，所以学生0和学生2也是朋友，所以他们三个在一个朋友圈，返回1。  

注意：

N 在[1,200]的范围内。
对于所有学生，有M[i][i] = 1。
如果有M[i][j] = 1，则有M[j][i] = 1。

**分析：**  
将朋友圈问题转化为使用深度优先搜索，查找无向图连通块的个数。从每个未被访问的节点开始深搜，每开始一次搜索就增加couont计数器一次。count数即为连通块数。

```java

// (有点难，还是没太懂深度优先搜索算法)
public void dfs(int[][] M, int[] visited, int i) {
        for (int j = 0; j < M.length; j++) {
            if (M[i][j] == 1 && visited[j] == 0) {
                visited[j] = 1;
                dfs(M, visited, j);
            }
        }
    }
    public int findCircleNum(int[][] M) {
<<<<<<< HEAD
        if (M == null) {
            return 0;
        }
        if (M.length < 2) {
            return M.length;
        }

        List<Integer> list = new ArrayList<>();
        int length = M.length;
        for (int x = 0; x < length; x++) {
            for (int y = x +1; y < length; y++) {
                if (list.contains(y)) {
                    continue;
                }

                int value = M[x][y];
                if (value == 1) {
                    list.add(y);
                    if (list.size() == length -1) {
                        return 1;
                    }
                }
            }
        }

        return length - list.size();
=======
        int[] visited = new int[M.length];
        int count = 0;
        for (int i = 0; i < M.length; i++) {
            if (visited[i] == 0) {
                dfs(M, visited, i);
                count++;
            }
        }
        return count;
>>>>>>> e6628c2053ce989bb10529f6a6f968acf0e4dd3f
    }
```

Date : 2020.05.19  
From : LeetCode | 探索字节跳动.[朋友圈](https://leetcode-cn.com/problems/friend-circles/solution/peng-you-quan-by-leetcode/)
