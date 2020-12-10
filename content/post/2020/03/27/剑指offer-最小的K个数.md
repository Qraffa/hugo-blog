---
title: 剑指offer-最小的K个数
date: 2020-03-27 14:49:07
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[最小的K个数](https://www.nowcoder.com/practice/6a296eb82cf844ca8539b57c23e6e9bf?tpId=13&tqId=11182&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。

<!--more-->
### 思路:

	- 思路一:使用快排的思想,选择一个基准数,做快排,当基准数的位置为k时,前k个就是我们要找的
	- 思路二:维护一个大小为k的最大堆,当有新数进来,若该数字大于堆顶,则直接跳过该数字,否则,将该数字与堆顶的数字交换,然后调整堆为最大堆,这样最后堆中的k个数就是最小的K个数字
	- 最大的k个数同理,找最大维护最小堆即可

### code

```java
public void adjustMaxHeap(int[] arr,int i,int len) {
    int tmp=arr[i];
    for (int k=i*2+1;k<len;k=k*2+1) {
        if(k+1<len && arr[k]<arr[k+1]) {
            k++;
        }
        if(tmp<arr[k]) {
            arr[i]=arr[k];
            i=k;
        } else {
            break;
        }
    }
    arr[i]=tmp;
}

public void buildMaxHeap(int[] arr) {
    int len=arr.length;
    for (int i = len/2-1; i>=0 ; --i) {
        adjustMaxHeap(arr,i,len);
    }
}

public void sortAscHeap(int[] arr) {
    buildMaxHeap(arr);
    int len=arr.length;
    for (int i = len-1; i>0 ; --i) {
        int tmp=arr[i];
        arr[i]=arr[0];
        arr[0]=tmp;
        adjustMaxHeap(arr,0,i);
    }
}

public ArrayList<Integer> GetLeastNumbers_Solution(int [] input, int k) {
    ArrayList<Integer> list=new ArrayList<>();
    if(k==0 || input==null || input.length==0 || input.length<k) {
        return list;
    }
    int[] heap=Arrays.copyOfRange(input,0,k);
    buildMaxHeap(heap);
    int len=input.length;
    for (int i = k; i < len; i++) {
        if(input[i]>heap[0]) {
            continue;
        } else {
            heap[0]=input[i];
            adjustMaxHeap(heap,0,k);
        }
    }
    sortAscHeap(heap);
    System.out.println(Arrays.toString(heap));
    for (int i = 0; i < k; i++) {
        list.add(heap[i]);
    }
    return list;
}
```

