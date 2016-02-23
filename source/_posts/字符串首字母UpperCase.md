title: 字符串首字母UpperCase
date: 2016-02-23 21:00:28
tags: javascript 杂技
---
这个题目估计大家都知道要用split去做分割然后替换首字母

但是比如 var strName ＝ "a b c"可以检点分割后处理，but 如果是 strName ＝ ”a   b，?c"呢？

so除了考察下split和基本字符串操作（你当然会比较+ 和 join的区别咯，不知道的可以趁机google下）,貌似并不实用. 其实正则写法挺简单的：

```
   var name = 'aaa ?bbb ,ccc';
   var uw=name.replace(/\b\w/g, function(word){
  	return word.toUpperCase()
   });
   console.log(uw)
```

> \b是单词的开头或者结尾匹配， 而且toUpperCase只对字母有效

嗯，感觉吊吊的
