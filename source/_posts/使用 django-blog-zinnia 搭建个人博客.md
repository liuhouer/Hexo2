
title: "使用 django-blog-zinnia 搭建个人博客"
date:  2016-09-09   23:46:49
tags: [blog]
categories: 博客
---

> 目前网上搭建个人博客的方案很多，虽然使用诸如 Wordpress ( PHP )、Hexo ( Node.js )
> 等可以方便快速地搭建一款功能齐全的高性能个人博客，但是本文将尝试一种更为小众化的方案 —— 一款基于 django-blog-zinnia
> ( Python ) 的个人博客应用。 django-blog-zinnia
> 虽然小巧，但是具备了个人博客应用的全部基础功能，且具有很高的拓展性，并且开箱即用。以下是官方列出的一些特性：

 - 评论

 - 站点地图（用于搜索引擎优化）
 - 文章归档视图（自动按时间归档博文，包括年、月、星期、日各个时间维度）
 - RSS 或者 Atom Feed
 - 分类和标签云
 - 全文搜索
 - Markdown 语法标记
 - 等等其他一些博客应用具备的全部基本功能。
 - 你可以参照它的官方文档 ( django-blog-zinnia documentation ) 的 installation 部分进行初始的安装，但本文也会给出详细的安装教程，并对相关的细节进行进一步补充，对功能进行进一步地拓展设置。

**注：本博客在写作时每一个步骤均在实际环境下测试了一遍，基本确保没有问题。但是由于个人写作时的疏忽或者计算机环境的差异，也可能会有一些错误导致你卡在某个地方无法继续进行下去。如果是这样请给我留言，我和你一起排查问题，如果发现是博客写作时的错误也好使我尽快更正。**

## 建立虚拟环境
因为在安装 django-blog-zinnia 的过程中会安装很多其他第三方依赖包，因此强烈建议使用虚拟环境安装，以免把系统环境弄乱。

假设你的 python 版本是 3.4 或更高（建议使用 3.4 以上版本，当然 django-blog-zinnia 本身是兼容 python2.7 及以上版本的），且已经安装了虚拟环境管理工具 virtualenv，如果没有的话通过`pip install virtualenv` 安装。打开命令行，进入到你想建立虚拟环境的目录，通过命令 `virtualenv zinnia_demo_env` 创建一个名为 zinnia_demo_env 的虚拟环境，当然 zinnia_demo_env 这个目录名你可以任意指定。进入到创建的虚拟环境的 Scripts 目录下，输入 activate 命令激活虚拟环境，此时命令行前多了一个 ( zinnia_demo_env ) 说明已经激活，例如我的是：

``` tex
(zinnia_demo_env) D:\Users\zmrenwu\Envs\zinnia_demo_env\Scripts>
```


通过 pip install django==1.9.6 安装 django，建议使用 1.9.6 版本，当然 >=1.9 的版本都是兼容的，但注意目前不兼容 django1.10。


<!--more-->


## 建立 django 工程
进入你喜欢的目录（确保依然在虚拟环境中，如果没有则按照上面的方法重新开启，且下边的操作默认都在虚拟环境中运行，因此不要退出），通过命令`python django-admin.py startproject zinnia_demo` 创建一个 django 工程。这里 zinnia_demo 是项目名，可以取任何你喜欢的名字。此时你会发现多了一个名为 zinnia_demo 的目录，这样 django 工程就建立好了。进入到这个目录，会看到有一个 manage.py 文件，尝试运行命令 `python manage.py runserver`，不报错的话，在浏览器输入 127.0.0.1:8000，看到如下字样说明 django 工程已经可以正确运行。

> It worked! Congratulations on your first Django-powered page. Of
> course, you haven’t actually done any work yet. Next, start your first
> app by running python manage.py startapp app_label.
> 
> You’re seeing this message because you have DEBUG = True in your
> Django settings file and you haven’t configured any URLs. Get to work!
> 按 Ctrl + c 退出服务器。

## 安装 zinnia 及其依赖
在虚拟环境中输入 `pip install django-blog-zinnia` 安装 django-blog-zinnia，必要的依赖其会自动帮我们安装，但是一些拓展依赖需要我们手动安装，包括：

`pip install markdown`安装 *markdown*，以便使博客文章支持 markdown 格式的文本。

`pip install pygments` 安装 *pygments*，以便支持代码语法高亮。

## 设置 settings.py 文件
进入到 zinnia_demo/zinnia_demo （当然你可能设置了其他项目名，但我相信你能找到），打开 settings.py 文件（用文本编辑器或者 python IDE 打开，不要直接运行），在 INSTALL_APPS 列表里添加以下的 APP，这些 APP 都是 django-blog-zinnia 依赖运行的 APP ：

``` sml
zinnia_demo/zinnia_demo/settings.py

INSTALLED_APPS = [
工程建立时默认添加的app
'django.contrib.admin',
'django.contrib.auth',
'django.contrib.contenttypes',
'django.contrib.sessions',
'django.contrib.messages',
'django.contrib.staticfiles',

项目添加的app
'django.contrib.sites',
'django_comments',
'mptt',
'tagging',
'zinnia',
]
```


在 TEMPLATES 列表的如下位置加入 `zinnia.context_processors.version` ，当然这一步是可选的，其作用只是在博客页面的底部显示一个django-blog-zinnia 的版本号：

``` nimrod
zinnia_demo/zinnia_demo/settings.py

TEMPLATES = [
{
'BACKEND': 'django.template.backends.django.DjangoTemplates',
'DIRS': [],
'APP_DIRS': True,
'OPTIONS': {
'context_processors': [
'django.template.context_processors.debug',
'django.template.context_processors.request',
'django.contrib.auth.context_processors.auth',
'django.contrib.messages.context_processors.messages',

添加这句
'zinnia.context_processors.version',  # Optional
],
},
},
]
```


在 ALLOWED_HOSTS = 的下面添加 SITE_ID = 1

``` protobuf
zinnia_demo/zinnia_demo/settings.py

ALLOWED_HOSTS = []
SITE_ID = 1
```


并修改语言和时区，获得更友善的语言和时间显示，注意 + 号表示添加的行，- 号表示删去的行：

``` sml
zinnia_demo/zinnia_demo/settings.py

LANGUAGE_CODE = 'en-us'
LANGUAGE_CODE = 'zh-hans'

TIME_ZONE = 'UTC'
TIME_ZONE = 'Asia/Shanghai'
```


## 设置 urls.py 文件
打开相同目录下的 urls.py 文件，做如下修改，注意 + 号表示添加的行，- 号表示删去的行：

``` python
zinnia_demo/zinnia_demo/urls.py

from django.conf.urls import url
from django.conf.urls import url,include
from django.contrib import admin

urlpatterns = [
url(r'^admin/', admin.site.urls),
url(r'^weblog/', include('zinnia.urls')),
    url(r'^comments/', include('django_comments.urls')),
]
```


## 同步数据库并创建后台管理员账户
在 manage.py 文件所在目录下输入 `python manage.py migrate` 建立相应的数据库表结构。此时会看到目录下多了一个 db.sqlite 文件，这是存储博客数据的数据库文件，默认使用的 sqlite3。输入命令 `python manage.py createsuperuser` 创建后台管理员账户，命令行会提示你输入用户名、邮箱、密码。注意密码输入时不会有任何显示，只管输下去就行。

## 开启开发服务器
再次运行`python manage.py runserver` 开启开发服务器，在浏览器输入 `127.0.0.1:8000/weblog` 将看到博客的首页面。输入 *127.0.0.1:8000/admin* 会进入后台登录页面，输入刚才创建的管理员账户用户名和密码就可以登录到后台管理界面。在日志后面点击增加按钮尝试着添加一篇博客看看！再次进入 *127.0.0.1:8000/weblog* 就可以看到刚才发表的博客了。



![](http://o8a5h1k2v.bkt.clouddn.com/16-9-8/33882098.jpg)

至此基本的博客已经搭建完毕，接下来是一些可选功能拓展，包括 markdown 语法支持，代码高亮，bootstrap 主题的安装。

## ( OPTIONAL ) Markdown 语法支持
再次打开 settings.py 文件，在文件的最后添加：

``` sml
zinnia_demo/zinnia_demo/settings.py

...
ZINNIA_MARKUP_LANGUAGE = 'markdown'
ZINNIA_MARKDOWN_EXTENSIONS = ['markdown.extensions.extra', 'markdown.extensions.codehilite']
```


bingo！！

ZINNIA_MARKUP_LANGUAGE 指明了使用 markdown 语法标记，ZINNIA_MARKDOWN_EXTENSIONS 做了两个拓展，其中 markdown.extensions.codehilite 表示支持语法高亮，markdown.extensions.extra 包含的特性请参见 markdown 相关文档。

## ( OPTIONAL ) 安装 Bootstrap 主题
如果你不喜欢原生的主题的话，django-blog-zinnia 为我们提供了一套 bootstrap 主题，相对来说更加好看一点。虽然说实在话内置的主题感觉都已经过时了，因此我重新为它设置了一套全新的主题，稍后会有介绍。

中断服务器的运行，进入到虚拟环境（如果你已经退出了的话），首先输入命令`pip install django-app-namespace-template-loader`安装 *django-app-namespace-template-loader*，这是替换主题的一个必要组件。再输入 `pip install zinnia-theme-bootstrap` 安装*主题 APP* ，打开 *settings.py* 文件，在 *INSTALLED_APPS* 中注册主题 APP ，注意主题 zinnia_bootstrap 一定要在 zinnia 之前：

``` stylus
zinnia_demo/zinnia_demo/settings.py

INSTALLED_APPS = [
...
'zinnia_bootstrap',
  'zinnia',
]
```


再将 TEMPLATES 列表做如下修改：

``` nimrod
zinnia_demo/zinnia_demo/settings.py

TEMPLATES = [
{
'BACKEND': 'django.template.backends.django.DjangoTemplates',
'DIRS': [],
'APP_DIRS': True,
    'APP_DIRS': False,
        'OPTIONS': {
        'context_processors': [
        'django.template.context_processors.debug',
        'django.template.context_processors.request',
        'django.contrib.auth.context_processors.auth',
        'django.contrib.messages.context_processors.messages',
        'zinnia.context_processors.version',  # Optional
        ],

        添加下面这几行
        'loaders': [
           'app_namespace.Loader',
           'django.template.loaders.filesystem.Loader',
           'django.template.loaders.app_directories.Loader',
        ],
        添加上面几行
        },
    },
]
```


此时再次开启服务器，进入主页 `127.0.0.1:8000/weblog` 就可以看到主题变成了 bootstrap 样式了。

## ( OPTIONAL ) 语法高亮支持
注意：这一步必须在安装完 bootstrap 主题之后。

pygments 已经帮我做好了一切语法高亮的准备，其原理就是把 html 中的代码文本分成很多块，用适当的 html 标签包裹，并且添加相应的 css 类，我们只需引入一个相应的 css 样式文件即可。

为了方便起见，我们新建一个 APP 来存放我的需要引入的 css 样式文件，在 zinnia_demo/ 目录下（与 manage.py 同级）下输入 `python manage.py startapp theme`，这样我们就创建了一个名为 theme 的 app，可以看到 zinnia_demo/ 多了一个 theme 的文件夹。

在 zinnia_demo/ 目录下（与 manage.py 同级）建立如下的目录结构和文件：

zinnia_demo/templates/zinnia/skeleton.html，把这里面的内容：skeleton 模板代码 ，复制到 skeleton.html中，并且在 skeleton.html 的 \ 标签里添加一行：

``` stylus
zinnia_demo/templates/zinnia/skeleton.html

<link rel="stylesheet" href="{% static "theme/css/github.css" %}" />
```


再在 zinnia_demo/ 下建立如下的目录结构和文件：

zinnia_demo/theme/static/theme/css/github.css，把这里面的内容：=[github.css][1] 样式代码 ，复制到 github.css 文件中。

打开 settings.py 文件，做如下修改：

``` gradle
zinnia_demo/zinnia_demo/settings.py

TEMPLATES = [
{
...
修改成下面的样子，作用是指明模板文件所在目录，即上面我们写的skeleton.html
'DIRS': [os.path.join(BASE_DIR, 'templates')],
...
}
]
```


将 theme app 注册到 INSTALLED_APPS 列表中：

``` stylus
zinnia_demo/zinnia_demo/settings.py

INSTALLED_APPS = [
...
'theme',
'zinnia_bootstrap',
'zinnia',
]
```


打开开发服务器，进入相应页面就可以看到代码高亮效果了。记得事先填充一些代码到博客文章中。

## 结束
PS：

自带的评论功能当有人回复你发表的博客文章后会发送一封 email 给你的后台管理员账户邮箱（创建后台管理员账户填写的）。不过需要设置好发送邮件的邮箱，参考配置如下，在 settings.py 中：

``` nix
zinnia_demo/zinnia_demo/settings.py

EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.163.com' # 如果不是163邮箱请替换为邮箱服务商的smtp服务器地址
EMAIL_PORT = 465
EMAIL_HOST_USER = '你的邮箱账号'  # add your own accounts for local test
EMAIL_HOST_PASSWORD = '你的邮箱密码'
EMAIL_USE_SSL = True
DEFAULT_FROM_EMAIL = EMAIL_HOST_USER
```


不过要确保你的邮箱开启了 SMTP，如果没有的话请参考邮箱服务商的相关设置进行开启。


  [1]: https://github.com/zmrenwu/ZinniaBlog/blob/master/zinnia_twts/static/zinnia_twts/theme/css/github.css