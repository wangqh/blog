title: 基于AngularJs-uiRouter-ocLazyLoader构建一个按需加载模块化的SPA且兼容ie8
tags:
  - angularJS
  - spa
  - ui-router
  - oc.lazyLoad
  - 兼容ie8
date: 2018-03-01 15:40:10
categories: 术业专攻
---


笔者项目中，模块是按业务划分的，每个模块单独管理自己的路由，而不是把路由都放在app（主）模块中；这样做可移植性更高，更加解耦。不过要实现这种模式也遇到一些问题：
  1. 主模块中没有业务模块的路由，怎么加载视图？ 是指首次加载的页面视图
  2. 业务模块之间怎么相互跳转？

文中会详述这些问题的解决方案。
<!-- more -->

## 环境准备
1. nodejs, npm 或 yarn
2. 构建工具 gulp 或 Grunt
3. angular 1.2.32 此版本兼容ie8
4. 核心依赖包：angular-ui-router 0.4.2, oclazyload 1.1.0


## 按需加载
懒加载文件依赖第三方angular模块oc.lazyLoad。主要作用就是按照模块名懒加载该模块的文件，包括js, css和其它文件。
在主模块中配置业务模块信息:
{% codeblock app.js %}
      angular
          .module('app', [
              'ui.router', 'oc.lazyLoad'
          ])
          .config(appConfig)

      /** @ngInject */
      function appConfig($ocLazyLoadProvider, LOAD_MODULES){
           $ocLazyLoadProvider.config({
             modules: LOAD_MODULES,
             debug: false
           })
      }
{% endcodeblock %} 
{% codeblock app.constant.js %}
(function(){
    'use strict';

      var version= '?v=1.0.0'; // 去缓存

      angular
        .module('app')
        .constant('LOAD_MODULES', [
          {
            name: 'ie8Polyfill',
            files: ['/assets/scripts/respond.min.js']
          },
          {
            name: 'layout',
            files: ['components/layout/main.js' + version, '/assets/styles/layout.css' + version]
          },
          {
            name: 'home',
            files: ['app/home/home.js' + version]
          }
        ]);
}());
{% endcodeblock %} 

加载模块的方法，利用$ocLazyLoad服务的load函数:
{% codeblock %}
//返回一个promise对象
$ocLazyLoad.load(moduleName).then(function() { 
  //加载完成后执行
});
{% endcodeblock %} 

## 路由改造
当用户直接输入地址或者从外部链接访问页面时，预先加载业务模块，然后由业务模块的路由来处理页面的渲染，这样就解决了**开篇提到的第一个问题**。实现代码如下：
{% codeblock app.js %}
      angular
          .module('app', [
              'ui.router', 'oc.lazyLoad'
          ])
          .run(appRun)

      /** @ngInject */
      function appRun($rootScope, $ocLazyLoad, $urlRouter, $location){
        // 当浏览器地址栏的URL变更时触发
        $rootScope.$on('$locationChangeSuccess',  function(event) {
          // 阻止原生事件, 不触发ui.router, 手动调用$urlRouter.sync();
          event.preventDefault();

          if(!$location.path() || $location.path() === '/') {
            $location.path('/home') // 转到首页
          } else {
            
            var mod = $location.path().split('/')[1]; 
  
            // 路由路径按照 "/模块/页面"的方式配置： 
            // 1. 避免不同模块的路径冲突 
            // 2. 可以通过路径判断模块
            $ocLazyLoad.load(mod).then(function () {
              $urlRouter.sync();
            });
          }

        });
{% endcodeblock %} 

ui.router常用的跳转方式有两种：一种是使用指令`ui-sref`,另一种是使用函数`$state.go()`。但是它们的底层实现都依赖了`$state.transitionTo()`方法。
如果首次从一个业务模块直接跳转到另一个模块是会报错的，因为后者并没有被加载过，所以找不到相应的路由。这时就需要对`$state.transitionTo()`改造一下，来实现模块预先加载，也就解决了**开篇提到的第二个问题**。
具体实现代码如下：
{% codeblock app.js %}
  angular
      .module('app', [
          'ui.router', 'oc.lazyLoad'
      ])
      .config(configDecorator)

    /** @ngInject */
    function configDecorator($provide){

      //处理模块间跳转 
      $provide.decorator('$state', decoratorFn)
      
      /** @ngInject */
      function decoratorFn($delegate, $ocLazyLoad) { 
          var state = {}; 
          // 这里做深拷贝不做浅拷贝是避免循环嵌套调用内存溢出 
          angular.copy($delegate, state); 

          $delegate.transitionTo = function (to, toParams) { 
            var stateName;

            // 跳转的时候有两种情况，一种是传入self对象，另一种是直接把state的name传进来 
            if (to.self) { 
              stateName = to.self.name;
            } else { 
              stateName = to
            } 
            var mod = stateName.split('.')[1]; 

            //模块加载完成后再调用默认的路由跳转函数 
            $ocLazyLoad.load(mod).then(function (){ 
              state.transitionTo.apply(null, [stateName, toParams]); 
            }); 
          }
          
          return $delegate;
        }
      
    }
{% endcodeblock %} 
> 注： angular中的decorator不太常用，它的作用是修改第三方模块的服务，且不改变源码。

## 兼容IE8
angular从1.3+就不再ie8了，除非引入h5,json3等一些填缝剂，但那样比较影响性能。当然，如果是对性能要求不高的项目也可以使用更高版本。
不过既然用了按需加载，势必对性能是有要求的，所以笔者选用了1.2.32版本（目前是1.2中最高版）
angular 1.2.x版本都是支持ie8的，但实际开发中还是要注意一些问题：
1. 如有引用bootstrap, 需要引入respond.js. 因为ie8不支持bootstrap的栅格系统，所以需要额外引入一些补丁来实现。一定要在载入栅格视图之前加载ie8修补文件，代码如下：
  {% codeblock %}
    if(nav.appName == 'Microsoft Internet Explorer' && nav.appVersion.split(';')[1].replace(/[ ]/g, '') == 'MSIE8.0'){
      // IE8 fix
      $ocLazyLoad.load('ie8Polyfill')
    }
  {% endcodeblock %} 
  > 注：ie8Polyfill 在上文配置模块信息中已经定义
2. 不要直接使用ie8的一些保留字，如：catch, delete等，否则会报错。可以这样使用它们`Promise['catch']` `$http['delete']`

## 总结
利用构建工具把每个模块下的html模板、js文件合并成一个文件，less转换成css；最终构建完的模块一般只有两个文件：一个js,一个css.
这样以来，在生产环境中按需加载模块时，最多加载两个文件就可以了，从而提高页面的访问速度，提升用户体验。
