# 序篇

笔者对于「想要拥有个人网站」这事儿，在过去几年总会偶尔灵光一现，心血来潮地买下几个月的轻应用服务器以及域名，然后兴高采烈地架设起来，更换各种博客主题，记录一些学习心得；然而，潮起之后马上潮落，有时课程或实习太充实，无暇更新。再者，每到续费的时候，掂量掂量口袋里的钞票，穷大学生最终还是选择了多吃几餐饕餮盛宴。

在这若干次的尝试中，用过阿里云、腾讯云的服务器，国内访问很快，但备案起来很麻烦；买过Amazon，Wix，GoDaddy 以及 Ghost.org 的服务，但是限制较多，需要信用卡，而且墙内速度令人抓狂。而在软件选择上，基本上都是在 [WordPress](https://zh-cn.wordpress.com/) 以及 [Ghost Blog](https://ghost.org) 中周旋。其实这两者对我来说，就像是 facebook 与 ins 对比的感觉。从官方提供的主题来看，后者真的优美许多。WordPress 插件多，但对我个人来说，喜欢且免费的主题太少。因为很少有开发者有那个佛心，无回报的去针对每一个 WordPress widget（小工具）进行兼容，对接和美化。再来，WordPress 着墨了很多在留言、Email、监控，这些都是小型网店才需要的配置，对于一个个人博客来说太重了；所以我更喜欢 Ghost Blog，轻便又优美，改样式也很方便，对于一个轻博客来说，功能足已。

过去想要架设一个 Ghost Blog，最简单的方法就是搞一个 linux 系统的服务器，然后跟着官网的步骤，很快就能上线了。再后来，Ghost.org 官方提供了托管服务，免去了一系列繁琐的架设流程，也省去了域名的配置，但转而换来的是一个每个月至少需要 $29 刀的价格。而就在这两年，Github Pages 的问世，让免费托管成了可能。

[Github Pages](https://pages.github.com/) 是 Github 提供的静态页面服务，可以通过 Jekyll 来进行博客管理。很多 Ghost Blog 的爱好者也关注到了这个平台，有大神在 Github 上提供了一键部署的功能。于是就有了我们今天要介绍的架设实录：如何在 Github Pages 上优雅地架设 Ghost 博客系统。



# 架设篇

> 【写在前面】本文适用于 Linux 或 macOS 系统，Windows 系统的小伙伴需自行参考各类软件的安装方式。需要用到的自动安装的 Github 项目在：[paladini/ghost-on-github-pages](https://github.com/paladini/ghost-on-github-pages)，以下步骤大部分也翻译于此。

## 项目思路

这个项目首先会自动安装 Ghost 博客系统，之后会将动态系统转化为静态页面，并自动部署到 Github Pages 中。值得注意的是，静态页面转化的功能其实用的是早在 2014 就不再更新的 [Buster.py](https://github.com/axitkhurana/buster)。这两个项目都多多少少存在一些 bug，所以如果你成功走完「架设篇」，请一定要看完[「修正篇」](#j)。

## 步骤

1. [环境配置](#i1)
2. [软件安装](#i2)
3. [更新博客](#i3)

<h2 id="i1">环境配置</h2>

**安装 [Python 2](https://www.python.org/download/releases/2.7.2/)**

该软件采用的是 Python 2.x 的环境，因为之后会自动调用的 buster 不支持 3.0+。

```java
# Debian-based systems
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get update
sudo apt-get install python2.7
```

**安装 [NodeJS](https://docs.ghost.org/docs/supported-node-versions)** 

安装可被 Ghost Blog 支持的 NodeJS 版本，目前官方支持的版本分别为 **8.x**、**10.x（官方推荐）** 与 **12.x**。

```java
# Debian-based systems
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs
```

**安装 [npm](https://nodejs.org/en/)**

```java
# Debian-based systems
sudo apt install npm
```

**安装 [wget](https://brew.sh/index_zh-cn.html) (MacOS)**

由于之后自动生成静态文件的过程需要使用 wget 命令，但 MacOS 并没有自带这个功能，需要通过 [homebrew](https://brew.sh/index_zh-cn.html) 进行安装。

```java
# 安装 homebrew
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

# 安装 wget
brew install wget
```

**开启「隐藏文件可见」(MacOS)**

该软件会将 Ghost 博客的文件安装到 MacOS 的 `～` 目录中，即 `/Users/用户名`中的一个`.ghost`的隐藏目录。若不开启「隐藏文件可见」，将无法修改这个目录的读写权限。

```typescript
defaults write com.apple.Finder AppleShowAllFiles YES
```

<h2 id="i2">软件安装</h2>

### 下载 ghost-on-github-pages

可以从项目地址 [paladini/ghost-on-github-pages](https://github.com/paladini/ghost-on-github-pages) 中寻找，或从[这里](https://github.com/paladini/ghost-on-github-pages/archive/master.zip)下载最新版本的软件，然后解压到任意位置。打开文件后，需要关注的只有一个 `install.sh` 文件。

### 创建一个新的 Github repository

首先，你需要开启 Github pages 的使用，方法可以参考[Github pages官方网站](https://pages.github.com/)。如果你要直接使用 `https://用户名.github.io` 作为这个博客的地址的话，则无需在 `用户名.github.io` 的基础上额外创建仓库。不过，我个人推荐还是在「pages项目」的基础上，额外创建一个新的项目管理博客系统，这样更方便管理，原本的页面也不会被吞掉。

在创建好 repository 之后，你需要记下来该项目 git 的 HTTPS 或 SSH 地址。这里推荐使用 HTTPS，因为这免去了系统一些密钥的配置。大致来说，新项目的 HTTPS 地址格式如下：`https://github.com/用户名/新项目名称.git`。这个地址很重要，因为运行安装之后，需要填写。

### 运行 install.sh 进行安装

在刚刚解压的文件夹打开 terminal，首先需要给予这个文件读写的权限（当然，也可以在 Folder 中右键修改）：

```java
chmod +x install.sh
sudo sh ./install.sh
```

而后，软件就会自动开始安装 Ghost 博客到 mac 的 `～` 目录（ `/Users/用户名`）中的一个`.ghost`隐藏文件夹中。或者 Linux 系统下的 `/home/用户名/.ghost`文件中。

在安装的过程中，如果因为系统或网络问题失败或卡住过久。则耐心查看错误原因，并删除`.ghost`文件夹，重新进行权限给予以及安装的过程。若成功安装 Ghost 博客之后，系统会让你回答三个问题：

```java
1. 你的github用户名：
2. 你要安装到的repository的git所需的HTTPS或SSH地址(即上文中记录的内容)
3. 你要安装到的repository的名称
```

填写并安装成功后，你的github项目就会自动提交上传，直接访问`https://用户名.github.io/项目名` 就可以看见成品了。（Github Pages 在上传后不会立即更新，一般需要 2～5 分钟不等，若过长时间没有更新，请检查浏览器缓存是否需要清除或检查Github绑定邮件，失败的话将会发送邮件告知原因。｜另外，这个项目太久没有维护，Github 将会提醒 6 个 warnings，选择“无视”按钮，这些警报就不会再提醒了。）

<h2 id="i3">更新博客</h2>

### Ghost 的简单操作 

在项目安装完成之后，Ghost Blog 会自动在系统中运行，并且开启了端口 `localhost:2368`。但实际上，该项目的配置文件使得它实际运行于 `localhost:2373`。可以在`.ghost`文件夹中，通过 `ghost ls` 命令就可以查看实际运行的端口了。在关闭电脑之后，ghost系统也会随之关闭，需要重新打开，以下是一些 ghost 的常用命令：

```java
// 如果因为权限问题，被deny：一可更改文件读写权限，二可在所有命令前加 sudo
// 开启
ghost start
// 重启
ghost restart
// 关闭
ghost stop
// 查看端口情况
ghost ls
```

在 ghost 服务开启之后，进入`localhost:2373`就可以查看当前的动态页面，而`localhost:2373/ghost`就可以进入服务器，进行文章等博客内容的管理了，首次登录时需要配置一些账号等基本信息，之后就可以直接使用了。

### deploy.sh

在更新之后，只需要在 .ghost 文件夹中运行 deploy.sh 就可以直接部署，十分方便：

```java
cd ~/.ghost
./deploy.sh
```

这个脚本首先会将所有动态内容通过Buster转化成为静态文件，然后通过git，自动部署到Github Pages。 同样，这需要数分钟，在 Github Pages 上才看得到最新的内容。



<h1 id="j">修正篇</h1>

该项目的持有者以及Buster都很久没有更新这两个项目了，项目多多少少存在一些问题。所以我反复检查了源码以及网上的资料，针对两个重要 bug 进行了一下的改进：

- 修改默认端口号，否则链接与头像图片会出错

  - 打开 .ghost 文件夹下的 `config.development.json`文件，将第一行端口改为：`"http://localhost:2373"`，只要与 port 相同即可。

- 修改`deploy.sh`文件，增加一段脚本，不然博文图片将无法显示

  - 修改`deploy.sh`，添加以下8行代码（注意从 CORRECTING BUSTER ERRORS 那行开始）到指定位置：

    ```java
    ...
      
    update() {
    	if [ -d "$GHOST_PATH" ]; then
    		cd "$GHOST_PATH"
    
    		# Generating static files
    		buster generate --domain="$GHOST_SERVER_URL"
    		
    		echo ' -------------- CORRECTING BUSTER ERRORS  -------------- '
    		sudo chmod -R 777 *
    		find . -type f -name '*.html' | xargs sed -i -e 's#.jpgg#.jpg#g'
    		find . -type f -name '*.html' | xargs sed -i -e 's#.jpgpg#.jpg#g'
    		find . -type f -name '*.html' | xargs sed -i -e 's#.jpgjpg#.jpg#g'
    		find . -type f -name '*.html' | xargs sed -i -e 's#.pngg#.png#g'
    		find . -type f -name '*.html' | xargs sed -i -e 's#.pngng#. png#g'
    		find . -type f -name '*.html' | xargs sed -i -e 's#.pngpng#. png#g'
    
    		echo ' -------------------- FIXING LINKS  -------------------- '
    		echo ''
    		read -p "Github username: "  gh_username
    		echo "Leave blank if repo name is username.github.io"
    		read -p "Repo name: " gh_repo
        
       	...
    ```

    他这出现一个致命的bug，图片的后缀莫名地变成了pngg，pngng，pngpng。这里遍历了所有代码，找到出错的地方，进行了修改。



> 结语：本文主要介绍了一个自动部署 Ghost 到 Github Pages 的方法。在下一篇文章中，我们将重点介绍，如何美化 Ghost 博客。

>参考：
>
>1. https://github.com/paladini/ghost-on-github-pages
>2. https://github.com/axitkhurana/buster
>3. https://github.com/axitkhurana/buster/issues/94

