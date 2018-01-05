---
layout: post
title: 关于数组的最长递增子序列的一个突发奇想
category: Java
tags: java
description: 算法
---

### 数组的最长递增子序列

这个问题就是要找出一个数组的一个子序列，子序列与子串不同，子序列中的元素在原数组中可以是不连续的几个元素，而子串中的元素在原数组中也是连续的，比如：
数组：3,4,5,1,2,7,9
它的最长递增子序列是3,4,5,7,9
而最长递增子串是1,2,7,9

问题的解法有多种，穷举、贪心、动态规划、分治法
然而这些算法我都没看懂，不懂人家的核心思想是什么。不过思考的过程中闪过了一个想法，记录在这里，代码贴在下面。
核心思路就是，先把原数组分解成多个链表，然后在将相关的链表连接起来，组成更长的链表，最终找到最长的链表就可以了。
算法实现的注意点：
1. 只要一个元素(a[n])的前一个元素比他大(a[n-1] > a[n])，他(a[n])就是一个链表的头节点。
2. 在第一次遍历初始化完成的所有链表中，可能会有不同的节点指向同一个节点，但是不会出现一个节点指向不同的下一个节点，所以在循环构建节点的时候一定要判断当前位置的节点是否为null，不是null就直接引用，是null才去创建。
3. 最后在拼接链表的时候要注意，我们要做的就是，判断一个头节点元素的前面有没有比它小的值，如果有，那就看看那个元素所在的链表的尾部长度是否比当前的头节点所在的链表长，在计算两个长度的时候，头节点所在的链表取的是链表的整体长度；另一个链表取的是在那个元素(那个节点不算在内)之后的链表的长度。

更多的解释在代码的注释中：
```java
import java.lang.reflect.Array;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class Lis {
    /*
    * 数组如下
    * 1,4,2,3,5,0,7,9,8,9
    * 找到最长递增子序列
    * */
    public static void main(String... args) {
        int[] array = {1,4,2,3,5,0,7,9,8,9};
        int len = lis(array);
        System.out.println("[1,4,2,3,5,0,7,9,8,9]的最长子序列长度是"+len);
    }

    private static int lis(int[] arr) {
        if (arr == null || arr.length == 0) return 0;
        int length = arr.length; //数组长度
        Entry[] list = new Entry[length]; //将原数组转化为一一对应的Entry数组，Entry为链表的节点
        List<Entry> heads = new ArrayList<>(); //Entry链表的头结点，因为会有多个链表，储存他们的头结点便于之后使用
        for (int i = 0; i < length; i++) {
            //System.out.println(i + ": " + list[i]);
            if (list[i] == null) {
                list[i] = new Entry();
                list[i].value = arr[i];
                list[i].index = i;
                if (i == 0 || arr[i - 1] > arr[i]) { //如果一个元素的前一个元素值比他大，那么他就是一个链表的头结点
                    list[i].isHead = true;
                    heads.add(list[i]);
                }
                //System.out.println(i + ": " + list[i]);
                int j = 1;
                for (int n = i; n < length;) {
                    //从当前的头结点向后查找，比他大的第一个元素作为链表的第二个结点，
                    //继续向后遍历，第一个比第二结点大的元素作为链表的第三结点，以此类推
                    //这种构建链表的方式使得数组后面的元素可能会包含在多个链表中
                    //如当前的例子中初始化后的链表有：
                    //1 4 5 7 9
                    //2 3 5 7 9
                    //0 7 9
                    //8 9
                    if (n + j < length && arr[n] < arr[n + j]) {
                        if (list[n + j] == null) { //为了保证链表的正确，最好不要重复构建结点，容易导致结点的next结点错误丢失
                            list[n + j] = new Entry();
                            list[n + j].value = arr[n + j];
                            list[n + j].isHead = false;
                            list[n + j].index = n + j;
                        }
                        list[n].next = list[n + j];
                        n = n + j;
                        j = 1;
                        System.out.println(n + ": " + list[n]);
                    } else if (n + j >= length){
                        break;
                    } else {
                        j++;
                    }
                }
            }
        }
        List<Entry> temp = new ArrayList<>();
        System.out.println("heads = " + heads);
        for (int i = 1; i < heads.size(); i++) {
            //System.out.println(heads);
            printList(heads.get(i));
            //检查每一个链表，如果头结点之前的元素中有更小的值，说明有更小的值可以替换这个头结点，让链表更长，
            //将这个头结点所在的链表的长度，与那个更小的元素所在的链表的尾部长度比较，
            //如果前者大于后者，则将这个头结点链接到前面的链表的那个小元素之后，并去掉头结点标志
            for (int j = heads.get(i).index; j >= heads.get(i - 1).index; j--) {
                if (heads.get(i).value > arr[j]) {
                    //System.out.println("heads["+i+"] = " + heads.get(i).value + ", arr["+j+"] = "+arr[j]);
                    if (list[j].tailLength() - 1 < heads.get(i).tailLength()) {
                        list[j].next = heads.get(i);
                        heads.get(i).isHead = false;
                        temp.add(heads.get(i));
                        System.out.println("remove = " + heads.get(i));
                    }
                }
            }
        }
        heads.removeAll(temp);
        int result = 0;
        for (Entry head : heads) { //遍历所有链表，最长的即为最后的结果
            printList(head);
            int len = head.tailLength();
            if (len > result) {
                printList(head);
                result = len;
            }
        }
        return result;
    }

    private static void printList(Entry entry) {
        while (entry != null) {
            System.out.print(entry + " ");
            entry = entry.next;
        }
        System.out.println();
    }

    static class Entry {
        int index = 0;
        int value = 0;
        boolean isHead = false;
        Entry next = null;

        int tailLength() {
            int i = 0;
            Entry current = this;
            while (current != null) {
                i++;
                current = current.next;
            }
            return i;
        }

        @Override
        public String toString() {
            return ""+value;
        }
    }
}
```
