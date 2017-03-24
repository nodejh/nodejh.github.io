+++
date = "2016-10-24T19:20:03+08:00"
description = "找出一个数组中的最大值"
title = "Get the Max Number in Array"
tags = ["JavaScript", "Algorithm"]

+++


本文介绍 JavaScript 的几种从数组中找出最大值的方法。

#### 使用递归函数

```
var arr = [9,8,55,66,49,68,109,55,33,6,2,1];
var max = arr[0];
function findMax( i ){
  if( i == arr.length ) return max;
  if( max < arr[i] ) max = arr[i];
  findMax(i+1);
}

findMax(1);
console.log(max);
```


#### 使用 for 循环遍历

```
var arr = [9,8,55,66,49,68,109,55,33,6,2,1];  
var max = arr[0];
for(var i = 1; i < arr.length; i++){
  if( max < arr[i] ){
    max = arr[i];
  }
}

console.log(max);
```

#### 使用apply将数组传入max方法中直接返回

```
Math.max.apply(null,[9,8,55,66,49,68,109,55,33,6,2,1])
```



除此之外，还有很多数组排序方式，都可以在排序后，根据新数组索引值获取 最大/最小 值。

```
var a=[1,2,3,5];
console.log(Math.max.apply(null, a));//最大值
console.log(Math.min.apply(null, a));//最小值
```

多维数组可以这么修改：

```
var a=[1,2,3,[5,6],[1,4,8]];
var ta=a.join(",").split(",");//转化为一维数组
console.log(Math.max.apply(null,ta));//最大值
console.log(Math.min.apply(null,ta));//最小值
```
---
Github Issue: [https://github.com/nodejh/nodejh.github.io/issues/9](https://github.com/nodejh/nodejh.github.io/issues/9)
