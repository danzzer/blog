title: react-router 快速印象
date: 2016-01-16 16:23:50
tags: react react-router javascript
---
# react-router 

[react-router 的git](https://github.com/rackt/react-router)

## react-router 基本使用
使用方式，直接看最底部Router的配置：

~~~
import React from 'react'
import { render } from 'react-dom'
import { Router, Route, Link, browserHistory } from 'react-router'

const App = React.createClass({/*...*/})
const About = React.createClass({/*...*/})
// etc.

const Users = React.createClass({
  render() {
    return (
      <div>
        <h1>Users</h1>
        <div className="master">
          <ul>
            {/* use Link to route around the app */}
            {this.state.users.map(user => (
              <li key={user.id}><Link to={`/user/${user.id}`}>{user.name}</Link></li>
            ))}
          </ul>
        </div>
        <div className="detail">
          {this.props.children}
        </div>
      </div>
    )
  }
})

const User = React.createClass({
  componentDidMount() {
    this.setState({
      // route components are rendered with useful information, like URL params
      user: findUserById(this.props.params.userId)
    })
  },

  render() {
    return (
      <div>
        <h2>{this.state.user.name}</h2>
        {/* etc. */}
      </div>
    )
  }
})

// Declarative route configuration (could also load this config lazily
// instead, all you really need is a single root route, you don't need to
// colocate the entire config).
render((
  <Router history={browserHistory}>
    <Route path="/" component={App}>
      <Route path="about" component={About}/>
      <Route path="users" component={Users}>
        <Route path="/user/:userId" component={User}/>
      </Route>
      <Route path="*" component={NoMatch}/>
    </Route>
  </Router>
), document.body)
~~~

以上Users中通过<LINK> 访问不同User, 注意Users中显示写了 this.props.children 用于User的更新。

关键词： Router， Route， LINK

NOTE: 

1. react-router只能是“层级嵌套”的view层级， 这意味着整体替换（比如APP整体结构都变了） 的层级变化是不能实现的 

2. 注意到component是没有带自定义参数的（目前没有搜索到），只能通过path携带， 这么一来。。。。好像不灵活了

3. LINK不是单纯a标签，跳转是有限制的

4. router 实际需要操作history ，注意 Router中的history 属性绑定


## 配合wepack实现异步加载module

router的example中的huge-apps 是一个异步加载module的示例， 主要是通过cmd的加载语法require.ensure 让webpack自动分开打包文件。

此项目文档很呵呵， 异步加载实际需要使用Router中的route属性，但是文档中找不到， 以下是过程的一个简写示例， 只看 rootRoute 即可

~~~
import React from 'react'
import { render } from 'react-dom'
import { createHistory, useBasename } from 'history'
import { Router } from 'react-router'
import stubbedCourses from './stubs/COURSES'

const history = useBasename(createHistory)({
  basename: '/huge-apps'
})

const rootRoute = {
  component: 'div',
  childRoutes: [ {
    path: '/',
    component: require('./components/App'),
    childRoutes: [
      {
        path: 'calendar',
        getComponent(location, cb) {
          require.ensure([], (require) => {
            cb(null, require('./routes/Calendar/components/Calendar'))
          })
        }
      },
      //后面这些实际都是对模块定义了path 和 getComponent方法（使用CMD的require.ensure加载， 加载时会映射到webpack对应的增量包）
      require('./routes/Course'),
      require('./routes/Grades'),
      require('./routes/Messages'),
      require('./routes/Profile')
    ]
  } ]
}

render(
  <Router history={history} routes={rootRoute} />,
  document.getElementById('example')
)
~~~

## 值得借鉴的地方

1. webpack中实现了写入到内存的的方式（webpackDevMiddleware中间件定向到内存， webpack中的output需要定义好规则），同样使用watch检测修改自动映射， 这可能比在“开发”时候 写code在src， 启动服务器在build下更好一点； build和src存在的一个问题是，容易在修改时候访问到不同文件夹的同名文件，频繁的话是比较恶心的，这个根据每个人的情况而定； 对于发布来说，放在内存比文件响应是要快一些的，同时webpack的过程放在了server.js中，发布也只需要执行一个命令（build 可能需要先build命令，然后执行启动命令）

2. webpack使用了code splitting 方式（之前的CMD require写法）， 同时通过wepack 配置插件webpack.optimize.CommonsChunkPlugin('shared.js')，自动提取了页面公用部分

## 高级使用部分请自行参看github中文档

## redux认为比较重要的想法
[Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.l3aljox8v)
