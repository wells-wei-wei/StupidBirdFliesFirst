<!--
 * @Author: your name
 * @Date: 2020-07-11 20:43:20
 * @LastEditTime: 2020-07-11 21:03:22
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \StupidBirdFliesFirst\DataStructure\string.md
--> 

# 字符串

串是一种数据对象和操作都特殊的线性表

不同之处在于串针对的是字符集，也就是串中的元素都是字符。对于串的基本操作与线性表是有很大的差别的。线性表更关注的是单个元素的操作，比如说查找一个元素，插入或者删除一个元素，但串中更多的是查找子串位置，得到指定子串，替换子串等操作。

## 非平凡子串
指非空且不同于字符串本身的子串。

设S为一个长度为n的字符串,其中的字符各不相同,则S中的互异的非平凡子串(非空且不同于S本身)的个数为(n²/2)+(n/2)-1

## KMP算法
Knuth-Morris-Pratt字符串查找算法（简称为KMP算法）可在一个主文本字符串S内查找一个词W的出现位置。

[算法详情](http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html)

KMP算法时间复杂度为O(m+n)，空间复杂度为O(m)。

## 空字符串
空字符串指的就是没有东西的字符串，和全都是空格的字符串是不一样的

## 模式匹配
模式匹配是数据结构中字符串的一种基本运算，给定一个子串，要求在某个字符串中找出与该子串相同的所有子串，这就是模式匹配。