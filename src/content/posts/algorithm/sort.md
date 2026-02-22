---
title: 十种排序算法详解
published: 2026-01-21
tags: [Learning,sort,algorithm,python,c++]
category: algorithm
draft: true
---


# 十种排序算法详解

## 冒泡排序

### 实现思路
遍历容器，如果当前元素比后面元素大就交换位置，移到下一个位置。这样一趟下来，能确保最后一个元素一定是最大的，然后再从1~n-1再进行同样操作，直到排序完成
### 实现代码

c++ 版本
```c++
void Bubble_sort(vector<int>& a){
    int size = a.size();
    for(int i=0;i<size;i++){
        for(int j=0;j<size-i-1;j++){
            if(a[j]>a[j+1]) swap(a[j],a[j+1]);
        }
    }
}
```

python 版本
```python
    def bubble_sort(arr):
        n = len(arr)
        for i in range(n):
            swapped = False
            for j in range(0, n-i-1):
                if arr[j] > arr[j+1]:
                    arr[j], arr[j+1] = arr[j+1], arr[j]
                    swapped = True
            if not swapped:
                break
        return arr

```

### 算法分析

时间复杂度分析：$O(n^2)$  
空间复杂度分析：$O(1)$  
是一种稳定的排序


## 选择排序



### 实现思路

顾名思义，从序列中选择一最小（最大）的数，把他放在最前（最后），然后对剩余未排序部分继续执行这一操作，直到序列排序完成

### 实现代码

c++ 版本
```c++
void select_sort(vector<int> & a){
    for(int i=0;i<a.size()-1;i++){
        int min = a[i];
        int min_id = i;
        for(int j=i;j<a.size();j++){
            if(a[j]<min){
                min = a[j];
                min_id = j;
            }
        }
        swap(a[i],a[min_id]);
    }
}
```

python 版本

```python
def selection_sort(arr):
        n = len(arr)
        for i in range(n):
            min_idx = i
            for j in range(i+1, n):
                if arr[j] < arr[min_idx]:
                    min_idx = j
            arr[i], arr[min_idx] = arr[min_idx], arr[i]
        return arr
```

### 算法分析
时间复杂度：$O(n^2)$  
空间复杂度： $O(1)$  
稳定性分析：是一种不稳定的排序。例：  
若给定数组 n = [25,25,1] 在第一遍遍历后，会将n[0] 与 n[2] 互换位置，此时两个25 的顺序调换

## 插入排序


### 实现思路
把整个数组分为两个部分，一部分作为已排序部分，一部分作为未排序部分，每次把未排序部分得的第一个数插入到已排序部分的合适位置。

### 实现代码

c++版本
```c++
void insert_sort(vector<int> &a){
    for(int i=1;i<a.size();i++){
        int j = i;
        while(j>0&&a[j-1] > a[j]){
            swap(a[j-1],a[j]);
            j--;
        }
    }
}
```
```python
def insertion_sort(arr):
    n = len(arr)

    for i in range(1,n):
        j = i
        while j>0 and arr[j-1] > arr[j]:
            arr[j-1],arr[j] = arr[j],arr[j-1]
            j -= 1

    return arr
```

### 算法分析
时间复杂度：$O(n^2)$
空间复杂度： $O(1)$
稳定性分析：是一种稳定的排序算法
## 希尔排序

### 实现思路
希尔排序是一种优化的插入排序。
先选择一个比较大的增量h。在数组中相隔为h的数记为1组，把原数组分为若干组后，进行插入排序。再递减h，直到h减少为1时，完成排序

该排序的关键在于选择h序列。
这里常用的h序列为 h = h*3+1; h< nums.size()
### 实现代码

```c++
void xier_sort(vector<int> & a){
    int len = a.size();
    int h = 1;
    while(h < len / 3){
        h = h*3 +1;
    }

    while(h >= 1){
       for(int i=0;i<h;i++){
            for(int j=i+h;j<len;j+=h){
                int k=j;
                while(k>=h&&a[k] < a[k-h]){
                    swap(a[k],a[k-h]);
                    k-=h;
                }
            }
       }

       h = (h-1)/3;
    }
}

```
### 算法分析

算法的时间复杂度与选择的希尔序列有关
空间复杂度为O(1)
不稳定的排序

## 归并排序



### 实现思路
递归的实现。
把数组分为左右两个部分，假设者两个部分已经排好序，然后将这个两个部分按序放入一个新的数组。最后再把新数组的值拷贝回来。对于这两个部分，递归的调用merge函数。即可

### 实现代码
```c++
void merge(int l,int r,vector<int>&a,vector<int>&b){
    if(l==r) return;

    int mid = (l+r) >> 1;
    merge(l,mid,a,b);
    merge(mid+1,r,a,b);

    int pl=l,pr=mid+1;
    int k=l;
    while(pl<=mid&&pr<=r){
        if(a[pl] < a[pr]){
            b[k] = a[pl];
            pl++;
        }else{
            b[k] = a[pr];
            pr++;
        }
        k++;
    }

    if(pr<=r){
       while(pr <= r){
            b[k] = a[pr];
            k++;pr++;
       }
    }

    if(pl<=mid){
        while(pl<=mid){
            b[k] = a[pl];
            k++;
            pl++;
        }
    }

    for(int i=l;i<=r;i++){
        a[i] = b[i];
    }
}


void merge_sort(vector<int>& a){
    b.resize(a.size());
    merge(0,a.size()-1,a,b);
}

```

### 算法分析
空间复杂度O(N)
时间复杂度O(NlogN)
是一种稳定的排序

## 快速排序


### 实现思路
分治思想，选择一个基准，把小于该基准的数放在数组左边把大于该基准的数放在数组右边，
把等于该基准的放在数组中间，最后递归求解。
### 实现代码
```c++
void quick_sort(vector<int> &a,int l,int r){
    if(l>=r) return;
    int pl=l-1,pr=r+1;
    int k=l;
    int base = a[l];
    while(k<pr){
        if(a[k] > base){
            swap(a[k],a[--pr]);
        }else if(a[k] < base){
            swap(a[k++],a[++pl]);
        }
        else if(a[k]==base) k++;
    }

    quick_sort(a,l,pl);
    quick_sort(a,pr,r);
}

```
### 算法分析

平均时间复杂度为O(NlogN)，但是最差时间复杂度为O(N**2);

## 堆排序

### 实现思路

### 实现代码

### 算法分析

## 计数排序

### 实现思路

### 实现代码

### 算法分析

## 桶排序

### 实现思路

### 实现代码

### 算法分析

## 基数排序

### 实现思路

### 实现代码

### 算法分析