title: KMP字符串匹配（java）
author: 禾田
tags:
  - 算法
categories:
  - 算法
date: 2018-06-22 19:45:00
---
### KMP算法介绍

参考：[字符串匹配的KMP算法](https://blog.csdn.net/u013289254/article/details/60597339)


### 问题：

> 
对于一个给定的 source 字符串和一个 target 字符串，你应该在 source 字符串中找出 target 字符串出现的第一个位置(从0开始)。如果不存在，则返回 -1。

样例  
如果 source = "source" 和 target = "target"，返回 -1。

如果 source = "abcdabcdefg" 和 target = "bcd"，返回 1。


### 暴力解法

```java
public int strStr(String source, String target) {
    // write your code here
    if (source == null) {
        return -1;
    }
    if (target == null) {
        return -1;
    }
    if (target.length() == 0) {
        return 0;
    }
    int sourceLength = source.length();
    int targetLength = target.length();
    char[] sourceCharArray = source.toCharArray();
    char[] targetCharArray = target.toCharArray();
    for (int i = 0; i < sourceLength; ++i) {
        if (sourceCharArray[i] == targetCharArray[0]) {
            int tempI = i;
            for (int j = 0; j < targetLength && tempI < sourceLength; ++j, ++tempI) {
                if (sourceCharArray[tempI] != targetCharArray[j]) {
                    break;
                }
            }
            if (tempI == (i + targetCharArray.length)) {
                return i;
            }
        }
    }
    return -1;
}
```

### KMP算法

```java
// 获取字符串部分匹配表
public int[] getPartMatchTable(String str) {
    if (str == null || str.length() == 0) {
        return null;
    }
    int length = str.length();
    char[] charArray = str.toCharArray();
    int[] table = new int[str.length()];
    int j = 0;
    int i = 1;
    int count = 0;
    table[0] = 0;
    while (i < length) {
        if (charArray[i] == charArray[j]) {
            ++count;
            table[i] = count;
            ++i;
            ++j;
        } else {
            count = 0;
            table[i] = count;
            j = 0;
            ++i;
        }
    }
    return table;
}

public int strStr(String source, String target) {
    // write your code here
    if (source == null) {
        return -1;
    }
    if (target == null) {
        return -1;
    }
    if (target.length() == 0) {
        return 0;
    }
    int sourceLength = source.length();
    int targetLength = target.length();
    char[] sourceCharArray = source.toCharArray();
    char[] targetCharArray = target.toCharArray();
    // 获取字符串部分匹配表
    int[] table = getPartMatchTable(target);
    // 遍历标记
    int i = 0;
    // 记录已经匹配的字符数
    int count = 0;
    // 定义临时标记
    int tempI = 0;
    while (i < sourceLength) {
        if (sourceCharArray[i] == targetCharArray[0]) {
            int j = 0;
            tempI = i;
            while (j < targetLength && tempI < sourceLength) {
                if (sourceCharArray[tempI] == targetCharArray[j]) {
                   ++count;
                } else {
                    break;
                }
                ++tempI;
                ++j;
            }
            if (tempI == (i + targetCharArray.length)) {
                return i;
            } else {
                // 关键代码
                i = i + (count - table[j]);
                count = 0;
            }
        } else {
            ++i;
        }
    }
    return -1;
}
```