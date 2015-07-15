title: 销售员专用 APP
date: 2015-07-14 16:25:12
categories: 设计文档
tags: 
- hybrid
- nodejs
---

一个移动端（hybrid app）混合应用，销售员通过域帐号认证登录，查看从 AP 系统推送过来的信息并且可以一键拨打客户电话。 

## 用到的技术
| 移动端 | 服务端 | 开发工具 | 
|:-------------------:|:-------------------:|:-------------------:| 
| [Angular.js](http://angularjs.org/) [![Angularjs](https://avatars1.githubusercontent.com/u/139426?s=30)]() | [Node.js](http://nodejs.org/) <img src="http://nodejs.org/images/logo-light.svg" height="30" width="80" /> | [Gulp](http://gulpjs.com/) ![Gulp](https://avatars0.githubusercontent.com/u/6200624?s=30) &nbsp; [Bower](http://bower.io/) ![Bower](https://avatars3.githubusercontent.com/u/3709251?s=30) | 
| [Ionic](http://ionicframework.com/) <img src="http://upload.wikimedia.org/wikipedia/commons/d/d1/Ionic_Logo.svg" height="45" width="80" /> | [MongoDB](http://www.mongodb.org/) ![MongoDB](https://avatars3.githubusercontent.com/u/45120?v=2&s=30) | [NPM](https://www.npmjs.org/) ![NPM](https://avatars0.githubusercontent.com/u/6078720?s=30) &nbsp; [Ansible](https://www.ansible.com/) ![Ansible](https://avatars3.githubusercontent.com/u/1507452?v=2&s=30) | 
| [Material Design](https://material.angularjs.org/) ![Angularjs](https://avatars1.githubusercontent.com/u/139426?s=30) | [Express.js](http://expressjs.com/)<img src="https://cldup.com/wpGXm1cWwB.png" height="40" width="145"> | [Vagrant](http://www.vagrantup.com/) <img src="https://www.vagrantup.com/images/logo_vagrant-81478652.png" height="40" width="145"> | 
| [Cordova](https://cordova.apache.org/) <img src="https://cordova.apache.org/images/cordova_256.png" height="35" width="45" /> | [Redis](http://redis.io/) <img src="http://upload.wikimedia.org/wikipedia/en/thumb/6/6b/Redis_Logo.svg/467px-Redis_Logo.svg.png?v=2&s=30" height="35" width="125"> | &nbsp; | 

## 应用架构
一个端到端的应用架构
{% asset_img salesperson-diagram.png [应用架构] %}

### 数据库层
选择MongoDB数据库,因为它易于存储JSON文档，且比较灵活。数据库有2个(collections)集合。

1. **User** - 存储销售员基本信息
2. **Order** - 存储销售员接收到的订单信息

### REST 服务层
这是整个应用的核心。已认证过的用户，向服务层请求时，根据不同请求作不同的数据库操作。

* 使用Node.js作为服务器
* 使用express框架实现 REST API
* 使用mongoose实现数据模型
* 使用JWT生成和验证Token
* 使用Redis管理与存储Token

### 客户端层
下列是主要用到的技术
* 主框架使用 IONIC(cordova + angularJs)
* 主题使用 Angular Material
* 消息推送使用 jPush 第三方服务