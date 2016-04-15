title: node 和 coffee 工程环境下的单步调试
date: 2016-04-15 10:24:07
tags: node
---

## 单步调试的基本原理

1. 运行环境执行debug模式，开启指令监听端口
2. 调试客户端对监听端口发出指令

## node 原生调试

1. 在命令行下，对需要调试的xxx.js中设置debugger

2. 使用如下命令

~~~
node debug xxx.js
~~~

3. 使用各种命令控制debug，参考[官方文档](https://nodejs.org/api/debugger.html)

> node的debug基于tcp，可以对pid和URI做debug, 请细看摘选

参考摘选
~~~
Advanced Usage#
An alternative way of enabling and accessing the debugger is to start Node.js with the --debug command-line flag or by signaling an existing Node.js process with <font style="color:green">SIGUSR1.

Once a process has been set in debug mode this way, it can be connected to using the Node.js debugger by either connecting to the pid of the running process or via URI reference to the listening debugger:

node debug -p <pid> - Connects to the process via the pid
node debug <URI> - Connects to the process via the URI such as localhost:5858
~~~

## coffee的调试原理
coffee的运行需要编译为js后再由node执行，不能直接debug coffee文件的;

对coffee的debug实际是在js中debug，然后通过sourceMap技术映射到coffee的文本中(映射到某行的提示是某些工具做的，大概是这样啦，[sourceMap技术文档](https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit?hl=en_US&pli=1&pli=1)和[intro](http://www.html5rocks.com/en/tutorials/developertools/sourcemaps/))

## 以下是各种coffee调试工具(js的基本就是少了coffee编译），欢迎补充

### webstorm下的调试, [官方参考](https://www.jetbrains.com/help/webstorm/11.0/debugging-coffeescript.html)

1. 设置File Watcher并启用, [官方参考](https://www.jetbrains.com/help/webstorm/11.0/transpiling-coffeescript-to-javascript.html#d119891e540)
2. 编辑coffee文件, 在coffee中设置webstorm的断点（编辑器左边空白鼠标左键）
3. 选择对应js文件 Run->debug 该文件

### node-inspector + chrome 调试
[node-inspector](https://github.com/node-inspector/node-inspector) 基本类似代理，暴露给chrome相关端口，代理访问node环境

0. 使用 coffee -c -m 命令同时生成sourceMap文件

1. node-inspector启动调试服务器

2. node --debug-brk app.js 开启调试
> --debug-brk制定开始时就中断，不然你的程序直接跑完了

3. chrome访问 node-inspector给出的本地地址，使用chrome内的dev工具调试，chrome根据sourceMap会自动映射到coffee文件

### devtool + chrome 调试（目前只发现能够运行js,es6能否直接debug还未测试，还不支持coffee sourceMap，但是看起来很酷）

[devtool github](https://github.com/Jam3/devtool), 在devtool中直接加载了文件和[iron-node](https://github.com/s-a/iron-node)类似（devtool列举了相似工程里就有它）



