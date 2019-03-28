---
title: 实现数组的Flat方法
date: 2018-02-28
tags: ['array', 'flat','js']
categories: ['array']
---
## flat含义及要求

> flat() 方法会递归到指定深度将所有子数组连接，并返回一个新数组。
<!--more-->
语法：

````javascript
/*
* @param {number} depth 深度
* @returns {array} array 处理完的数组
*/
var newArray = arr.flat(depth)
````
特点：

1. 将多维数组扁平化
2. 移除数组中的空项

## 实现

### 方法一
思路：

* 借助数组的reduce方法，迭代数组的每一项，将每一项合并成新数组。
* 同时记录递归次数，继续递归至次数已经达到传入的depth，操作结束。

代码实现：

````javascript
function flat(arr, depth = 1) {
  if(Number.isNaN(Number(depth))||Number(depth)<1) return arr

  let timer = 1

  function recursionFun(arr) {
    return arr.reduce((res, item) => {
      if(Array.isArray(item) && timer < depth) {
        timer++
        return res.concat(recursionFun(item))
      }else {
        return res.concat(item)
      }
    }, [])
  }

  return recursionFun(arr, depth)
}
````
### 方法二(不考虑depth参数)

思路：使用数组的toString方法

````javascript
function flat(arr) {
  return arr.toString().split(',').map(item => +item);
}
````
## 参考资料
* [TC39规范](https://tc39.github.io/proposal-flatMap/#sec-Array.prototype.flat)
* [Array.prototype.flat()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/flat)