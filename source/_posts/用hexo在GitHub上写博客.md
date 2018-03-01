title: 用hexo在GitHub上写博客
date: 2016-05-27 14:11:07
categories: 触类旁通
tags: [nodejs, git, hexo, blog]
---

hexo出自台湾大学生 [tommy351](http://twitter.com/tommy351) 之手，是一个基于Node.js的静态博客程序，其编译上百篇文字只需要几秒。hexo生成的静态网页可以直接放到GitHub Pages，BAE，SAE等平台上。
<!-- more -->
## 环境准备

* 安装NodeJS
* 安装Git
* 自己喜欢的编辑器或者IDE

{% blockquote %}
node和git安装方法官网都有，这里不再赘述；IDE本人用的 Webstorm，适合项目多的使用，比较好用的编辑器有 Subline Text 等
{% endblockquote %}

## 安装
Node 和 Git 都安装好后，可执行如下命令安装hexo:
{% codeblock %}
npm install -g hexo-cli
{% endcodeblock %}

## 初始化
然后，执行init命令初始化hexo到你指定的目录：
{% codeblock %}
hexo init <folder>
cd <folder>
npm install
{% endcodeblock %}
{% blockquote %}
也可以先cd到目标目录，然后再执行hexo init.
{% endblockquote %}

## 生成静态页面
cd 到你的init目录，执行如下命令，生成静态页面至 hexo\public\ 目录：
{% codeblock %}
hexo generate
{% endcodeblock %}
{% blockquote %}
命令必须在init目录下执行，否则不成功，但也不会报错。
当你修改文章Tag或内容，不能正确的重新生成内容，可以删除 hexo\db.json 后重试，还不行就到 public 目录删除对应的文件，重新生成。
{% endblockquote %}

## 本地启动
执行如下命令，启动本地服务，进行文章预览调试。
{% codeblock %}
hexo server
{% endcodeblock %}
浏览器输入 [http://localhost:4000](http://localhost:4000) 就可以看到效果

## 写文章
执行new命令，生成指定名称的文章至 hexo\source\_posts\postName.md。
{% codeblock %}
hexo new [layout] "postName"
{% endcodeblock %}
1. 其中 layout 是可选参数，默认值为 post。有哪些 layout 可以在 scaffolds 目录下查看，这些文件名称就是 layout 名称。当然你也可以添加自己的 layout, 方法就是添加一个文件即可，同时你也可以编辑现有的 layout，比如 post 的 layout 默认是 hexo\scaffolds\post.md
{% codeblock %}
title: { { title } }
date: { { date } }
tags:
---
{% endcodeblock %}
{% blockquote %}
注意：大括号与大括号之间我多加了空格，否则会读出本博文的变量。
{% endblockquote %}
我想添加Categories, 以免每次手工输入，只需要修改这个文件添加一行，如下：
{% codeblock %}
title: { { title } }
date: { { date } }
categories:
tags:
---
{% endcodeblock %}
2. postName是md文件的名字，同时也出现在你文章的URL中，postName如果包含空格，必须用""将其包围，postName可以为中文。
看一下刚才生成的文件 hexo\source\_posts\postName.md ，内容如下：
{% codeblock %}
title: postName #文章页面上的显示名称，可以任意修改，不会出现在URL中
date: 2016-05-27 14:11:07 #文章生成时间，一般不改，当然也可以任意修改
categories: #文章分类目录，可以为空，注意:后面有个空格
tags: #文章标签，可空，多标签请用格式[tag1,tag2,tag3]，注意:后面有个空格
---
这里开始使用markdown格式输入你的正文。
{% endcodeblock %}
接下来，你就可以用喜爱的编辑器尽情书写你的文章。关于markdown语法，可以查阅官网API。

## 文章摘要
在需要显示摘要的地方添加如下代码即可：
{% codeblock %}
以上是摘要
<!-- more -->
以下是余下全文
{% endcodeblock %}
more以上内容即是文章摘要，在主页显示，more以下内容点击『 More >> 』链接打开全文才显示。
{% blockquote %}
注意：hexo中所有文件的编码格式均是UTF-8。
{% endblockquote %}

## 主题安装
萝卜白菜各有所爱，玩博客换主题是必不可少的，hexo 的主题列表 [Hexo Themes](https://hexo.io/docs/themes.html) 。
我比较喜欢 [pacman](http://github.com/A-limon/pacman) ， [yelee](https://github.com/MOxFIVE/hexo-theme-yelee) 。 Pacman 简洁大方小清新，同时移动版本支持的也很好，但作者并没有把很多参数分离出来给出可配置项，我最终选择了 yelee, 个人感觉效果更炫一点 。
1. 安装主题的方法就是一句git命令：
{% codeblock %}
git clone https://github.com/MOxFIVE/hexo-theme-yelee.git themes/yelee
{% endcodeblock %}

2. 安装完成后，打开 hexo\_config.yml ，修改主题为 yelee
{% codeblock %}
theme: yelee
{% endcodeblock %}
打开 hexo\themes\yelee 目录，编辑主题配置文件 _config.yml ；详细配置参照 [Yelee API](https://github.com/MOxFIVE/hexo-theme-yelee)

3. 更新主题
{% codeblock %}
cd themes/yelee
git pull
{% endcodeblock %}

## 404 页面
GitHub Pages 自定义404页面 非常容易，直接在根目录下创建自己的 404.html 就可以。但是自定义404页面仅对绑定顶级域名的项目才起作用，GitHub默认分配的二级域名是不起作用的，使用 hexo server 在本机调试也是不起作用的。
其实，404页面可以做更多有意义的事，来做个404公益项目吧。做点有意义的事情，也对得起这个域名。
目前有如下几个公益404接入地址，我选择了腾讯的。404页面，每个人可以做的更多。
* [腾讯公益404](http://www.qq.com/404/)
* [404公益_益云(公益互联网)社会创新中心](http://yibo.iyiyun.com/Index/web404)
* [失蹤兒童少年資料管理中心404](http://404page.missingkids.org.tw/)

## 申请域名（可选）
GitHubPages默认为每个用户分配了一个二级域名『your_user_name.github.io』。
如果你对上述域名不满意，可以到 [狗爹](http://www.godaddy.com/) 上申请一个自己的域名，然后绑定到GitHub Pages。绑定方法很简单，在repo根目录下建立一个CNAME文件，里面写上域名即可。

### GoDaddy
买域名首选狗爹，国内的服务商大家都懂的。
目前.site域名只要￥20，但据说续费比较贵，我是先玩下，一年后再换，至于搜索引擎重新索引之类的，无所谓。.me和.com域名稍微贵点，大约￥60-100，网上有很多优惠码可用，可惜有的优惠码有限制。比如有个.com域名优惠码只要$0.99，但只能用国外信用卡购买。更多优惠码可以自行谷歌或到 [独特优惠码](http://www.dute.me/) 找。不着急的同学可以将中意的域名加入购物车先不付款，过几天，狗爹就会发优惠信息给你。狗爹不定期也会有活动，可以多关注。
付款后，需要稍微等一会你才会拿到域名，特别是支付宝付款的，要等大约半小时左右。此外域名要一年年的买，这样比较划算。

### DNSPod
GoDaddy的NameServers有时会被墙，因此墙裂推荐国内的 [DNSPod](http://www.dnspod.cn/) 解析域名，免费服务真心不错。支持微信/邮件提醒，监控与报警，访问统计，健康诊断，搜索引擎推送，速度哇哇的，对于我来说足够。
两步设置就可以搞定，怎么操作参考 [Godaddy注册商域名修改DNS地址](http://support.dnspod.cn/Kb/showarticle/tsid/42) 。

## 命令
### 常用命令
{% codeblock %}
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo new draft "postName" #新建草稿
hexo publish "postName" #发布草稿到文章
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #将.deploy目录部署到GitHub
{% endcodeblock %}
### 常用复合命令
{% codeblock %}
hexo deploy -g
hexo server -w
{% endcodeblock %}
### 简写
{% codeblock %}
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy
{% endcodeblock %}