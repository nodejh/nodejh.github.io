+++
date = "2016-10-29T00:43:22+08:00"
description = "Symmetric Difference"
title = "求给定数组的对等差分(symmetric difference) (△ or ⊕)数组"
tags = ["JavaScript", "Algorithm"]
+++


**题目**

创建一个函数，接受两个或多个数组，返回所给数组的 对等差分(symmetric difference) (△ or ⊕)数组.

<!--more-->


给出两个集合 (如集合 A = {1, 2, 3} 和集合 B = {2, 3, 4}), 而数学术语 "对等差分" 的集合就是指由所有只在两个集合其中之一的元素组成的集合(A △ B = C = {1, 4}). 对于传入的额外集合 (如 D = {2, 3}), 你应该安装前面原则求前两个集合的结果与新集合的对等差分集合 (C △ D = {1, 4} △ {2, 3} = {1, 2, 3, 4}).


**解题思路**

+ 使用 [Array.prototype.reduce](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) 对数组进行遍历。传入的两个参数分别是 prev (对等差分后数组)，curr（当前数组）
+ 将 sym() 的第一个参数作为初始对等差分数组，即第一个 prev
+ 对 prev 数组和 curr 数组去重，防止重复元素影响对等差分的结果
+ 遍历 curr 数组，以此判断 curr 数组中每个元素是否在 prev 中出现。如果出现，则从 prev 中删除该元素；如果没有出现，则将其加入到 prev 中


代码如下：

```
// 对等差分
function sym(args) {
  var arr = Array.prototype.slice.call(arguments);
  return arr.reduce(function(prev, curr) {
      prev = unique(prev);
      curr = unique(curr);
      for (var i=0; i<curr.length; i++) {
          if (prev.indexOf(curr[i]) === -1) {
             prev.push(curr[i]);
          } else {
              prev.splice(prev.indexOf(curr[i]), 1);
          }
          console.log('new: ', prev);
      }
      return prev.sort();
  });
}


// 数组去重
function unique(arr) {
    var res = [];
    for (var i=0; i<arr.length; i++) {
        if (res.indexOf(arr[i]) === -1) {
            res.push(arr[i]);
        }
    }
    return res;
}


// sym([1, 2, 3], [5, 2, 1, 4]);
sym([1, 1, 2, 5], [2, 2, 3, 5], [3, 4, 5, 5])

```

运行效果：[Symmetric Difference](https://www.freecodecamp.cn/challenges/symmetric-difference)

---
Github Issue: [https://github.com/nodejh/nodejh.github.io/issues/1](https://github.com/nodejh/nodejh.github.io/issues/1)
