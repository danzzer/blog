title: Nginx Location匹配规则
date: 2016-03-11 14:53:49
tags: nginx
---
## Location语法规则

```
location optional_modifier location_match {

    . . .

}
```

其中<font style="color:red">location_match</font>为URI匹配字符串

optional_modifier:

* (none): 无修饰则为前缀匹配
* =     : 前缀完全匹配, 即/path只匹配/path/单一目录的访问
* ~ : 正则匹配，且大小写敏感
* ~* : 正则匹配，且大小不敏感
* ^~ : 前缀匹配，且如果匹配则不再进行正则匹配

## Location匹配顺序

A. 先进行前缀匹配(非正则匹配)

	1. 对=进行前缀完全匹配，如果有匹配则使用该规则且停止匹配
	2. 如果没有=匹配且存在^~的匹配则使用该规则且停止匹配
	3. 如果(none)匹配则先缓存该匹配
B. 进行正则匹配，按照书写顺序第一个满足的将被使用且停止匹配
C. 如果A3有匹配则使用该规则
D. 无匹配则4XX

## 可能常用写法(注意规则顺序)

```
location = /B/test.html {
}

location ^~ /B/C {
}

location ~ /B/.*$ {
}

location ~ /(.*)$ {
    #此处使用正则捕获, $1可以取出匹配的uri，当然也可以用nginx的变量实现
}

```
## 来源Blog
digitalocean上的此篇[Blog](https://www.digitalocean.com/community/tutorials/understanding-nginx-server-and-location-block-selection-algorithms)不错，推荐下
	
