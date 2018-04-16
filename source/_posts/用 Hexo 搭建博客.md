---
title: 用 Hexo 搭建博客
date: 2017-09-21 15:07:28
tags:
---

## 尝试自己写博客
------
当然先要参考些其它文章，然后自己再搞，发现这篇文章还是有些参考价值[http://lowrank.science/Hexo-Github/](http://lowrank.science/Hexo-Github/)

然后自己开始搭，终于搭建起来了 没有主题，没有自己的域名，只是一个普通的博客，先写一下试试感觉，哈哈哈
![](/img/20170921_01.jpg)

## 为何自己要写博客？

以前做过的东西老是忘，虽然后来能百度到自己所需要的东西，但是肯定是费时费力，有些东西是可以记录一下的，这样下次再遇到,能第一时间知道自己以前遇到过，处理思路和方法就能出现在脑海里了，而写博客就能加深对一个东西的理解，而且能够锻炼自己写作能力，梳理思路，让脑子变得更清晰,哈哈哈 浅显的理解

## 怎么搭建博客？
### 1.安装前准备
a.安装Node.js(可以去[中文官网](http://nodejs.cn/)下载按照)
Node.js:基于 Chrome V8 引擎的 JavaScript 运行环境，是JavaScript运行在服务端的一项技术，Node.js是单线程的。Node.js 的包管理器 npm，是全球最大的开源库生态系统。

b.安装git或github客户端(用于跟github那边进行数据的传输)
### 2.利用npm安装hexo
Hexo 是一个基于nodejs 的静态博客网站生成器，作者是来自台湾的 Tommy Chen

特点：
>  * 不可思议的快速 ─ 只要一眨眼静态文件即生成完成
>  * 支持 Markdown
>  * 仅需一道指令即可部署到 GitHub Pages 和 Heroku
>  * 已移植 Octopress 插件
>  * 高扩展性、自订性
>  * 兼容于 Windows, Mac & Linux
进一步入门了解可以看这篇博客
[hexo —— 简单、快速、强大的Node.js静态博客框架](https://segmentfault.com/a/1190000000370778)

c.终端安装
```JavaScript
$ sudo npm install -g hexo
```

d.初始化
```JavaScript
$hexo init myhexo
```

myhexo是你建立的文件夹名称。cd到myhexo文件夹下，执行如下命令，安装npm：
```JavaScript
$ npm install
```

执行如下命令，开启hexo服务器：
```JavaScript
$ hexo s
```

此时，浏览器中打开网址http://localhost:4000，能看到如下页面：
![](/img/hexo4000.png)

本地设置好后，接下来开始关联Github。

怎么配置github自己查资料去吧,已经关联过很多次了
![](/img/20170921_02.jpg)


### 3.上传和更新文章

如果自己要写新文章，而不是让博客里只有一个简单的hello world
```JavaScript
$hexo n 'newPage name'
```

配置好后，先生成静态页面，不生成，就无法显示之前配置的效果
```JavaScript
$hexo g
```
然后直接配置到github上，因为之前设置了ssh，所有可以直接上传，因为ssh是加密的数据流，具有一定的安全性
```JavaScript
$hexo d
```


每次修改完本地的文件后要执行以下命令，重新部署到github上：
hexo clean
hexo generate
hexo deploy或者hexo d
常用命令

```
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #将.deploy目录部署到GitHub
hexo help # 查看帮助
hexo version #查看Hexo的版本
```
Hexo简写命令

```
hexo n "postName" #新建文章，或者source\_posts手动编辑
hexo g #生成public静态文件至public目录
hexo s #本地发布预览效果 http://localhost:4000 ('ctrl + c'关闭server)
hexo d #将.deploy目录部署到GitHub
```
Hexo复合命令

```
hexo d -g #生成并部署上传
hexo s -g #生成并本地发布预览
hexo clean && hexo d -g #清空缓存然后生成并部署上传
hexo clean && hexo s -g #清空缓存然后生成并本地发布预览
```

此时可以打开
用户名.github.io来欣赏自己的搭建博客了

这样就可以在你的hexo主文件夹\source\_posts(我新生成的文章文件在文稿目录下的 myhexo\source\_posts中)
然后利用markdown语法，对md文件进行博客内容的编写，markdown可以是一种提高写作效率语法，目的让我们更加专注于内容，可以看看这篇博客[Markdown语法说明](http://www.markdown.cn/)

## 还有什么可以折腾的呢？
* 用自己的域名跟自己创建的github博客进行绑定
* 找到一个适合自己的主题
* 寻找一些合适的编辑博客的插件

先写这么多吧，哈哈哈

