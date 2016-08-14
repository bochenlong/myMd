### 前言
想搭建个人博客，Github+Hexo是个不错的选择。

### 为什么选择Github+Hexo

* Git - 最先进的开源分布式版本控制系统
* GitHub - 全球最大的社交编程及代码托管网站
	
	* Github Pages - Github提供的**免费**的静态站点，Github pages具备两种模式：   
		1 User/Organization Pages个人或公司站点   
		2 Project Pages项目站点
* Hexo快速、简洁且高效的博客框架 

	* 急速生成页面
	* 支持Markdown
	* 一键部署博客
	* 丰富的插件、主题支持

### 创建Github Pages
1. Github网站注册账号
2. 创建仓库username.github.io   

> 需要注意，新建仓库名称必须符合标准要求，username为Github账号的用户名称，建立成功之后，Github会自动将识别此仓库为Github Pages站点

### 创建博客
> 以下为Mac环境操作记录，个别安装请根据自己系统调整

#### Hexo安装

* 安装依赖软件
	
	* Node.js ( 含npm )
	
		`$ brew install node`
	* Git
		
		`$ brew install git`
* 安装Hexo   
	
	`$ sudo npm install hexo-cli -g`	

	> Linux / OS X 系统下需要 sudo 管理员安装，否则会出现以下错误。如果出现可以尝试 **npm install hexo --no-optional** 安装解决。但此错误并不影响Hexo正常使用。
		
	```
	{ [Error: Cannot find module './build/Release/DTraceProviderBindings'] code: 'MODULE_NOT_FOUND' }    
	{ [Error: Cannot find module './build/default/DTraceProviderBindings'] code: 'MODULE_NOT_FOUND' }
	{ [Error: Cannot find module './build/Debug/DTraceProviderBindings'] code: 'MODULE_NOT_FOUND' }
	```
<!-- more -->	
#### 创建博客

* 创建博客  

	```
	$ hexo init demo  
	$ cd demo   
	$ npm install
	```
	以上命令会自动初始化博客目录，具体目录不再列出；
* 启动博客
	
	```
	$ hexo server
	// -p 重设端口，默认4000
	```
	打开浏览器http://localhost:4000/ （默认端口为4000），便可看到Hexo默认的博客。

### 安装NexT主题
* 下载NexT主题

	```
	$ cd your-hexo-site
	$ git clone https://github.com/iissnan/hexo-theme-next themes/next
	```
* 启用主题   
	打开 **站点配置文件**，找到 *theme* 字段，并将其值更改为 *next* 。`theme: next`；启动Hexo，即可看到NexT的主题页面，即本站主题。   

### 个性化配置
到现在，我们完成了网站的初步建设。接下来，我们给网站装扮一番，让它更加实用和个性化。
#### 网站基本配置

* 网站信息设置：   
	* **网站标题**   
	修改站点配置文件:`title: 某某的博客啦`    
	如果有副标题，则改：`subtitle: 明智图新`
	* **建站时间**   
	修改主题配置文件:`since: 2013`
	* **作者名称**  
	修改站点配置文件:`author: yourname`
	* **网站语言**   
	修改站点配置文件:`language: zh-Hans`
	* **网站时区**   
	修改站点配置文件:`timezone: Asia/Shanghai`   
	* **网站地址**   
	修改站点配置文件:`url: http://yourusername.github.io`
	* **网站订阅RSS设置**   
	1， 安装插件 hexo-generator-feed   
	`npm install hexo-generator-feed --save`   
	2， 修改站点配置文件
		
	```	
	feed:
	  type: atom
	  path: atom.xml
	  limit: 20
	  hub:
	```
	> **type** - Feed type. (atom/rss2)   
	**path** - Feed path. (Default: atom.xml/rss2.xml)   
	**limit** - Maximum number of posts in the feed (Use 0 or false to show all posts)   
	**hub** - URL of the PubSubHubbub hubs (Leave it empty if you don't use it)
	* **作者头像**   
	修改主题配置文件：`avatar: /images/avatar.png`   
	*个人头像可放置主题路径 source/images 或者网站路径 source/uploads*
	* **站点描述**   
	修改站点配置文件:`description: 唯有努力不负人`   
	*站点描述可以是你喜欢的一句签名:）*
	* **社交账号**   
	修改主题配置文件:   
	1 账号连接：  
	
	```
	social:
	  GitHub: https://github.com/your-user-name
	  Twitter: https://twitter.com/your-user-name  
	  微博: http://weibo.com/your-user-name
	  # 等等
	```
	2 连接图标： 
	  
	```
	social_icons:
	  enable: true
	  GitHub: github
	  Twitter: twitter
	  微博: weibo
	```
	* **友情连接**   
	修改主题配置文件:
	
	```
	links_title: Links
	links:
	  MacTalk: http://macshuo.com/
	  Title: http://example.com/
	```

#### 标签/分类页面
新建标签页面

```
$ cd your-hexo-site
$ hexo new page tags
```
设置页面类型
```
title: 标签
date: 2014-12-22 12:39:04
type: "tags"
comment: "false"
---	
```
`commet: false`是关闭标签页面的评论   
分类同标签一样，将tags关键字替换成categories即可

#### 文章打赏
修改主题配置文件：

```
reward_comment: 我知道是不会有人点的，但万一有人想不开呢？
wechatpay: /images/wechat-reward.png
alipay: /images/alipay-reward.png
```
#### 文章点击次数   
文章点击次数使用leanclound。前往https://leancloud.cn/注册一个新账户。   
1 创建应用 
![LeanCloud01](http://o7ar2k9lr.bkt.clouddn.com/LeanCloud01.png)    
2 点击图1的存储，配置数据库；创建class，切记class名称必须为 Counter。
![LeanCloud01](http://o7ar2k9lr.bkt.clouddn.com/LeanCloud02.png)
3 点击图1右上角配置，复制应用Key下的App ID/App Key。修改主题配置文件
```
leancloud_visitors:
  enable: true
  app_id: yourappid
  app_key: yourappkey
```
#### 文章评论   
评论使用disqus。前往https://disqus.com/注册一个新账户。   
选择右上角个人信息下的 install on site，根据提示一步一步创建站点信息。
![disqus03](http://o7ar2k9lr.bkt.clouddn.com/disqus03.png)
创建过程中配置的Disqus URL即为Disqus的shot_name，修改主题配置文件
`disqus_shortname: yourdisqusshotname`
#### 站内搜索
搜索使用Swiftype。前往https://swiftype.com/注册一个新账户。   
1 创建一个搜索引擎
![Swiftype01](http://o7ar2k9lr.bkt.clouddn.com/Swiftype01.png)
2 创建成功之后，即进入引擎控制台。在Overview中可安装站内搜索和订制搜索引擎。
![Swiftype02](http://o7ar2k9lr.bkt.clouddn.com/Swiftype02.png)
选择Edit Install Setup，安装搜索到网站。获取Key.
![Swiftype03](http://o7ar2k9lr.bkt.clouddn.com/Swiftype03.png)
![Swiftype05](http://o7ar2k9lr.bkt.clouddn.com/Swiftype05.png)
3 修改主题配置文件   
`swiftype_key: yourswiftpekey`

### 发表文章部署到Github

* 创建文章 `$ hexo new <postname>`
* 部署到Github
	
	```
	$ hexo clean
	$ hexo generate
	```
	下载Github部署的依赖查插件：`$ npm install hexo-deployer-git --save`
	修改主题配置文件：
	
	```
	deploy:
	  type: git
	  repo: git@yourgit
	  branch: master
	  message: 
	  name: username
	  email: email
	```
	执行`$ sudo hexo deploy`
	> 特别注意，执行部署需要root权限。sudo hexo deploy命令执行的时候应该会去读取的oot用户的公钥，因此需要生成且在Github网站配置root用户的SSH配置。否则会一直`Error: Permission denied (publickey). fatal: Could not read from remote repository.`
	
**参考**   
Hexo官网：<https://hexo.io/zh-cn/>   
NexT官网：<http://theme-next.iissnan.com/>