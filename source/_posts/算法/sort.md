---
title: 常用排序算法
date: 2021-05-11 08:49:35
categories:
  - 算法
---

前端可视化学习网站：https://www.cs.usfca.edu/~galles/visualization/ComparisonSort.html

结合图可以更好地理解算法。

### 插入排序

插入排序虽然时间复杂度也是O(n^2)，但由于其在少数据时表现较好，在归并排序和快排中可以在分治规划的一组数据值较少时使用，因此也需要掌握。

```java
public class Solution {
// 插入排序：稳定排序，在接近有序的情况下，表现优异
    public int[] sortArray(int[] nums) {
        int len = nums.length;
        // 循环不变量：将 nums[i] 插入到区间 [0, i) 使之成为有序数组
        for (int i = 1; i < len; i++) {
            // 先暂存这个元素，然后之前元素逐个后移，留出空位
            int temp = nums[i];/
            int j = i;
            // 注意边界 j > 0
            while (j > 0 && nums[j - 1] > temp) {
                nums[j] = nums[j - 1];
                j--;
            }
            nums[j] = temp;
        }
        return nums;
    }
}
```
<!--more-->

### 归并排序

分治的思想，二分法从中间分，先分后和，双指针合并即可。

```java
public class Solution {
    // 归并排序

    /**
     * 列表大小等于或小于该大小，将优先于 mergeSort 使用插入排序
     */
    private static final int INSERTION_SORT_THRESHOLD = 7;

    public int[] sortArray(int[] nums) {
        int len = nums.length;
        int[] temp = new int[len];
        mergeSort(nums, 0, len - 1, temp);
        return nums;
    }

    /**
     * 对数组 nums 的子区间 [left, right] 进行归并排序
     *
     * @param nums
     * @param left
     * @param right
     * @param temp  用于合并两个有序数组的辅助数组，全局使用一份，避免多次创建和销毁
     */
    private void mergeSort(int[] nums, int left, int right, int[] temp) {
        // 小区间使用插入排序
        if (right - left <= INSERTION_SORT_THRESHOLD) {
            insertionSort(nums, left, right);
            return;
        }

        int mid = left + (right - left) / 2;
        // Java 里有更优的写法，在 left 和 right 都是大整数时，即使溢出，结论依然正确
        // int mid = (left + right) >>> 1;

        mergeSort(nums, left, mid, temp);
        mergeSort(nums, mid + 1, right, temp);
        // 如果数组的这个子区间本身有序，无需合并
        // nums[mid]是左边最大的
        // num[mid+1]是右边最小的
        if (nums[mid] <= nums[mid + 1]) {
            return;
        }
        mergeOfTwoSortedArray(nums, left, mid, right, temp);
    }

    /**
     * 对数组 arr 的子区间 [left, right] 使用插入排序
     *
     * @param arr   给定数组
     * @param left  左边界，能取到
     * @param right 右边界，能取到
     */
    private void insertionSort(int[] arr, int left, int right) {
        for (int i = left + 1; i <= right; i++) {
            int temp = arr[i];
            int j = i;
            while (j > left && arr[j - 1] > temp) {
                arr[j] = arr[j - 1];
                j--;
            }
            arr[j] = temp;
        }
    }

    /**
     * 合并两个有序数组：先把值复制到临时数组，再合并回去
     *
     * @param nums
     * @param left
     * @param mid   [left, mid] 有序，[mid + 1, right] 有序
     * @param right
     * @param temp  全局使用的临时数组
     */
    private void mergeOfTwoSortedArray(int[] nums, int left, int mid, int right, int[] temp) {
        System.arraycopy(nums, left, temp, left, right + 1 - left);

        int i = left;
        int j = mid + 1;

        for (int k = left; k <= right; k++) {
            if (i == mid + 1) {
                nums[k] = temp[j];
                j++;
            } else if (j == right + 1) {
                nums[k] = temp[i];
                i++;
            } else if (temp[i] <= temp[j]) {
                // 注意写成 < 就丢失了稳定性（相同元素原来靠前的排序以后依然靠前）
                nums[k] = temp[i];
                i++;
            } else {
                // temp[i] > temp[j]
                nums[k] = temp[j];
                j++;
            }
        }
    }
}
```

### 快速排序

常见的快排有三种，主要是在**partition**时对与基准值相同的值如何处理的不同进行了改善

- 基本快排，相同值完全落到一端，可能造成递归树倾斜。
- 双指针，相同值等概率落到两端
- 三指针，相同值集中在pivot值两边

#### 基本快速排序

```java
import java.util.Random;

public class Solution {

    // 快速排序 1：基本快速排序

    /**
     * 列表大小等于或小于该大小，将优先于 quickSort 使用插入排序
     */
    private static final int INSERTION_SORT_THRESHOLD = 7;

    private static final Random RANDOM = new Random();


    public int[] sortArray(int[] nums) {
        int len = nums.length;
        quickSort(nums, 0, len - 1);
        return nums;
    }

    private void quickSort(int[] nums, int left, int right) {
        // 小区间使用插入排序
        if (right - left <= INSERTION_SORT_THRESHOLD) {
            insertionSort(nums, left, right);
            return;
        }

        int pIndex = partition(nums, left, right);
        quickSort(nums, left, pIndex - 1);
        quickSort(nums, pIndex + 1, right);
    }

    /**
     * 对数组 nums 的子区间 [left, right] 使用插入排序
     *
     * @param nums  给定数组
     * @param left  左边界，能取到
     * @param right 右边界，能取到
     */
    private void insertionSort(int[] nums, int left, int right) {
        for (int i = left + 1; i <= right; i++) {
            int temp = nums[i];
            int j = i;
            while (j > left && nums[j - 1] > temp) {
                nums[j] = nums[j - 1];
                j--;
            }
            nums[j] = temp;
        }
    }

    private int partition(int[] nums, int left, int right) {
        // 去一个left到right的随机数，然后放到最左边
        int randomIndex = RANDOM.nextInt(right - left + 1) + left;
        swap(nums, left, randomIndex);

        // 基准值（随机选出）
        int pivot = nums[left];
        int lt = left;
        // 循环不变量：
        // all in [left + 1, lt] < pivot
        // all in [lt + 1, i) >= pivot
        for (int i = left + 1; i <= right; i++) {
            if (nums[i] < pivot) {
                lt++;
                swap(nums, i, lt);
            }
        }
        swap(nums, left, lt);
        return lt;
    }

    private void swap(int[] nums, int index1, int index2) {
        int temp = nums[index1];
        nums[index1] = nums[index2];
        nums[index2] = temp;
    }
}
```

#### 双指针：推荐

在jdk1.8源码中默认采用这种方式。

```java
private int partition(int[] nums, int left, int right) {
    int randomIndex = left + RANDOM.nextInt(right - left + 1);
    swap(nums, randomIndex, left);

    int pivot = nums[left];
    int lt = left + 1;
    int gt = right;

    // 循环不变量：
    // all in [left + 1, lt) <= pivot
    // all in (gt, right] >= pivot
    while (true) {
        while (lt <= right && nums[lt] < pivot) {
            lt++;
        }

        while (gt > left && nums[gt] > pivot) {
            gt--;
        }

        if (lt >= gt) {
            break;
        }

        // 细节：相等的元素通过交换，等概率分到数组的两边
        swap(nums, lt, gt);
        lt++;
        gt--;
    }
    swap(nums, left, gt); // gt交换时，这个值一定是<=pivot的
    return gt;
}
```

