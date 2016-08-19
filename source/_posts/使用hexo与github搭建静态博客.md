title: 使用hexo与github搭建静态博客
date: 2016-02-15
tags: 博客相关
---
前言：本文是一篇简单的独立博客搭建教程，内容不够完善，在我使用博客的过程中会不断实践并进行补充。<!--more-->


## 1.安装node.js ##
下载[node.js](http://nodejs.cn/ "node.js中文地址")（node.js中文站偶尔抽风打不开，可以访问[node.js英文官网](https://nodejs.org/en/)）安装过程很简单。

## 2.安装hexo ##
找个地方新建个文件夹，命名随意，我们先假设将其命名为hexo。在新建的hexo文件夹内最上面的路径显示框中输入CMD,然后回车打开命令行程序并且路径是当前文件夹。

注意：本文中hexo相关所有命令都要在此站点目录下。

在命令行界面中执行如下命令安装hexo：

    npm install -g hexo

然后输入如下命令自动生成建立网站所需要的文件：

    hexo init
然后按照提示，运行如下命令安装node_modules：

    npm install
这时hexo基本的安装就已经完成，运行如下命令启动本地hexo服务：

    hexo server

提示服务启动后在浏览器中打开[http://localhost:4000/](http://localhost:4000/)进行预览，这时可以看到Hexo已自动生成了一篇blog，停止服务按Ctrl+C。

## 3.新建文章 ##

在上步命令行窗口中执行：

    hexo new "新建文章标题"
该命令会在\source\_posts目录下心生成一个.md为后缀的[markdown](http://baike.baidu.com/link?url=xNu_6Vbv6T8zOIdoIMAqofazcCQFYyLlskDSddWSjq9N3OhbPoLOX4zCuHY0kD2gEKGlyzbO9EFBpsgG_Ymfhq)文件。当然也可以直接在\source_posts中新建一个md文件。用markdown编辑器将其打开就可以写文章了。

markdown编辑器我使用的是[markdownpad](http://markdownpad.com/)。

在写文章时如果需要使用图片，可以先把图片上传到github中再引用。github使用方法在下文。

hexo首页默认显示的是文章全文，如果你想只显示文章的简介，就在文章中你想在首页显示的部分后加上 
    
    <!--more-->


## 4.更换主题 ##

hexo有很多主题[可以从这里挑个](http://www.zhihu.com/question/24422335)。

以[hexo-theme-yilia](https://github.com/litten/hexo-theme-yilia)主题为例，在命令行窗口中执行如下命令将主题文件克隆到themes文件夹中：

    git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
然后打开站点配置文件_config.yml，找到theme 字段，并将其值更改为 yilia。然后执行

    hexo server
并在浏览器中打开[http://localhost:4000/](http://localhost:4000/)进行预览，看是否更换主题成功。

修改部分网站信息比如标题、描述、语言设置也是在_config.yml文件中。

## 5.404网页 ##

404页面网上有很多模板，推荐使用[腾讯的寻子公益404页面](http://www.qq.com/404/)。

在/source目录下新建txt文件，将下面代码复制粘贴到其中

    <script type="text/javascript" src="http://www.qq.com/404/search_children.js" charset="utf-8"></script>
然后保存，将其重命名为404.html。

404网页在本地预览看不到效果，上传到github后可以查看。

## 6.部署到github ##

注册登录[github官网](https://github.com/),点击网页右侧加号New repository。

仓库名必须与你用户名对应，格式为owner_name.github.io，比如我的github名为zhaochenpu,博客的仓库名就是zhaochenpu.github.io，这样的库每个用户名下只能建立一个。

在命令行窗口中执行:

    hexo generate
这个命令会在站点目录下生成public文件夹，其中包含网站的html、css等文件。

其他人大多是使用git上传，我采用的是[github客户端](https://desktop.github.com/)上传，其实都差不多。使用git的hexo可以参考如下教程[Hexo搭建Github静态博客](http://www.cnblogs.com/zhcncn/p/4097881.html)。

因为网络原因，github下载和安装的过程可能很慢，有时还会失败，重试下就好了。

安装并登陆了github客户端后，点击左上角的加号,在弹出的界面中点clone,选择前面创建的库克隆到本地。

然后我们将hexo站点目录下的public文件夹内所有文件复制粘贴到本地的库文件夹中，这时github客户端中就会显示有chenges,在summary框中输入摘要，点击Commit按钮，如果是第一次提交点击左上角的publish按钮，之后的代码提交时publish按钮会变成sync按钮。

然后打开github网页刷新查看，github网站上有时更新很慢，等到库中显示的是最新代码后，在浏览器中打开[zhaochenpu.github.io](http://zhaochenpu.github.io/)（将名字换成你的）查看。

之后每次博客有新的改动需要部署到github时，就在hexo站点根目录下进入命令行，执行：

    hexo g

生成静态文件

或者

    hexo s -g
进行预览加生成静态文件
    
然后将生成的public文件夹内所有文件复制粘贴到库文件夹内覆盖，再用github客户端同步到github。

当然你也可以直接在本地的库文件夹中执行hexo的安装和其他配置，然后将public文件夹内所有文件粘贴到库文件夹根目录下，在github客户端中的Repository settings中添加忽略的文件，只上传需要的文件。


***

谁说程序员不能文艺？欢迎关注微信订阅号《核桃42》。推送的内容关于科技、设计、心理、人文，用好文补脑。
![核桃42](http://upload-images.jianshu.io/upload_images/828721-2b9cde7b3a350c73.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





