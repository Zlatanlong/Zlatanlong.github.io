---
title: hanoi塔算法理解
typora-copy-images-to: ../../source/assets
typora-root-url: ../../source
date: 2020-02-24 21:55:06
updated:
tags:
categories:
---

最近回忆起一个经典的递归题，汉诺塔，捋顺一下思路和代码。

前提：用abc代表汉诺塔的三根柱子，要求从a搬到c。

首先抽象出一个方法`hanoi(n, ori, mid, last)`,参数n是共计n层hanoi，ori是最初在哪根上，last是最后移到哪根上。

<!--more-->

### 现在聚焦a=>c过程中最底下的盘子n，将整个搬移步骤分位三步：

1. 将上面共n-1层一起移动到柱b，这样就会呈现a柱只有第n层，b柱有共n-1层，c柱没有
2. 将n层从a=>c

3. 再将b上的共n-1层移动到c。

### 上面三步正好可以转化成如下的动作：

1. hanoi(n-1, a, c, b)
2. a=>c
3. hanoi(n-1, b, a, c)

具体的思考过程还可以通过n不断变大来递推，上面两段相当于直接告诉了提示的答案。

## 递归小练习

为了趁热打铁，想起了一个好的例子，比如之前配置的`tree`命令，打印出当前目录下所有文件，这个就是一个递归的好例子。废话不多说看代码

这里levelCount 代表以当前文件夹等级为第一级，每一代子文件加一级，这样就可以根据等级显示|有多少个了。

```java
import java.io.File;
import java.util.Scanner;

public class DirTree {
    public int levelCount = 1;

    /**
     * print this line's information, use | as file level
     * @param str file name
     */
    public void printLine(String str) {
        StringBuffer thisLine = new StringBuffer();
        thisLine.append("|");
        for (int i = 1; i < levelCount - 1; i++) {
            thisLine.append(" |");
        }
        thisLine.append("--" + str);
        System.out.println(thisLine);
    }

    /**
     * the recursive function
     * @param fPath a absolutely file path
     */
    public void getDir(String fPath) {
        File currentFile = new File(fPath);
        if (currentFile.isDirectory()) {
            String[] childrenFileNameLists = currentFile.list();
            if (childrenFileNameLists.length > 0) {
                levelCount++;
                for (String childrenFileName : childrenFileNameLists) {
                    File childrenFile = new File(fPath + "/" + childrenFileName);
                    printLine(childrenFileName);
                    if (childrenFile.isDirectory()) {
                        getDir(fPath + "/" + childrenFileName);
                    }
                }
                levelCount--;
                return;
            } else {
                return;
            }
        } else {
            System.out.println("不是目录");
            return;
        }
    }

    public static void main(String[] args) {
        DirTree dirTree = new DirTree();
        System.out.println("请输入想打印的文件目录");
        Scanner sc = new Scanner(System.in);
        String filePath = sc.nextLine();

        System.out.println(filePath);
        System.out.println(".");
        dirTree.getDir(filePath);
    }
}
```

运行效果如下:

![结果](48697D36-D378-4CB9-ABA0-49D53A00270F.png)

和文件结构一样，运行成功。