title: 销售员专用（hybrid app）混合应用
date: 2015-07-14 16:25:12
categories: 项目设计
tags: 
- hybrid
- nodejs
- angularJs
- cordova
- mongoDB
---

一个移动端（hybrid app）混合应用，销售员通过域帐号认证登录，查看从 AP 系统推送过来的信息，一键拨打客户电话并且确认是否跟踪订单。
<!-- more -->
## 用到的技术
| 移动端 | 服务端 |
|:-------------------:|:-------------------:| 
| [Angular.js](http://angularjs.org/) ![Angular.js](https://avatars1.githubusercontent.com/u/139426?s=30) | [Node.js](http://nodejs.org/) <img src="http://nodejs.org/images/logo-light.svg" height="30" width="80" /> |
| [Ionic](http://ionicframework.com/) <img src="http://upload.wikimedia.org/wikipedia/commons/d/d1/Ionic_Logo.svg" height="45" width="80" /> | [MongoDB](http://www.mongodb.org/) ![MongoDB](https://avatars3.githubusercontent.com/u/45120?v=2&s=30) |
| [Material Design](https://material.angularjs.org/) ![Angularjs](https://avatars1.githubusercontent.com/u/139426?s=30) | [Express.js](http://expressjs.com/)<img src="https://cldup.com/wpGXm1cWwB.png" height="40" width="145"> |
| [Cordova](https://cordova.apache.org/) <img src="https://cordova.apache.org/images/cordova_256.png" height="35" width="45" />  &nbsp; [JPush](https://www.jpush.cn/)极光推送 | [Redis](http://redis.io/) <img src="http://upload.wikimedia.org/wikipedia/en/thumb/6/6b/Redis_Logo.svg/467px-Redis_Logo.svg.png?v=2&s=30" height="35" width="125"> | 

## 应用架构
一个端到端的应用架构
{% asset_img salesperson-diagram.png [应用架构] %}

### 数据库层
选择MongoDB数据库,因为它易于存储JSON文档，且比较灵活。数据库有3个(collections)集合。

1. **users** - 存储销售员基本信息, 下面是doc结构：
{% codeblock User %}
    {
        name:  "姓名",
        username: "用户名",
        password: "密码",
        salt: "",
        sex: "性别",
        nation: "民族",
        provider: "local",// passport strategy
        updated: Date,
        created: Date
    }
{% endcodeblock %} 
    
2. **orders** - 存储销售员的订单信息, 下面是doc结构：
{% codeblock Order %}
    {
        "customer": {
            "id": "id43214",
            "name": "李文",
            "phone": "15210104324", //电话
            "company": "中国海外建设" //公司名称
        },
        "salesperson": "wangqh-a", //销售员ID
        "toTrack": false, //是否跟进
        "hasPirate": true // 是否有盗版
    }
{% endcodeblock %} 

3. **projects** - 存储订单中的工程信息, 下面是doc结构：
{% codeblock Project %}
    {
        "name": "工程名称",
        "hasPirate": true, // 是否有盗版
        "keys": [
            {
                "keyNumber": "v423423",
                "isPirate": true, // 是否为盗版
                "notes": "", // 备注
                "usedTime": 432142214324 // 使用时间
            }
        ],
        "order": "orderId" // 与 order 关联
    }
{% endcodeblock %} 


从ＡＰ推送过来的数据结构:
{% codeblock AP 推送信息结构 %}
    {
        "customer": {
            "id": "id43214",
            "name": "李文",
            "phone": "15210104324", //电话
            "company": "中国海外建设" //公司名称
        },
        "salesperson": "wangqh-a", //销售员ID
        "toTrack": false, //是否跟进
        "hasPirate": true, // 是否有盗版
        "projects": [ 
            {
                "name": "工程名称",
                "hasPirate": true, // 是否有盗版
                "details": [
                    {
                        "keyNumber": "v423423",
                        "isPirate": true, // 是否为盗版
                        "notes": "", // 备注
                        "usedTime": 432142214324 // 使用时间
                    }
                ]
            }
        ]
    }
{% endcodeblock %} 

### REST 服务层
这是整个应用的核心。已认证过的用户，向服务层请求时，根据不同请求作不同的数据库操作。

* 使用Node.js作为服务器
* 使用express框架实现 REST API
* 使用mongoose实现数据模型
* 使用JWT生成和验证Token
* 使用Redis管理与存储Token

**服务端 Controller**
1. Sign In: **登录** 先在本地库中查询是否存在用户，存在即返回 Token；
    不存在就向公司域帐号请求认证，成功后向hr系统请求用户信息并保存到数据库。
2. Sign Out: **退出** 删除当前 Token
3. Orders: **订单** 接收从AP系统推送的订单，并用JPush推到前端。 另外，跟踪订单：把订单的跟踪状态置为 true
4. Projects: **工程** 根据orderId获得工程列表，或用projectIda 获取 keys(工程内的加密锁列表)

### 客户端层
下列是主要用到的技术
* 主框架使用 IONIC(cordova + angularJs)
* 主题使用 Angular Material
* 消息推送使用 jPush 第三方服务


** 客户端 Controller **
1. Sign In: 登录页
2. Account: 销售员个人信息页面
3. Orders: 订单列表页面，列出ap系统推送给销售员的订单
4. Projects: 工程列表页面，订单的下级页面，列出此订单里的所有工程
5. keys: 工程详情页，工程的下级页面，列出此工程里的加密锁列表