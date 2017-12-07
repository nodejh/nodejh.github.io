+++
title = "什么是动态规划"
description = "What Is Dynamic Programming"
date = 2017-08-08T22:24:05+08:00
tags = [""]
categories = [""]
draft = true
+++


## 题目：

有一座高度是10级台阶的楼梯，从下往上走，每跨一步只能向上1级或者2级台阶。要求用程序来求出一共有多少种走法。

比如，每次走1级台阶，一共走10步，这是其中一种走法。我们可以简写成 1,1,1,1,1,1,1,1,1,1。

再比如，每次走2级台阶，一共走5步，这是另一种走法。我们可以简写成 2,2,2,2,2。

当然，除此之外，还有很多很多种走法。


## 备忘录算法


```cpp
#include <iostream>
#include <string>
#include <map>

using namespace std;


int f(int n, map<int, int> hash)
{
    int res;
    if (n == 1) return 1;
    if (n == 2) return 2;
    map<int, int>::iterator iter;
    iter = hash.find(n);
    if (iter == hash.end()) {
        res = f(n-1, hash) + f(n-2, hash);
        hash.insert(pair<int, int>(n, res));
        return res;
    } else {
        return iter->second;
    }
}


int main()
{
    int n, res;
    cin>>n;
    map<int, int> hash;
    res = f(n, hash);
    cout<<res<<endl;
    return 0;
}
```

## 动态规划

```cpp
#include <iostream>
#include <string>
#include <map>

using namespace std;


int f(int n)
{
    int temp=0, a=1, b=2, i;
    if (n == 1) return 1;
    if (n == 2) return 2;

    for (i=3; i<=n; i++) {
        temp = a + b;
        a = b;
        b = temp;
    }

    return temp;
}


int main()
{
    int n, res;
    cin>>n;
    res = f(n);
    cout<<res<<endl;
    return 0;
}
```


---

- [什么是动态规划](https://mp.weixin.qq.com/s?__biz=MjM5OTA1MDUyMA==&mid=2655438647&idx=1&sn=4634f712fa4d0236aba60b8e8b7cc2cb&key=02a45e2d696653c093e70edecda83bfa3aa1e25dae845087e48128d76f3cbc2df678e0f25a2098308a7454ba3f92872bd4f68d03572020054e50925e07936f81c868153b691d9ded41ab4c043c5e01e6&ascene=0&uin=MTczMjI5NTQwMA%3D%3D&devicetype=iMac+MacBookAir7%2C2+OSX+OSX+10.12.6+build(16G29)&version=12020810&nettype=WIFI&fontScale=100&pass_ticket=XoeiXCvHyuayRKBA%2BE6l6mm7qScUb%2BPqhck4NAw0rNq67TkY%2Bg4EtgUXm%2FDZ4g9c)
