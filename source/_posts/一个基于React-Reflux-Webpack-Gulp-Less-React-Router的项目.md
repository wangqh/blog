title: 一个基于React+Reflux+Webpack+Gulp+Less+React-Router的项目
date: 2016-06-02 17:30:31
categories: 术业专攻
tags:
- React
- Reflux
- React-Router
---

此项目基于 [gulp](https://github.com/gulpjs/gulp), [less](https://github.com/less/) 和 [webpack](https://github.com/webpack/webpack)构建. View 层用 [React](https://github.com/facebook/react) 实现, 内部数据流使用  [Reflux](https://github.com/spoike/refluxjs)架构； 路由管理使用 [React-Router](https://github.com/rackt/react-router).
<!-- more -->

## Reflux 架构
一个单向数据流架构，其中包含三个重要组成部分：Actions, Stores, Components (View)
详细参考第三方[中文版API](https://segmentfault.com/a/1190000002793786)
{% asset_img reflux.png [Reflux架构] %}

## 项目文件目录

```
.
├── dist    // build目录
│  ├── css
│  ├── fonts
│  ├── js
│  └── index.html
└── src     // 源文件目录
    ├── images    // 图片目录
    ├── less    // less 文件目录
    ├── scripts     // js 源码
    │   ├── app     // 与 routes 对应的 smart components （智能组件）
    │   │   ├── activity    // 活动页面的 actions stores view
    │   │   │   ├── api     // ajax 请求
    │   │   │   ├── components    // 活动相关组件 view
    │   │   │   ├── stores    // 活动页面的数据模型层
    │   │   │   └── actions.js    // 活动页面的动作交互
    │   │   ├── error     // 404, 403 UI
    │   │   ├── event     // 事件页面的 actions stores view （与活动类同）
    │   │   ├── login     // 登录页面UI
    │   │   └── app.jsx     // layout UI
    │   ├── components    // 通用木偶(Dumb)组件
    │   ├── utils     // auth 服务，常用工具类
    │   ├── config.js     // 常量配置
    │   ├── main.js     // 把 router 组件绑定到 html
    │   └── routes.js     // 路由配置
    └── index.html
```
> 智能(smart)组件：含有状态(state)控制的组件。
> 木偶(Dumb)组件：没有状态(state)控制的展示组件，通常是可复用的部分。

## React Router 路由配置

路由通常是一个单页应用的核心，用来控制不同视图间切换。监听 location 中 hash 的变化来实现不同 components 的展视。
此应用的 URLs 与 components 对照：

| URL                               | Components |
| --------------------------------- | ---------- |
| `/`                               | `App` |
| `/activity`                       | `App -> ActivityManage` |
| `/activity/add`                   | `App -> ActivityManage -> ActivityAdd` |
| `/activity/:activityId/edit`      | `App -> ActivityManage -> ActivityAdd` |
| `/activity/:activityId/unit/add`  | `App -> UnitAdd` |
| `/activity/unit/:unitId/edit`     | `App -> UnitAdd` |
| `/activity/:id/detail`            | `App -> ActivityDetail` |
| `/activity/unit/:id/detail`       | `App -> UnitDetail` |
| `/event`                          | `App -> EventManage` |
| `/accessdenied`                   | `App -> AccessDenied` |
| `/*`                              | `App -> NotFound` |

### 代码片段
```js
# src/scripts/routes.js

import { Router, Route, IndexRoute, IndexRedirect, useRouterHistory, hashHistory } from 'react-router';
import { createHashHistory } from 'history'

const appHistory = useRouterHistory(createHashHistory)({ queryKey: true });

const routes = (
  <Router history={appHistory}>
    <Route path='/' component={ App } onEnter={onEnterApp}>
      <IndexRedirect to="activity" />
      <Route path='activity' component={ ActivityManage } onEnter={requireAuth.bind(this, roles.activity)} >
        <Route path='add' component={ActivityAdd} onEnter={requireAuth.bind(this, roles.activity)} />
        <Route path=':activityId/edit' component={ActivityAdd} onEnter={requireAuth.bind(this, roles.activity)} />
      </Route>
      <Route path="activity/:activityId/unit/add" component={UnitAdd} onEnter={requireAuth.bind(this, roles.activity)}></Route>
      <Route path="activity/unit/:unitId/edit" component={UnitAdd} onEnter={requireAuth.bind(this, roles.activity)}></Route>
      <Route path="activity/:id/detail" component={ActivityDetail} onEnter={requireAuth.bind(this, roles.activity)}></Route>
      <Route path="activity/unit/:id/detail" component={UnitDetail} onEnter={requireAuth.bind(this, roles.activity)}></Route>
      <Route path='event' component={ EventManage } onEnter={requireAuth.bind(this, roles.event)} />
      <Route path='accessdenied' component={AccessDenied}/>
      <Route path='*' component={NotFound}/>
    </Route>
  </Router>
);

function onEnterApp(nextState, replace, callback) {
  auth.authorize(null, callback);
}

function requireAuth(roles, nextState, replace, callback) {
  if(Principal.isIdentityResolved()){
    auth.authorize(roles, callback)
  }
}

export default routes;
```
### Router 的 history 属性
React Router 是基于history的。所以 history 是 Router 组件上主要的属性。这里的 History 有三种类型 browserHistory, hashHistory, createMemoryHistory。先简单介绍下这三种类型：
* **browserHistory:** 它利用 HTML5 History API 去操纵浏览器的 URL， 并创建真正的 URLs，如：`example.com/some/path`。同时需要后端程序的配合，把所有前端routes请求都响应index.html。常用在现代浏览器中，IE9及以下体验不好，会刷新整个页面。
* **hashHistory:** 它利用URL中的hash(#)值来实现视图的切换。如：`example.com/#/some/path`。这种方式不需要后端做任何操作，同时能兼容低版的浏览器。此项目选用的就是这种类型。
* **createMemoryHistory:** 此类型不能读写浏览器的地址栏URL。常被用在服务端渲染，也可用于测试和其它环境（如：React Native）.它和上面两种有点小区别，就是必须要先创建才能使用，这样也有助于测试。
   ```js
   const history = createMemoryHistory(location)
   ```
> **注意：**此项目中其实可以在 history 属性中直接引用 hashHistory 的，之所以用当前的方式，是因为参数 `{ queryKey: true}` 这个参数是个坑，后面详说。
> **queryKey:** 它用来控制地址栏中是否显示`?_k=12l0vf`这种随机生成的字符参数；它的作用是用来记录路由切换时的 state 数据；如果此值为`false`就不会显示这个串，但也不能接收到上个视图的数据。
> 当时强迫症使然，觉得这个串太碍眼，觉得没啥大用，所以就查了API把`queryKey: false`了，结果是我的眼睛爽了，但出现了 Bug，调试了好久。。。才发现是因为这个设置导致不能接收到 `location.state`，后来就把这个参数置为 `true`了，等价于使用 `hashHistory`。

### Route 组件的属性
* __path:__ 此属性设置路由的 URL，与父级拼接形成绝对路径，如：`/activity/:activityId/edit`
* __component:__ 此属性设置对应的组件（component）,也就是此路由对应的视图（view）
* __onEnter:__ 此属性绑定一个路由进入之前的事件回调函数，它有三个参数 nextState, replace, callback. 具体含义不再赘述，参阅官方API；其中第三个参数 callback 如果传入，代表是异步完成，否则是同步。

此项目的主页面逻辑较为复杂，个人感觉用 Reflux 搞的有点乱，最近在学习 [Redux](http://redux.js.org/) 架构，打算有时间重构一下。

