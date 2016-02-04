title: web加载性能
date: 2016-02-04 16:48:24
tags:
---
## DOM和页面内容基本特征

1. HTML parse 为DOM的过程（初步的Layout）不受页面内标签影响

2. ‘同步’的script的加载会阻塞所有(没有优化的情况下)后续资源的加载;css由于不会修改DOM加载都是并行的, 但是可能阻塞‘同时’访问对应属性的script

以下引用[参考blog](http://taligarsiel.com/Projects/howbrowserswork1.htm)详细描述, 注意有颜色部分：

> The order of processing scripts and style sheets

> Scripts

> The model of the web is <font color='green'>synchronous</font>. Authors expect scripts to be parsed and executed immediately when the parser reaches a script tag. <font color='green'>The parsing of the document halts until the script was executed</font>. If the script is external then the resource must be first fetched from the network - this is also done synchronously, <font color='green'>the parsing halts until the resource is fetched</font>. This was the model for many years and is also specified in HTML 4 and 5 specifications. Authors could mark the script as <font color="blue">"defer"</font> and thus it will not halt the document parsing and will execute <font color="blue">after it is parsed</font>. HTML5 adds an option to mark the script as asynchronous so it will be parsed and <font color="blue">executed by a different thread</font>.

注意：script的defer是在dom的parse完成后再执行，async是异步(不同现成)并行执行，这里应该解释的很清晰了

> Speculative parsing(部分浏览器内核的优化)

> Both Webkit and Firefox do this optimization. While executing scripts, <font color="green">another thread</font> <font color="red">parses</font> the rest of the document and finds out what other resources need to be loaded from the network and loads them. These way resources can be loaded on parallel connections and the overall speed is better. Note - <font color="blue">the speculative parser doesn't modify the DOM tree and leaves that to the main parser</font>, it <font color="red">only parses</font> references to external resources like external scripts, style sheets and images.

注意：另外的线程仅仅是‘parse document’（html->DOM）和‘加载’(network),由于涉及到DOM,javascript的执行过程还是在main parser(UI thread!)中的

> Style sheets

> Style sheets on the other hand have a different model. Conceptually it seems that since style sheets don't change the DOM tree, <font color="green">there is no reason to wait for them and stop the document parsing</font>. There is an issue, though, of scripts asking for style information during the document parsing stage. If the style is not loaded and parsed yet, the script will get wrong answers and apparently this caused lots of problems. It seems to be an edge case but is quite common. Firefox blocks all scripts when there is a style sheet that is still being loaded and parsed. Webkit blocks scripts only when they try to access for certain style properties that may be effected by unloaded style sheets.

注意：javascript对css中的属性有依赖时候可能导致的问题


## 页面加载慢的一些原因探寻

结合目前的一些已有的[tips(from yahoo)](https://developer.yahoo.com/performance/rules.html)，将按照以下层次探索加载的问题和解决策略

1. 网络通信建立
2. HTTP请求头部,内容大小,传输速率,并发和逻辑依赖
3. 用户体验和心理

### 网络通信建立

HTTP由于基于TCP/IP stack存在基本socket通信的一些问题

1. URL到IP的DNS解析需要消耗时间（20-120ms）
> Server DNS的TTL是一个很小的影响因素，主要是client的浏览器和系统中的缓存时间过小会频繁做DNS解析，引用雅虎的做法如下（可能目前大部分浏览器都已经自动优化了）：
> 
> Internet Explorer caches DNS lookups for 30 minutes by default, as specified by the DnsCacheTimeout registry setting. Firefox caches DNS lookups for 1 minute, controlled by the network.dnsCacheExpiration configuration setting. (Fasterfox changes this to 1 hour.)

2. socket的建立和销毁需要时间
> 如果每个资源的获取都要和不同IP建立socket，那么client的系统中会有socket的资源创建过程，必然会耗费一定的时间；socket销毁时也会有TIME-WAIT状态（等待足够的时间以确保远程TCP接收到连接中断请求的确认）导致资源的暂时不可用；如果资源能够策略性通过有限的socket（socket会被复用）获取可以减少此部分的消耗

3. IP报文的拆分和合并，主要影响信息利用率，对HTTP的影响可以忽略
> 以太网的MTU是1500字（由于IPv6的发展，目前网络情况是可能支持巨大包传输的，依赖ISP的实施情况，这里还是基于基础IPv4网络通识解释）,由于此限制 IP层报文过大时会被网络设备自动拆包分组传输，会影响传输效率，但是时间消耗（当前国内网络设备的影响都是毫秒级的）可以被忽视

### HTTP

1. HTTP （目前常用1.1）协议是无状态协议，那么在一个请求中为了区分客户端状态会附加cookies，如果所有请求都统一cookie那么会造成浪费；各个浏览器可能限制cookies最多4KB以内， 按照目前的网络条件可能是一个微小的改进项

2. Yahoo指出AJAX（XMLHttpRequest）请求方式GET 和 POST 在浏览器上实现不同， 大多数POST都需要俩个TCP报文，GET只有一个；在都能满足需求的基础上，选择GET是减少通信时间的方式（注意是AJAX的GET和POST), [相关内容](https://josephscott.org/archives/2009/08/xmlhttprequest-xhr-uses-multiple-packets-for-http-post/)

3. HTTP中文件和内容大小是我们容易想到的因素，对于图片和文本文件我们可以寻找对应压缩方法，这是工作中经常会处理的

> 图片压缩方法有很多：css sprite做图片合并（复用颜色）同时使用background引用图片）， 使用无损的webfont代替较大的位图，使用webq图片压缩....当然新技术可能会有兼容性问题，<font color="gree">css的base64并不是图片压缩，只是提前了图片渲染的时序</font>

> css压缩： 工具化去掉空格和换行，尽量少地写css, 多css文件合并（这个是为了尽快加载所有css）等

> js压缩：基本都是使用uglify.js做混淆（人不可读）和压缩空格和换行等

> web服务器端支持gzip，一般开启后会有不错的效果

4. HTTP的传输速率上很自然会想到服务器带宽，同时CDN的使用可以在地域上部分加速

5. HTTP在浏览器中的‘对一个host的并发请求’和‘并发请求总数’是有限制的，[参考来源](http://www.browserscope.org/?category=network&v=top), 合理安排请求的host既可以在tcp层复用socket同时也能按照该特性充分使用并发的限制

> 如果浏览器限制对一个host最多有5个并发请求，总并发请求不超过10。那么同等网络条件下（无限带宽和资源），对一个host请求10个文件将比对两个host分别请求5个文件要慢
 
> 因此使用cdn的时候可以设置多个domain（多host）来应用以上的技巧

6. 实际使用中通常会看到javascript通过添加script来加载动态的脚本引用，该引用脚本将对DOM的一些样式属性或者DOM做操作，那么这些DOM的样式在脚本执行完成前对于用户来说将是未完成的显示样式（前端MV*的框架一般可能空白一会!）

> 一般不在javascript中做dom的操作是应为相对很慢


### 用户体验

页面的加载性能和用户‘快速’体验不是同等的概念：加载只是针对资源，而用户看到的只是浏览器可视区域的内容，注意用户需要的是他们眼里的：

1. 快速：开了页面就能看到东西，虽然用户并不知道那是什么，但是他知道这个页面在运行，并且有基本信息可以阅读

> 某些时候mobile环境下网速<5KB,感觉一切都是白费的

很自然的我们可以：

1. 在html内的style内使用css做缓冲动画（比如旋转的各种图标）, 真实的内容渲染时再去掉它，那么用户第一眼感觉上是这个图片在加载
2. 对于图片这种大文件可以使用 lowsrc做图片预载（先使用模糊小图片，后用完全尺寸替换), 可以使用默认占位的图片或者背景色
3. 由于用户看到的是可视区域内的内容，那么可视区域外的图片（貌似也只能对图片）可以做懒加载（判断是否在window.innerXXX范围内）或者延迟加载（使用settimeout延迟加载资源）

> 懒加载 需要在window的一些事件中对图片遍历判断是否开始出现在可视区域（对的你得自己维护一个或者多个队列），具体做法请参考[这里](http://stackoverflow.com/questions/123999/how-to-tell-if-a-dom-element-is-visible-in-the-current-viewport/7557433#7557433)，开始出现时再通过javascript加载


## 总结

目前只想到这么多，好的想法请Pull request




