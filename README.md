

![](https://raw.githubusercontent.com/qiubaiying/qiubaiying.github.io/master/img/readme-home.png)

[![Build Status](https://travis-ci.org/qiubaiying/qiubaiying.github.io.svg?branch=master)](https://travis-ci.org/qiubaiying/qiubaiying.github.io)
[![codebeat badge](https://codebeat.co/badges/5f031df3-f6c1-4ec0-911a-ff6617ca50b9)](https://codebeat.co/projects/github-com-qiubaiying-qiubaiying-github-io-master)
[![GitHub issues](https://img.shields.io/github/issues/qiubaiying/qiubaiying.github.io.svg?style=flat)](https://github.com/qiubaiying/qiubaiying.github.io/issues)
[![License MIT](https://img.shields.io/badge/license-MIT-blue.svg?style=flat)](https://github.com/home-assistant/home-assistant-iOS/blob/master/LICENSE)
[![](https://img.shields.io/github/stars/qiubaiying/qiubaiying.github.io.svg?style=social&label=Star)](https://github.com/qiubaiying/qiubaiying.github.io)
[![](https://img.shields.io/github/forks/qiubaiying/qiubaiying.github.io.svg?style=social&label=Fork)](https://github.com/qiubaiying/qiubaiying.github.io)


博客的搭建教程修改自 [Hux](https://github.com/Huxpro/huxpro.github.io) 
 
更为详细的教程戳这 [《利用 GitHub Pages 快速搭建个人博客》](http://www.jianshu.com/p/e68fba58f75c) 或 [wiki](https://github.com/qiubaiying/qiubaiying.github.io/wiki/%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA%E8%AF%A6%E7%BB%86%E6%95%99%E7%A8%8B)

>
### [查看博客戳这里 👆](http://qiubaiying.github.io)



## 使用

* 开始
	* [环境](#环境)
	* [开始](#开始)
	* [撰写博文](#撰写博文)
* 组件
	* [侧边栏](#侧边栏)
	* [迷你关于我](#mini-about-me)
	* [推荐标签](#featured-tags)
	* [好友链接](#friends)
	* [HTML5 演示文档布局](#keynote-layout)
* 评论与 Google/Baidu Analytics
	* [评论](#comment)
	* [网站分析](#analytics) 
* 高级部分
	* [自定义](#customization)
	* [标题底图](#header-image)
	* [搜索展示标题-头文件](#seo-title)



### 环境

如果你安装了 [jekyll](http://jekyllcn.com/)，那你只需要在命令行输入`jekyll serve` 或 `jekyll s`就能在本地浏览器中输入`http://127.0.0.1:4000/`预览主题，对主题的修改也能实时展示（需要强刷浏览器）。



### 开始

你可以通用修改 `_config.yml`文件来轻松的开始搭建自己的博客:

```
# Site settings
title: BY Blog                    # 你的博客网站标题
SEOTitle: 柏荧的博客 | BY Blog		# SEO 标题
description: "Hey"	   	   # 随便说点，描述一下

# SNS settings      
github_username: qiubaiying     # 你的github账号
jianshu_username: e71990ada2fd  # 你的简书ID。

# Build settings
# paginate: 10              # 一页你准备放几篇文章
```

Jekyll官方网站还有很多的参数可以调，比如设置文章的链接形式...网址在这里：[Jekyll - Official Site](http://jekyllrb.com/) 中文版的在这里：[Jekyll中文](http://jekyllcn.com/).

### 撰写博文

要发表的文章一般以 **Markdown** 的格式放在这里`_posts/`，你只要看看这篇模板里的文章你就立刻明白该如何设置。

yaml 头文件长这样:

```
---
layout:     post
title:      定时器 你真的会使用吗？
subtitle:   iOS定时器详解
date:       2016-12-13
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: 	 true
tags:
    - iOS
    - 定时器
---

```

### 侧边栏

看右边:
![](https://raw.githubusercontent.com/qiubaiying/qiubaiying.github.io/master/img/readme-side.png)

设置是在 `_config.yml`文件里面的`Sidebar settings`那块。

```
# Sidebar settings
sidebar: true  #添加侧边栏
sidebar-about-description: "简单的描述一下你自己"
sidebar-avatar: /img/avatar-by.jpg     #你的大头贴，请使用绝对地址.注意：名字区分大小写！后缀名也是
```

侧边栏是响应式布局的，当屏幕尺寸小于992px的时候，侧边栏就会移动到底部。具体请见bootstrap栅格系统 <http://v3.bootcss.com/css/>


### Mini About Me

Mini-About-Me 这个模块将在你的头像下面，展示你所有的社交账号。这个也是响应式布局，当屏幕变小时候，会将其移动到页面底部，只不过会稍微有点小变化，具体请看代码。

### Featured Tags

看到这个网站 [Medium](http://medium.com) 的推荐标签非常的炫酷，所以我将他加了进来。
这个模块现在是独立的，可以呈现在所有页面，包括主页和发表的每一篇文章标题的头上。

```
# Featured Tags
featured-tags: true  
featured-condition-size: 1     # A tag will be featured if the size of it is more than this condition value
```

唯一需要注意的是`featured-condition-size`: 如果一个标签的 SIZE，也就是使用该标签的文章数大于上面设定的条件值，这个标签就会在首页上被推荐。
 
内部有一个条件模板 `{% if tag[1].size > {{site.featured-condition-size}} %}` 是用来做筛选过滤的.

### Social-media Account

在下面输入的社交账号，没有的添加的不会显示在侧边框中。新加入了[简书](https:/www.jianshu.com)链接, <http://www.jianshu.com/u/e71990ada2fd>

	# SNS settings
	RSS: false
	jianshu_username: 	jianshu_id 
	zhihu_username:     username
	facebook_username:  username
	github_username:    username
	# weibo_username:   username
	
	

![](http://ww4.sinaimg.cn/large/006tKfTcgy1fgrgbgf77aj308i02v748.jpg)

### Friends

好友链接部分。这会在全部页面显示。

设置是在 `_config.yml`文件里面的`Friends`那块，自己加吧。

```
# Friends
friends: [
    {
        title: "BY Blog",
        href: "https://qiubaiying.github.io/"
    },
    {
        title: "Apple",
        href: "https://apple.com/"
    }
]
```


### Keynote Layout

HTML5幻灯片的排版：

![](https://camo.githubusercontent.com/f30347a118171820b46befdf77e7b7c53a5641a9/687474703a2f2f6875616e677875616e2e6d652f696d672f626c6f672d6b65796e6f74652e6a7067)

这部分是用于占用html格式的幻灯片的，一般用到的是 Reveal.js, Impress.js, Slides, Prezi 等等.我认为一个现代化的博客怎么能少了放html幻灯的功能呢~

其主要原理是添加一个 `iframe`，在里面加入外部链接。你可以直接写到头文件里面去，详情请见下面的yaml头文件的写法。

```
---
layout:     keynote
iframe:     "http://huangxuan.me/js-module-7day/"
---
```

iframe在不同的设备中，将会自动的调整大小。保留内边距是为了让手机用户可以向下滑动，以及添加更多的内容。


### Comment

博客不仅支持 [Disqus](http://disqus.com) 评论系统,还加入了 [Gitalk](https://gitalk.github.io/) 评论系统，[支持 Markdwon 语法](https://guides.github.com/features/mastering-markdown/)，cool~

#### Disqus

优点：国际比较流行，界面也很大气、简洁，如果有人评论，还能实时通知，直接回复通知的邮件就行了；

缺点：评论必须要去注册一个disqus账号，分享一般只有Facebook和Twitter，另外在墙内加载速度略慢了一点。想要知道长啥样，可以看以前的版本点[这里](http://brucezhaor.github.io/about.html) 最下面就可以看到。

> Node：有很多人反映 Disqus 插件加载不出来，可能墙又架高了，有条件的话翻个墙就好了~

**使用：**

**首先**，你需要去注册一个Disqus帐号。**不要直接使用我的啊！**

**其次**，你只需要在下面的 yaml 头文件中设置一下就可以了。

```
# 评论系统
# Disqus（https://disqus.com/）
disqus_username: qiubaiying
```

#### Gitalk

优点：界面干净简洁，利用 Github issue API 做的评论插件，使用 Github 帐号进行登录和评论，最喜欢的支持 Markdown 语法，对于程序员来说真是太 cool 了。

缺点：配置比较繁琐，每篇文章的评论都需要初始化。

**使用：**

参考我的这篇文章：[《为博客添加 Gitalk 评论插件》](http://qiubaiying.top/2017/12/19/%E4%B8%BA%E5%8D%9A%E5%AE%A2%E6%B7%BB%E5%8A%A0-Gitalk-%E8%AF%84%E8%AE%BA%E6%8F%92%E4%BB%B6/)


### Analytics

网站分析，现在支持百度统计和Google Analytics。需要去官方网站注册一下，然后将返回的code贴在下面：

```
# Baidu Analytics
ba_track_id: 4cc1f2d8f3067386cc5cdb626a202900

# Google Analytics
ga_track_id: 'UA-49627206-1'            # 你用Google账号去注册一个就会给你一个这样的id
ga_domain: huangxuan.me			# 默认的是 auto, 这里我是自定义了的域名，你如果没有自己的域名，需要改成auto。
```

### Customization

如果你喜欢折腾，你可以去自定义这个模板的 Code。

**如果你可以理解 `_include/` 和 `_layouts/`文件夹下的代码（这里是整个界面布局的地方），你就可以使用 Jekyll 使用的模版引擎 [Liquid](https://github.com/Shopify/liquid/wiki)的语法直接修改/添加代码，来进行更有创意的自定义界面啦！**

### Header Image

博客每页的标题底图是可以自己选的，看看几篇示例post你就知道如何设置了。
  
标题底图的选取完全是看个人的审美了。每一篇文章可以有不同的底图，你想放什么就放什么，最后宽度要够，大小不要太大，否则加载慢啊。

> 上传的图片最好先压缩，这里推荐 imageOptim 图片压缩软件，让你的博客起飞。

但是需要注意的是本模板的标题是**白色**的，所以背景色要设置为**灰色**或者**黑色**，总之深色系就对了。当然你还可以自定义修改字体颜色，总之，用github pages就是可以完全的个性定制自己的博客。

### SEO Title

我的博客标题是 **“BY Blog”** 但是我想要在搜索的时候显示 **“柏荧的博客 | BY Blog”** ，这个就需要 SEO Title 来定义了。

其实这个 SEO Title 就是定义了<head><title>标题</title></head>这个里面的东西和多说分享的标题，你可以自行修改的。

### 关于收到"Page Build Warning"的 Email

由于jekyll升级到3.0.x,对原来的 pygments 代码高亮不再支持，现只支持一种-rouge，所以你需要在 `_config.yml`文件中修改`highlighter: rouge`.另外还需要在`_config.yml`文件中加上`gems: [jekyll-paginate]`.

同时,你需要更新你的本地 jekyll 环境.

使用`jekyll server`的同学需要这样：

1. `gem update jekyll` # 更新jekyll
2. `gem update github-pages` #更新依赖的包

使用`bundle exec jekyll server`的同学在更新 jekyll 后，需要输入`bundle update`来更新依赖的包.

> Note：
> 可以使用 `jekyll -s` 命令在本地实时配置博客，提高效率。详见 [Jekyll.com](http://jekyllcn.com/)

参考文档：[using jekyll with pages](https://help.github.com/articles/using-jekyll-with-pages/) & [Upgrading from 2.x to 3.x](http://jekyllrb.com/docs/upgrading/2-to-3/)


## 致谢

1. 这个模板是从这里 [Hux](https://github.com/Huxpro/huxpro.github.io) fork 的, 感谢这个作者。 
2. 感谢 Jekyll、Github Pages 和 Bootstrap!

## License

遵循 MIT 许可证。有关详细,请参阅 [LICENSE](https://github.com/qiubaiying/qiubaiying.github.io/blob/master/LICENSE)。


## 备录内容
怎么在ubuntu server 上搭建git服务器，以腾讯云为例
以下步骤最好使用管理员权限进行，避免某些步骤出现没有权限的问题
安装git的核心 git core
cd ~
1. sudo apt-get install git-core openssh-server openssh-client
上面的命令安装失败的话，需要更新数据源
2. sudo apt-get update 再重新执行第一条命令即可
3. sudo apt-get install python-setuptools 
安装python的 setuptools 和gitosis
4. 配置给git用户信息
git config --global user.name "name"
git config --global user.email "Email"
接着安装gitosis
5. git clone https://github.com/res0nat0r/gitosis.git
进入gitosis里面 cd gitosis
6. sudo python setup.py install
接下来对git进行一些基本的配置
cd ~/gitosis
7. sudo useradd -m git
8. sudo passwd git 设置的密码记住
9. sudo mkdir /home/gitrepository
10. sudo chown git:git /home/gitrepository/
11. sudo chmod 700 /home/gitrepository/
cd /home/git
12. sudo ln -s /home/gitrepository /home/git/repositories
接下来再自己的window10 机器里面生成密钥ssh 》 ssh-keygen -t rsa
使用winscp链接云服务器，一般只需要把实例的用户名和密码填上就好了，但是也会出现链接被拒绝的情况
1. 查看云服务去的22端口有没有打开
2. sudo /etc/init.d/ssh restart
3. sudo vi /etc/ssh/ssh_config或者sudo vi /etc/ssh/sshd_config 把PermitRootLogin no 改成yes
重启sshd服务就可以链接
4. service sshd restart
把刚刚再Windows10 上生成的密钥传到服务器上
接着回到服务器，使用密钥对服务器上的rsa进行初始化
cd /home/git
13. sudo -H -u git gitosis-init < /tmp/xjy.pub
14. sudo chmod 755 /home/gitrepository/gitosis-admin.git/hooks/post-update
使用git账户创建项目仓库和目录
15. su git
16. cd /home/gitrepository
17. mkdir mytask.git
18. cd mytask.git
19. git init --bare
20. exit
回到客户机上，打开git bash
测试可以直接远程操控 shh 用户名@IP地址
21. git clone git@公网IP:/gitosis-admin.git 用户名就是登陆进去实例的时候的用户名
22. 文件夹下包含两个文件，gitisos.conf用于配置权限，keydir用来存放shh公钥文件
[group gitosis-admin]
members = geekli@DESKTOP-01RHDFO
writable = gitosis-admin mytask
[group geeklh]
members = geeklh
writable = mytask
[group readonly]
members = read
readonly = test
members 为用户名 与.pub文件对应 多个用户以空格隔开

writable 可写项目组 ，以空格隔开

readonly 只读项目组，以空格隔开

接着再提交到服务器上：
git add . 
git commit -m "add new"
git push origin master

新增用户不生效？重启SSH服务：sudo /etc/init.d/ssh restart

最后就可以进行clone了  git clone git@公网IP:/mytask.git
已经实验过了，可以正常的check out 和push到master

Ubuntu系统链接出现Someone could be eavesdropping on you right now (man-in-the-middle attack)!
解决：
主要原因是之前电脑有链接过服务器，服务器把登录标识证书记录下来了，下次登录时会对比之前的记录，由于服务器的系统重装表示变了，导致不能继续登录，解决方案就是执行ssh-kengen -R 服务器公网IP，即可解决这个问题，然后再执行ssh 用户名@公网IP，即可链接成功。


## spring boot 内容
# 常用注解
@SpringBootApplication 通常用于启动类，相当于同时加上

@MapperScan 在springBoot中集成mybatis，可以在mapper接口上添加@mapper注解，将mapper注入到spring；使用MapperScan来扫描包
=>@MapperScan("com.demo.mapper")：扫描指定包中的接口

@MapperScan("com.demo.*.mapper")：一个*代表任意字符串，但只代表一级包,比如可以扫到com.demo.aaa.mapper,不能扫到com.demo.aaa.bbb.mapper

@MapperScan("com.demo.**.mapper")：两个*代表任意个包,比如可以扫到com.demo.aaa.mapper,也可以扫到com.demo.aaa.bbb.mapper


 @Service("userService")/@Service(value = "userService")
@Service("userService")注解是告诉Spring，当Spring要创建UserServiceImpl的的实例时，bean的名字必须叫做"userService"，这样当Action需要使用UserServiceImpl的的实例时,就可以由Spring创建好的"userService"，然后注入给Action：在Action只需要声明一个名字叫“userService”的变量来接收由Spring注入的"userService"即可

@Autowired 表示被修饰的类需要注入对象，springihui扫描而所有被Autowired标注的类，然后根据类型在ioc容器中找到匹配的类注入

@Override，编辑器帮助验证@Override下面的方法名是否是父类中所有的，如果没有就报错。
在代码中重载后者重写方法时会用到
重载：方法名一样，但是参数类型或者个数会不一样，返回值可以相同也可以不同。
重写：子类对于父类方法的继承，在此基础上对部分方法进行修改。新方法会直接覆盖旧的

@Controller 处理HTTP请求
直接在项目中直接用Controller，然后再请求，就会发生Whitelabel Error Page
是因为没有使用模板，使用Controller是用来相应页面的，Controller必须配合模板来使用
FreeMarker
Groovy
Thymeleaf （Spring 官网使用这个）
Velocity

注意：@RestController新加入注解，返回的json需要@ResponseBody和@Controller配合

@RequestMapping 配置url映射
RequestMapping 可以作用在控制器上某个方法，也可以作用在此控制器类上。
当控制器在类级别上添加@RequestMapping注解时，这个注解会应用到控制器的所有处理器方法上。处理器方法上的@RequestMapping注解会对类级别上的@RequestMapping的声明进行补充。

1，@Controller表明该类所有的方法返回页面路径，但是在方法上加了@ResponseBody后，该方法返回的是数据。《《======
2，@RestController则相当于@Controller和@ResponseBody同时使用的效果，返回的也是数据，不是界面
3，如果我们还想返回界面可以使用ModelAndView方法

    1、@RequestBody需要把所有请求参数作为json解析，因此，不能包含key=value这样的写法在请求url中，所有的请求参数都是一个json
    2、直接通过浏览器输入url时，@RequestBody获取不到json对象，需要用java编程或者基于ajax的方法请求，将Content-Type设置为application/json

##理解什么是spring boot的bean
#什么是bean？
1、Java面向对象，对象有方法和属性，就需要对象实例来调用方法和实例（实例化）
2、凡是有方法和属性的类都需要实例化，这样才能具象化去使用这些方法和属性
3、把bean理解成类的代理（通过反射、代理来实现），这样就能代表类该拥有的东西

1、一类是使用Bean，即是把已经在xml文件中配置好的Bean拿来用，完成属性、方法的组装；比如@Autowired , @Resource，可以通过byTYPE（@Autowired）、byNAME（@Resource）的方式获取Bean；

2、一类是注册Bean,@Component , @Repository , @ Controller , @Service , @Configration这些注解都是把你要实例化的对象转化成一个Bean，放在IoC容器中，等你要用的时候，它会和上面的@Autowired , @Resource配合到一起，把对象、属性、方法完美组装。
源码：
<pre class="code">

     @Bean

     public MyBean myBean() {

         // instantiate and configure MyBean obj

         return obj;

    }</pre>

@Bean明确地指示了一种方法，产生一个bean方法，并且交给sping容器管理。被注释的方法，你给我产生一个bean，然后交给spring容器，剩下的你就别管了。

对bean的总结：
1、凡是子类以及带属性、方法的类被注册bean到spring中，交给spring容器来管理；
2、@Bean用在方法上，告诉spring容器，可以从下面这个方法中拿到一个bean


## spring boot记录

