title: 使用Promise合理处理模块封装和数据的关系
date: 2016-02-16 14:23:19
tags:
---
## 数据到展现的过程

对Javascript代码做模块封装时一般都会在数据处理过程如何拆分上纠缠很久, 
主要需要考虑<font color='green'>'view的通用性'</font>和<font color='green'>'data源的多样性'</font>如何结合. 

抽象data->view的过程如下(MV*都需要手工编码这个过程):

1. 获取数据源(url)
2. 从数据源获取数据后,配适数据为view需要的格式
3. view展现数据


结合以上过程解释下:
* view的通用性: 第3过程基本编码后能够在不修改的情况下在其他编码中使用,提高代码复用程度
* data源的多样性: 数据源可能有不同, 相同数据可能需要为不同view配适

## 曾经的过程化编码

独立开发或者维护的时候经常会出现一批如下不能复用的代码, 主要原因大概是写的时候有个念头-"这个代码过程只会在这里用",当然结局都是填坑比挖坑还累

~~~
   //伪代码,不是jquery
   $.ajax({
    //1. 获取数据源 
    url:"url", 
    queryCondition: {}
    
    success: function(data) {
        
        //这里假设data已经转化为object
        //2. parse数据
        var displayData = {
            name: data.othername
        }
        
        //3. view展现数据
        document.getElementById("view").innerHtml(...)
    }})
~~~

## 怎么复用?

以上过程化代码存在如下问题

1. 对于异步请求处理, 使用了callback写法, callback hell 不可避免, 需要使用Promise解决.[问题参考](http://callbackhell.com/), [中文的](https://www.phodal.com/blog/javascript-promise/)
2. 对于url,parse,view 的需求变化需要hard code, 必须做封装

为了封装 "获取数据源", 我曾经这么写

~~~
   var dataSource = [{
        url: "abc",
        queryCondition,
        ....
    }]
    .....
    function dataFetchFactory (source) {
        return $.promisedAjax({
            url: source.url,
            queryCondition: source.queryCondition
        })
    }
    .....
    var handleData = dataFetchFactory(dataSource[0])
    handle.then(function(data) {
        var displayData = {
            name: data.othername
        }
        
        document.getElementById("view").innerHtml(...)
    })
~~~

虽然'获取数据源' 被封装成了类似工厂的用法,但是工厂函数内依然逻辑结构上耦合了数据源的数据结构, 如果dataSource修改了那么factory的函数也必须修改;同时依然不能处理多个parse的需求

近期改写过程如下,此方案还是需要深入思考和修改

~~~
   //1.promise直接视为数据源,不再需要factory, 不同数据源只需要创建不同的promise
   //如果需要错误处理,则在$.promisedAjax上再包装一层promise
   var dataSource = [
        $.promisedAjax({
            url: source.url,
            queryCondition: source.queryCondition
        })
   ]
   .....
   //2. 为了处理parse的不同需求问题,需要不同的parser
  
   var parser1 = function(data) {
        return { name: data.othername }
   }
   var parser2 = function(data) {
        //满足相同data的不同parse需求
        return { 名字: data.othername}
   }
   var parse3 = function(data) {
        //不同data的parse
        return { name: data.cutename }
   }
   
   // parser 也可以考虑promise化
   var parsePromise = function(promise) {
        var deferred = Q()
        promise.then(function(d) {
            var data = { name: data.cutename }
            deferred.resolve(data)
        })
       
        return deffered.promise
   }
   .....
   //3. model到view过程已经有很多的框架或者既有写法了,不再赘述
   function display(data) {
        document.getElementById("view").innerHtml(...)
   }
   .....
   // 在代码中复用以上的代码
   
   dataSource[0].then(function(d) {
        display(parser1(d))
   })
   
   // 如果parser 也promise化, 那么看起来如下, 有点自解释的感觉
   dataSource[0]
    .then(parsePromise)
    .then(display)
~~~

三者之间除了2和3在最终数据格式上有逻辑规约,其他没有耦合;即使逻辑规约改变了, parser和display的替换都比hardcode 方便

## 总结

注意promise 除了很好避免了callback hell, 同时更可贵的是在抽象层面上将"过程"很好的封装, 假设所有中间过程的处理结果都是promise化,那么代码能够很清晰反应逻辑顺序

~~~
    //语言有时候很无力,看下面你大概就知道了
    var begin = A()
    begin.then(B).then(C).....
~~~