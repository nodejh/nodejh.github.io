+++
title = "JS 判断字符串是否是拼音音节"
description = "Get Chinese Phonetic Syllable With JS"
date = "2016-04-13T10:35:40+08:00"
tags = ["JavaScript"]

+++



## 最终函数

最近在抓取 Rice 大学的博士生姓名，并从中获取到中国人的姓名。由于博士有中国人和外国人，而中国人的姓名是由拼音组成，所以最终需求是这样的，判断一个字符串是否是由拼音音节组成。于是写了下面这个函数：

<!--more-->


```
/**
 * 判断输入的一个字符串是不是拼音
 * @param string 需要测试的字符串
 * @returns {*}
 */
function is_pinyin(string) {

  var list = ['a', 'ai', 'an', 'ang', 'ao', 'ba', 'bai', 'ban', 'bang', 'bao', 'bei', 'ben',
    'beng', 'bi', 'bian', 'biao', 'bie', 'bin', 'bing', 'bo', 'bu', 'ca', 'cai', 'can', 'cang',
    'cao', 'ce', 'cen', 'ceng', 'cha', 'chai', 'chan', 'chang', 'chao', 'che', 'chen', 'cheng', 'chi',
    'chong', 'chou', 'chu', 'chua', 'chuai', 'chuan', 'chuang', 'chui', 'chun', 'chuo', 'ci', 'cong',
    'cou', 'cu', 'cuan', 'cui', 'cun', 'cuo', 'da', 'dai', 'dan', 'dang', 'dao', 'de', 'dei', 'den',
    'deng', 'di', 'dia', 'dian', 'diao', 'die', 'ding', 'diu', 'dong', 'dou', 'du', 'duan', 'dui', 'dun',
    'duo', 'e', 'en', 'eng', 'er', 'fa', 'fan', 'fang', 'fei', 'fen', 'feng', 'fiao', 'fo', 'fou', 'fu',
    'ga', 'gai', 'gan', 'gang', 'gao', 'ge', 'gei', 'gen', 'geng', 'gong', 'gou', 'gu', 'gua', 'guai', 'guan',
    'guang', 'gui', 'gun', 'guo', 'ha', 'hai', 'han', 'hang', 'hao', 'he', 'hei', 'hen', 'heng', 'hong', 'hou',
    'hu', 'hua', 'huai', 'huan', 'huang', 'hui', 'hun', 'huo', 'ji', 'jia', 'jian', 'jiang', 'jiao', 'jie',
    'jin', 'jing', 'jiong', 'jiu', 'ju', 'juan', 'jue', 'ka', 'kai', 'kan', 'kang', 'kao', 'ke', 'ken',
    'keng', 'kong', 'kou', 'ku', 'kua', 'kuai', 'kuan', 'kuang', 'kui', 'kun', 'kuo', 'la', 'lai', 'lan',
    'lang', 'lao', 'le', 'lei', 'leng', 'li', 'lia', 'lian', 'liang', 'liao', 'lie', 'lin', 'ling', 'liu',
    'lo', 'long', 'lou', 'lu', 'luan', 'lun', 'luo', 'lv', 'lve', 'ma', 'mai', 'man', 'mang', 'mao', 'me',
    'mei', 'men', 'meng', 'mi', 'mian', 'miao', 'mie', 'min', 'ming', 'miu', 'mo', 'mou', 'mu', 'na', 'nai',
    'nan', 'nang', 'nao', 'ne', 'nei', 'nen', 'neng', 'ni', 'nian', 'niang', 'niao', 'nie', 'nin', 'ning',
    'niu', 'nong', 'nou', 'nu', 'nuan', 'nun', 'nuo', 'nv', 'nve', 'o', 'ou', 'pa', 'pai', 'pan', 'pang', 'pao',
    'pei', 'pen', 'peng', 'pi', 'pian', 'piao', 'pie', 'pin', 'ping', 'po', 'pou', 'pu', 'qi', 'qia', 'qian',
    'qiang', 'qiao', 'qie', 'qin', 'qing', 'qiong', 'qiu', 'qu', 'quan', 'que', 'qun', 'ran', 'rang', 'rao',
    're', 'ren', 'reng', 'ri', 'rong', 'rou', 'ru', 'rua', 'ruan', 'rui', 'run', 'ruo', 'sa', 'sai', 'san',
    'sang', 'sao', 'se', 'sen', 'seng', 'sha', 'shai', 'shan', 'shang', 'shao', 'she', 'shei', 'shen', 'sheng',
    'shi', 'shou', 'shu', 'shua', 'shuai', 'shuan', 'shuang', 'shui', 'shun', 'shuo', 'si', 'song', 'sou',
    'su', 'suan', 'sui', 'sun', 'suo', 'ta', 'tai', 'tan', 'tang', 'tao', 'te', 'tei', 'teng', 'ti', 'tian',
    'tiao', 'tie', 'ting', 'tong', 'tou', 'tu', 'tuan', 'tui', 'tun', 'tuo', 'wa', 'wai', 'wan', 'wang',
    'wei', 'wen', 'weng', 'wo', 'wu', 'xi', 'xia', 'xian', 'xiang', 'xiao', 'xie', 'xin', 'xing', 'xiong',
    'xiu', 'xu', 'xuan', 'xue', 'xun', 'ya', 'yan', 'yang', 'yao', 'ye', 'yi', 'yin', 'ying', 'yo', 'yong',
    'you', 'yu', 'yuan', 'yue', 'yun', 'za', 'zai', 'zan', 'zang', 'zao', 'ze', 'zei', 'zen', 'zeng', 'zha',
    'zhai', 'zhan', 'zhang', 'zhao', 'zhe', 'zhei', 'zhen', 'zheng', 'zhi', 'zhong', 'zhou', 'zhu', 'zhua',
    'zhuai', 'zhuan', 'zhuang', 'zhui', 'zhun', 'zhuo', 'zi', 'zong', 'zou', 'zu', 'zuan', 'zui', 'zun', 'zuo'];
  var lowerString = string.toLowerCase();
  var length = lowerString.length;
  var index = -1;

  for (var i=0; i<length; i++) {
    var name = lowerString.substring(0, i+1);
    index = list.lastIndexOf(name) > index ? list.lastIndexOf(name) : index;
  }

  // 判断当前 lowerString 是不是拼音(lowerString 在 list 中就是;不在就不是)
  if (index >= 0) {
    var item = list[index];
     lowerString = lowerString.substring(item.length);
    if (lowerString.length == 0) {
      return string;
    } else {
       return arguments.callee(lowerString);
    }
  } else {
    return false;
  }
}
```

## 思路

#### 1. 找到所有拼音音节表

首先是找到所有的拼音列表，将其存入数组 `ist`。

汉语的拼音音节共 399 个，可以在百度百科找到：[汉语拼音音节](http://baike.baidu.com/view/1253220.htm)

#### 2. 递归判断字符串

接下来要循环判断传入的字符串是否是拼音音节。也就是说，我们要判断所传入的字符串是否在 `lsit` 数组中，聪明的可能想到了，可以用 `indexOf()` 来判断。

那么问题来了，我们所要判断的字符串是姓名，形如 `jianghang`，是由两个甚至更多的拼音音节组成的。所以不能直接 `list.indexOf('jianghang')`，而是要不断循环截取 `jianghang` 这个字符串，判读所截取的字符串是否在 `list` 数组中。如第一次循环截取的是 `j`，然后判断 `j` 是否在 `lsit` 里面，如果不在，则继续截取 `ji` 再判断；如果是，则又从字符串的第三位开始继续判断，直到判断完整个字符串。其实也就是递归。

然后可能你又发现了，这么做还是有问题。因为 `ji` 是拼音音节，`jia`、`jian`、`jiang` 也都是拼音音节。所以这就涉及到一个最优的问题，我们要找到 `jiang` 而不是 `ji` 或其它。

根据 `lsit` 的规则，排在后面的音节总是最优的音节。所以我们使用 `lastIndexOf()` 对数组从后往前筛选。

#### 3. 还是递归

这里需要注意的是 `return arguments.callee(lowerString);` 这一行。`arguments` 是一个类数组对象，它包含着传入函数的所有参数。这个对象有一个 `callee` 属性，该属性是一个指针，指向拥有 `arguments` 对象的函数。

这里我们使用 `return  arguments.callee(lowerString)` 而不是用 `return is_pinyin(lowerString)` 的好处就是，当我们改变函数名的时候，我们仍然可以正确使用该函数。


---
Github Issue: [https://github.com/nodejh/nodejh.github.io/issues/8](https://github.com/nodejh/nodejh.github.io/issues/8)
