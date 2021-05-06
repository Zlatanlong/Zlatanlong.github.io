---
title: 简单理解KMP代码
typora-copy-images-to: ./img
typora-root-url: ./img
categories:
  - 算法
---

今天的每日一题是[28. 实现 strStr()](https://leetcode-cn.com/problems/implement-strstr/)，这竟然是一道简单题，好吧学习一下KMP算法，对于KMP算法过程的讲解，在leetcode题解中[【宫水三叶】简单题学 KMP 算法](https://leetcode-cn.com/problems/implement-strstr/solution/shua-chuan-lc-shuang-bai-po-su-jie-fa-km-tb86/)的图解，还有知乎的回答[如何更好地理解和掌握 KMP 算法? - 阮行止的回答 - 知乎](https://www.zhihu.com/question/21923021/answer/1032665486)已经颇为详细，我这里只是简单对代码进行一下简述，方便理解(bèi sòng)。

```java
public static int kmp(String ss, String pp) {
        if (pp.isEmpty()) return 0;

        int n = ss.length(), m = pp.length();
    	// 前面加一个空格，后面匹配时下标表示个数，同时避免 j = next[j-1] 时 j-1=-1
        ss = " " + ss;
        pp = " " + pp;
        char[] s = ss.toCharArray();
        char[] p = pp.toCharArray();
        int[] next = new int[m + 1];

        /**
         * 下面两个过程很像，求next其实就是自己和自己匹配，
         * 无论如何 匹配失败 j=next[j];
         * 失败条件是(j > 0 && 数组[i] != p[j + 1])
         * 成功时将j向前移
         * 每次都要进行的是：
         *  求next： 对next数组进行赋值
         *  匹配： 判断是否匹配
         *  i++： 这一步在for循环中进行
         *
         * 求next自己与自己匹配，只有1位时不考虑，所以i从2开始
         * 匹配时 i 从 1开始
         *
         */


        // 求next过程
        for (int i = 2, j = 0; i <= m; i++) {
            while (j > 0 && p[i] != p[j + 1]) j = next[j];
            if (p[i] == p[j + 1]) j++;
            next[i] = j;
        }

        // 匹配过程
        for (int i = 1, j = 0; i <= n; i++) {
            while (j > 0 && s[i] != p[j + 1]) j = next[j];
            if (s[i] == p[j + 1]) j++;
            if (j == m) return i - m;
        }
        return -1;
    }
```

