---
title: 常见排序算法的JavaScript实现
date: 2018-06-09
tags: ['排序', 'js']
categories: ['算法']
---
## 冒泡排序

冒泡排序（英语：Bubble Sort）是一种简单的排序算法。它重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误就把他们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。这个算法的名字由来是因为越小的元素会经由交换慢慢“浮”到数列的顶端。
<!--more-->
### 算法步骤

1. 比较相邻的元素。如果第一个比第二个大，就交换他们两个。
2. 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。这步做完后，最后的元素会是最大的数。
3. 针对所有的元素重复以上的步骤，除了最后一个。
4. 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。

### 动图演示

![bubbleSotrt.gif](https://i.loli.net/2019/02/26/5c75078e5682c.gif)

### 实现
````javascript
function bubble(array) {
  for(let i = array.length - 1; i > 0; i--) {
    for(let j = 0; j < i; j++) {
      if(array[j] > array[j+1]) {
        swap(array[j], array[j+1])
      }
    }
  }
}//(n-1)*(n-2)
````
## 插入排序

插入排序（英语：Insertion Sort）是一种简单直观的排序算法。它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。插入排序在实现上，通常采用in-place排序（即只需用到 {\displaystyle O(1)} {\displaystyle O(1)}的额外空间的排序），因而在从后向前扫描过程中，需要反复把已排序元素逐步向后挪位，为最新元素提供插入空间。

### 算法步骤

1. 从第一个元素开始，该元素可以认为已经被排序
2. 取出下一个元素，在已经排序的元素序列中从后向前扫描
3. 如果该元素（已排序）大于新元素，将该元素移到下一位置
4. 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置
5. 将新元素插入到该位置后
6. 重复步骤2~5

### 动图演示
![insertSort.gif](https://i.loli.net/2019/02/26/5c75078f47282.gif)

### 实现
````javascript
function insertion(array) {
  for(let i = 1; i < array.length; i++) {
    for(let j = i - 1; j >= 0; j--) {
      if(array[j] > array[j+1]) {
        swap(array[j], array[j+1])
      }
    }
  }
}
````

## 选择排序

选择排序（Selection sort）是一种简单直观的排序算法。它的工作原理如下。首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。

### 算法步骤

1. 对比数组中前一个元素跟后一个元素的大小，如果后面的元素比前面的元素小则用一个变量k来记住他的位置。
2. 接着第二次比较，前面“后一个元素”现变成了“前一个元素”，继续跟他的“后一个元素”进行比较如果后面的元素比他要小则用变量k记住它在数组中的位置(下标)，等到循环结束的时候，我们应该找到了最小的那个数的下标了。
4. 然后进行判断，如果这个元素的下标不是第一个元素的下标，就让第一个元素跟他交换一下值，这样就找到整个数组中最小的数了。
5. 然后找到数组中第二小的数，让他跟数组中第二个元素交换一下值，以此类推。
### 动图演示

![selectSort.gif](https://i.loli.net/2019/02/26/5c75078f6ad90.gif)

### 实现
````javascript
function selection(array) {
  for(let i = 0; i < array.length - 1; i++) {
    let minIndex = i
    for(let j = i + 1; j < array.length; j++) {
      minIndex = array[j] < array[minIndex] ? j : minIndex
    }
    swap(array[minIndex], array[i])
  }
}
````

## 并归排序

归并排序（英语：Merge sort，或mergesort），是创建在归并操作上的一种有效的排序算法，效率为 O(nlog n)（大O符号）。1945年由约翰·冯·诺伊曼首次提出。该算法是采用分治法（Divide and Conquer）的一个非常典型的应用，且各层分治递归可以同时进行。

### 算法步骤

* 递归法（Top-down）
1. 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列。
2. 设定两个指针，最初位置分别为两个已经排序序列的起始位置。
3. 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置。
4. 重复步骤3直到某一指针到达序列尾。
5. 将另一序列剩下的所有元素直接复制到合并序列尾。

* 迭代法（Bottom-up）
原理如下（假设序列共有 n 个元素）：

1. 将序列每相邻两个数字进行归并操作，形成 ceil(n/2) 个序列，排序后每个序列包含两/一个元素。
2. 若此时序列数不是1个则将上述序列再次归并，形成 ceil(n/4) 个序列，每个序列包含四/三个元素。
3. 重复步骤2，直到所有元素排序完毕，即序列数为1。

### 动图演示

![mergeSort.gif](https://i.loli.net/2019/02/26/5c75078f45650.gif)

### 实现

* 递归法
````javascript
function merge(left, right) {
  let result = []
  while(left.length > 0 && right.length > 0) {
    if(left[0] < right[0]) {
      result.push(left.shift())
    }else {
      result.push(right.shift())
    }
  }

  return result.concat(left, right)
}

function mergeSort(array) {
  if(array.length < 2) return array

  let middle = (array.length / 2)
  let left = array.slice(0, middle)
  let right = array.slice(middle)

  return merge(mergeSort(left), mergeSort(right))
}
````
* 迭代法
````javascript
function mergeSort(arr){
  if(arr.length<2){
    return;
  }
  //设置子序列的大小
  var step=1; 
  var left,right;

  while(step<arr.length){
    left=0;
    right=step;
    while(right+step<=arr.length){
      mergeArrays(arr,left,left+step,right,right+step);
      left=right+step;
      right=left+step;
    }
    if(right<arr.length){
      mergeArrays(arr,left,left+step,right,arr.length);
    }
    step*=2;
  }

}
//对左右序列进行排序
function mergeArrays(arr,startLeft,stopLeft,startRight,stopRight){
  // 建立一个左、右数组
  var rightArr=new Array(stopRight-startRight+1);
  var leftArr=new Array(stopLeft-startLeft+1);

  // 给右数组赋值
  k=startRight;
  for(var i=0;i<(rightArr.length-1);++i){
    rightArr[i]=arr[k];
    ++k;
  }
   // 给左数组赋值
  k=startLeft;
  for(var i=0;i<(leftArr.length-1);++i){
    leftArr[i]=arr[k];
    ++k;
  }
  //设置哨兵值，当左子列或右子列读取到最后一位时，即Infinity，可以让另一个剩下的列中的值直接插入到数组中
  rightArr[rightArr.length-1]=Infinity;
  leftArr[leftArr.length-1]=Infinity;

  var m=0;
  var n=0;
  // 比较左子列和右子列第一个值的大小，小的先填入数组，接着再进行比较
  for(var k=startLeft;k<stopRight;++k){
    if(leftArr[m]<=rightArr[n]){
      arr[k]=leftArr[m];
      m++; 
    }
    else{
      arr[k]=rightArr[n];
      n++;
    }
  }
  
}
````
## 快速排序

快速排序（英语：Quicksort），又称划分交换排序（partition-exchange sort），简称快排，一种排序算法，最早由东尼·霍尔提出。在平均状况下，排序 n 个项目要 O(nlog n)（大O符号）次比较。在最坏状况下则需要 O(n^{2}) 次比较，但这种状况并不常见。事实上，快速排序 (n\log n) 通常明显比其他算法更快，因为它的内部循环（inner loop）可以在大部分的架构上很有效率地达成。

### 算法步骤

1. 从数列中挑出一个元素，称为“基准”（pivot），
2. 重新排序数列，所有比基准值小的元素摆放在基准前面，所有比基准值大的元素摆在基准后面（相同的数可以到任何一边）。在这个分割结束之后，该基准就处于数列的中间位置。这个称为分割（partition）操作。
3. 递归地（recursively）把小于基准值元素的子数列和大于基准值元素的子数列排序。
4. 递归到最底部时，数列的大小是零或一，也就是已经排序好了。这个算法一定会结束，因为在每次的迭代（iteration）中，它至少会把一个元素摆到它最后的位置去。
### 动图演示

![quickSort.gif](https://i.loli.net/2019/02/26/5c75078e54e94.gif)

### 实现
````javascript
function quickSort(array) {
  let l = array.length
  if(l < 2) return array

  let basic = array[0]
  let left = []
  let right = []

  for(let i = 1; i < l; i++) {
    if(array[i] < basic) {
      left.push(array[i])
    }else {
      right.push(array[i])
    }
  }

  return quickSort(left).concat(basic, quickSort.concat(right))
}
````

## 复杂度

|算法|时间复杂度最好情况|时间复杂度平均情况|时间复杂度最坏情况|空间复杂度|稳定性|
|---|---|---|---|---|---|
|冒泡排序|O(n)|O(n^2)|O(n^2)|总共 O(n)，需要辅助空间 O(1)|是|
|插入排序|O(n)|O(n^2)|O(n^2)|总共 O(n)，需要辅助空间 O(1)|是|
|选择排序|O(n^2)|O(n^2)|O(n^2)|总共 O(n)，需要辅助空间 O(1)|否|
|并归排序|O(nlogn)|O(nlogn)|O(nlogn)|总共 O(n)，需要辅助空间 O(1)|是|
|快速排序|O(nlogn)|O(nlogn)|O(n^2)|O(n) 或者 O(nlogn)|是|

## 参考资料

* [排序算法](https://zh.wikipedia.org/wiki/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95)