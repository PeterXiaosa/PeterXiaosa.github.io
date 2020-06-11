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
    }
```

Date : 2020.05.19  
From : LeetCode | 探索字节跳动.[朋友圈](https://leetcode-cn.com/problems/friend-circles/solution/peng-you-quan-by-leetcode/)  


##  合并区间   

**题目描述**


**说明：**  
* 给定 n 的范围是 [1, 9]。
* 给定 k 的范围是[1,  n!]。

**示例：**
* 示例1  
输入: [[1,3],[2,6],[8,10],[15,18]]  
输出: [[1,6],[8,10],[15,18]]  
解释: 区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].  

* 示例2  
输入: [[1,4],[4,5]]  
输出: [[1,5]]  
解释: 区间 [1,4] 和 [4,5] 可被视为重叠区间。  

**分析：**  


``` java
public int[][] merge(int[][] intervals) {
        // 先按照区间起始位置排序
        Arrays.sort(intervals, new Comparator<int[]>() {
            @Override
            public int compare(int[] o1, int[] o2) {
                return o1[0] - o2[0];
            }
        });
        // 遍历区间
        int[][] res = new int[intervals.length][2];
        int idx = -1;
        for (int[] interval: intervals) {
            // 如果结果数组是空的，或者当前区间的起始位置 > 结果数组中最后区间的终止位置，
            // 则不合并，直接将当前区间加入结果数组。
            if (idx == -1 || interval[0] > res[idx][1]) {
                res[++idx] = interval;
            } else {
                // 反之将当前区间合并至结果数组的最后区间
                res[idx][1] = Math.max(res[idx][1], interval[1]);
            }
        }
        return Arrays.copyOf(res, idx + 1);
    }
```

Date : 2020.05.20  
From : LeetCode | 探索字节跳动.[合并区间](https://leetcode-cn.com/explore/interview/card/bytedance/243/array-and-sorting/1046/)


##  反转链表  

**题目描述**

反转一个单链表。


**说明：**  
* 进阶:  
你可以迭代或递归地反转链表。


**示例：**

* 示例1  
输入: 1->2->3->4->5->NULL  
输出: 5->4->3->2->1->NULL  

**分析：**  
其实在做这道题目之前，应该先知道如何生成一个链表。链表生成有两种方法，头插法和尾插法。（此时应该想到Hashmap的扩容，在JDK1.7中采用头插法，多线程下会产生死锁。JDK1.8中采用尾插法，多线程下会覆盖旧数据）

```java
    public void addition_isCorrect() {
        ListNode node = new ListNode(1);
        node = addNode(node, 2);
        node = addNode(node, 3);
        node = addNode(node, 4);
        node = addNode(node, 5);

        ListNode result = reverseList(node);
        while (result != null) {
            System.out.println(result.val);
            result = result.next;
        }
    }

    /**
     *   尾插法
     * @param node
     * @param value
     * @return
     */
  public ListNode addNode(ListNode node, int value) {
      ListNode head, tail, newNode;
      if (node == null) {
            node = new ListNode(value);
            head = node;
        } else {
            head = tail = node;
            while (tail.next != null) {
                tail = tail.next;
            }
            newNode = new ListNode(value);
            newNode.next = null;
            tail.next = newNode;
        }

        return head;
  }

    public ListNode reverseList(ListNode head) {
              ListNode head, tail;
      head = tail = new ListNode(-1);
        Stack<Integer> stack = new Stack<>();
        while (node != null) {
            int value = node.val;
            stack.push(value);
            node = node.next;
        }

        while (!stack.isEmpty()) {
            while (tail.next != null) {
                tail = tail.next;
            }
            tail.next = new ListNode(stack.pop());
        }

        return head.next;
    }
```

Date : 2020.05.21  
From : LeetCode | 探索字节跳动.[反转链表](https://leetcode-cn.com/explore/interview/card/bytedance/244/linked-list-and-tree/1038/)



##  数组中重复的数字 

**题目描述**

在一个长度为n的数组里的所有数字都在0~n-1的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。例如，如果输入长度为7的数组{2,3,1,0,2,5,3}，那么对应的输出是重复的数字2或者3。


<!-- **说明：**  
* 进阶:  
你可以迭代或递归地反转链表。 -->


**示例：**

* 示例1  
输入: {2,3,1,0,2,5,3}  
输出: true

**分析：**  
首先，第一反应是可以将数组排序，排序之后依次对比数组中元素是否相等即可得出，时间复杂度为nlog(n)。其次，可以使用哈希表key-value，来存储数组中的元素，如果哈希表中没有数组值，那么就将数组元素添加进去。如果包含，那么也可知有重复元素。时间复杂度为n，但是多用了一个哈希表的空间。在不产生额外空间的情况下并且保持时间复杂度为n,可将数组中元素逐个扫描，从第0个开始。第0个元素是否等于i,如果相等扫码下一个元素。如果不等，将i和array[i]交换。再次进行比较array[i]是否和第array[i]个元素相等，如果相等返回true，如果不等继续。直到遍历完最后没有找到相等的元素即返回false。

```java
    public boolean duplicate(int[] array) {
        for (int i = 0; i < array.length; i++) {
            int value = array[i];
            if (value == i) {
                continue;
            }

            while (value != i) {
                if (value == array[value]) {
                    return true;
                } else {
                    int temp = array[value];
                    array[value] = value;
                    value = array[i] = temp;
                }
            }

        }
        return false;
    }
```

Date : 2020.05.25  
From : LeetCode | 《剑指offer》 面试题3：数组中重复的数字


##  重建二叉树 

**题目描述**

输入某二叉树的前序遍历和中序遍历的结果，请重建该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如，输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6},则重建二叉树并输入它的头结点。


<!-- **说明：**  
* 进阶:  
你可以迭代或递归地反转链表。 -->


<!-- **示例：**

* 示例1  
输入: {2,3,1,0,2,5,3}  
输出: true -->

**分析：**  
前序遍历序列的第一个数字必为二叉树的根节点。然后在中序遍历序列中遍历找到根节点的位置index。中序遍历序列中在index左侧是根节点左子树的值(根节点左子数的中序遍历序列)，index右边是根节点右子树的值(右子树的中序遍历序列)。而前序遍历序列的1-index为则为根节点左子树的值（根节点左子树的前序遍历序列），inde+1-end位为根节点右子树的值（根节点右子树的前序遍历序列）。这样我们又分别知道了根节点的左右字数的前序遍历序列和中序遍历序列，使用递归即可得出该二叉树。

```java
    class BinaryTree {
        int value;
        BinaryTree leftTree;
        BinaryTree rightTree;
    }

    BinaryTree getBinaryTree(int[] front, int[] middle) {
        BinaryTree binaryTree = null;
        if (front.length == 0 || middle.length == 0)
            return binaryTree;
        binaryTree = new BinaryTree();

        int root = front[0];
        binaryTree.value = root;
        int index = getIndex(middle, root);

        if (index == 0) {
            // 无左节点
            binaryTree.leftTree = null;
            binaryTree.rightTree = getBinaryTree(Arrays.copyOfRange(front, 1, front.length), Arrays.copyOfRange(middle, 1, middle.length));
        } else if (index == middle.length - 1){
            // 无右节点
            binaryTree.rightTree = null;
            binaryTree.leftTree = getBinaryTree(Arrays.copyOfRange(front, 1, front.length), Arrays.copyOfRange(middle, 0, middle.length -1));
        } else {
            binaryTree.leftTree = getBinaryTree(Arrays.copyOfRange(front, 1, index + 1), Arrays.copyOfRange(middle, 0, index));
            binaryTree.rightTree = getBinaryTree(Arrays.copyOfRange(front, index + 1, front.length), Arrays.copyOfRange(middle, index + 1, middle.length));
        }

        return binaryTree;
    }

    int getIndex(int[] array, int n) {
        for (int i =0; i < array.length;i++) {
            if (n == array[i]) {
                return i;
            }
        }
        return -1;
    }
```

Date : 2020.05.28  
From : LeetCode | 《剑指offer》 面试题7：重建二叉树


##  旋转数组 

**题目描述**

给定一个数组，将数组中的元素向右移动 k 个位置，其中 k 是非负数。


 **说明：**  
* 尽可能想出更多的解决方案，至少有三种不同的方法可以解决这个问题。
* 要求使用空间复杂度为 O(1) 的 原地 算法。


 **示例：**

* 示例1  
输入: [1,2,3,4,5,6,7] 和 k = 3  
输出: [5,6,7,1,2,3,4]  
解释:  
向右旋转 1 步: [7,1,2,3,4,5,6]  
向右旋转 2 步: [6,7,1,2,3,4,5]  
向右旋转 3 步: [5,6,7,1,2,3,4] 

* 示例2  
输入: [-1,-100,3,99] 和 k = 2  
输出: [3,99,-1,-100]  
解释:   
向右旋转 1 步: [99,-1,-100,3]  
向右旋转 2 步: [3,99,-1,-100]  

**分析：**  


```java
    public void rotate(int[] nums, int k) {
        k = k % nums.length;
        reverse(nums, 0, nums.length -1);
        reverse(nums, 0 , k -1);
        reverse(nums, k, nums.lengtn -1);
    }

    private void reverse(int[] nums, int start, int stop){
        if(nums == null || start > stop) {
            return;
        }
        int first = start;
        int end = stop;

        while(first <= end) {
            int temp = nums[first];
            nums[first] = nums[end];
            nums[end] = temp;
            first++;
            end--;
        }
    }
```

Date : 2020.05.29  
From : LeetCode | 189.旋转数组

##  二叉树的层序遍历 

**题目描述**  

给你一个二叉树，请你返回其按 层序遍历 得到的节点值。 （即逐层地，从左到右访问所有节点）。  


<!-- **说明：**  
* 进阶:  
你可以迭代或递归地反转链表。 -->


 **示例：**

* 示例1  
二叉树：[3,9,20,null,null,15,7],  

    3  
   / \  
  9  20  
    /  \  
   15   7  
返回其层次遍历结果：  

[  
  [3],  
  [9,20],  
  [15,7]  
]
 

**分析：**  
二叉树的层序遍历，使用队列。先将根节点加入队列，然后依次输出队列中的结点。将节点的值加入list,同时将每个节点的左右子节点加入队列，直到队列中无值，输出list，即可得二叉树层序遍历结果。

```java

    public class TreeNode {
        int val = 0;
        TreeNode left = null;
        TreeNode right = null;
        public TreeNode(int val) {
            this.val = val;
        }
    }

    /**
    不需考虑层数，只需要依次输出二叉树层序遍历结果
    **/
    public ArrayList<Integer> PrintFromTopToBottom(TreeNode root) {
        ArrayList<Integer> resultList = new ArrayList();
        if(root == null) {
            return resultList;
        }
        Queue<TreeNode> queue = new LinkedList();
        queue.offer(root);
        while(!queue.isEmpty()) {
            TreeNode node = queue.poll();
            if(node == null) {
                continue;
            }
            resultList.add(node.val);
            if(node.left != null) {
                queue.offer(node.left);
            }
            if(node.right != null) {
                queue.offer(node.right);
            }
        }
        return resultList;
    }

    /**
    需要将二叉树层序遍历，根据层数输出来，思路和上述方法相似，只是需要每次在每层时遍历将该层所有节点的值加入到这一层的List中。
    **/

    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> resultList = new ArrayList();
        if(root == null) {
            return resultList;
        }

        Queue<TreeNode> queue = new LinkedList();
        queue.offer(root);
        while(!queue.isEmpty()) {
            int size = queue.size();
            List<Integer> list = new ArrayList();
            for(int i = 0; i < size; i++) {
                TreeNode node = queue.poll();
                if(node == null) {
                    continue;
                }
                list.add(node.val);
                if(node.left != null) {
                    queue.offer(node.left);
                }
                if(node.right != null) {
                    queue.offer(node.right);
                }
            }
            resultList.add(list);
        }
        return resultList;
    }
```

Date : 2020.05.30  
From : LeetCode | 102.二叉树的层序遍历

##  二叉树的右视图 

**题目描述**  

给定一棵二叉树，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。


<!-- **说明：**  
* 进阶:  
你可以迭代或递归地反转链表。 -->


 **示例：**

* 示例1  
输入: [1,2,3,null,5,null,4]  
输出: [1, 3, 4]  
解释:

   1            <---
 /   \
2     3         <---
 \     \
  5     4       <---
 

**分析：**  
右视图，也及根右左的遍历，和前序根左右相反，所以可采用深度优先搜索。  
另外一种方法，可广度搜索，例如二叉树层序遍历，然后每层输出最后一个数字。

```java
class Solution {
    List<Integer> res = new ArrayList();

    public List<Integer> rightSideView(TreeNode root) {
        dfs(root, 0);
        return res;
    }

    private void dfs(TreeNode node, int depth) {
        if(node == null) {
            return;
        }
        if(res.size == depth) {
            res.add(node.val);
        }
        depth++;
        dfs(node, depth);
        dfs(node, depth);
    }
}
```

Date : 2020.05.30  
From : LeetCode | 199.二叉树的右视图


##  每日温度 

**题目描述**  

请根据每日 气温 列表，重新生成一个列表。对应位置的输出为：要想观测到更高的气温，至少需要等待的天数。如果气温在这之后都不会升高，请在该位置用 0 来代替。


<!-- **说明：**  
* 进阶:  
你可以迭代或递归地反转链表。 -->


 **示例：**

例如，给定一个列表 temperatures = [73, 74, 75, 71, 69, 72, 76, 73]，你的输出应该是 [1, 1, 4, 2, 1, 1, 0, 0]。
 

**分析：**  
可以维护一个存储下标的单调栈，从栈底到栈顶的下标对应的温度列表中的温度依次递减。如果一个下标在单调栈里，则表示尚未找到下一次温度更高的下标。

```java
    public int[] dailyTemperatures(int[] T) {
        LinkedList<Integer> stack = new LinkedList<>();
        
        int n = T.length;
        int[] result = new int[n];
        
        for(int i = 0; i < n; i++) {
            
            while(!stack.isEmpty() && T[i] > T[stack.peek()]) {
                int index = stack.pop();
                result[index] = i - index;
            }
            stack.push(i);
        }
        return result;
    }
```

时间复杂度：O(n)  
空间复杂度：O(n)

Date : 2020.06.11  
From : LeetCode | 739.每日温度