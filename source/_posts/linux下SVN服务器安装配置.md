title: "linux环境SVN服务器安装配置"
date: 2015-3-14 16:58:02
tags: [代码同步,svn]
categories: SVN
---


ubuntu下SVN服务器安装配置
	•	一、SVN安装
1.安装包
```
$ sudo apt-get install subversion
```
2.添加svn管理用户及subversion组
```
$ sudo adduser svnuser zhangyang226
$ sudo addgroup subversion
$ sudo addgroup svnuser subversion 
```
<!--more-->

3.创建项目目录
```
$ sudo mkdir /home/svn
$ cd /home/svn
$ sudo mkdir robots
$ sudo chown -R root:subversion robots
$ sudo chmod -R g+rws robots
```
4.创建SVN文件仓库
```
$ sudo svnadmin create /home/svn/robots
```
5.访问方式及项目导入：
```
$ svn co file:///home/svn/robots
或者
$ svn co file://localhost/home/svn/robots
```
注意：
> 如果您并不确定主机的名称，您必须使用三个斜杠(///)，而如果您指定了主机的名称，则您必须使用两个斜杠(//).
> //--
> 下面的命令用于将项目导入到SVN 文件仓库：
> $ svn import -m "New import" /home/svn/robots file:///home/svnuser/src/robots
> 一定要注明导入信息
> //--------------------------//
6.访问权限设置
```
修改 /home/svn/robots目录下：
svnserve.conf 、passwd 、authz三个文件,行最前端不允许有空格
//--
编辑svnserve.conf文件,把如下两行取消注释
password-db = password
authz-db = authz

//补充说明
# [general]
anon-access = read
auth-access = write
password-db = passwd
其中 anon-access 和 auth-access 分别为匿名和有权限用户的权限，默认给匿名用户只读的权限,但如果想拒绝匿

名用户的访问，只需把 read 改成 none 就能达到目的。

//--
编辑/home/svnuser/etc/passwd 如下:
[users]
mirze = 123456
test1 = 123456
test2 = 123456
//--
编辑/home/svnuser/etc/authz如下
[groups]
admin = mirze,test1
test = test2
[/]
@admin=rw
*=r
这里设置了三个用户mirze,test1,test2密码都是123456
其中mirze和test1属于admin组，有读和写的权限,test2属于test组只有读的权限
```
7.启动SVN服务
```
svnserve -d -r /home/svn
描述说明：
-d 表示svnserver以“守护”进程模式运行
-r 指定文件系统的根位置（版本库的根目录），这样客户端不用输入全路径，就可以访问版本库
如: svn://192.168.12.118/robots

这时SVN安装就完成了.
局域网访问方式：
例如：svn checkout svn://192.168.12.118/robots --username mirze --password 123456 /var/www/robots
```

	•	二、HTTP:// [apache]
1.安装包 [已安装subversion]
```
$ sudo apt-get install libapache2-svn
```
创建版本仓库：
```
sudo svnadmin create /目录地址
```
```
目录地址必须存在，这个就是保存版本仓库的地方，不同的版本仓库创建不同的文件夹即可，比如：
sudo svnadmin create /home/svn/project
本来/home/svn/project这个目录下什么都没有，执行下面的命令之后再去看一下，多出一些文件和文件夹，我们需要操作的是conf这个文件夹，这个文件夹下有一个文件，叫做passwd，用来存放用户名和密码。
然后把这个版本仓库目录授权给apache读写：
sudo chown -R www-data:www-data /目录地址
然后来到打开apache配置文件：
sudo gedit /etc/apache2/mods-available/dav_svn.conf
```
加入如下内容：
```
<Location /project>
DAV svn
SVNPath /home/svn/project
AuthType Basic
AuthName “myproject subversion repository”
AuthUserFile /home/svn/project/conf/passwd
#<LimitExcept GET PROPFIND OPTIONS REPORT>
Require valid-user
#</LimitExcept>
</Location>
```
location说的是访问地址，比如上述地址，访问的时候就是
```
http://127.0.0.1/project
其中有两行被注释掉了，以保证每次都需要用户名密码。
最后一步就是创建访问用户了，建议将用户名密码文件存放在当前版本仓库下conf文件夹下，这样版本仓库多的时候无至于太乱。
因为conf文件夹下已经存在passwd文件了，所以直接添加用户：
sudo htpasswd -c /home/svn/project/conf/passwd test
然后输入两遍密码，laoyang这个用户就创建好了。
打开/home/svn/project/conf/passwd这个文件，会开到形如如下形式的文本：
test:WEd.83H.gealA //后面是加密后的密码。
创建以后，再次需要往别的版本仓库添加这个用户，直接把这一行复制过去就可以了。
重启apache就可以了。
sudo /etc/init.d/apache2 restart
```


	•	三、同步更新 [勾子]
同步程序思路：用户提交程序到SVN，SVN触发hooks,按不同的hooks进行处理，这里用到的是post-commit，利用post-commit到代码检出到SVN服务器的本地硬盘目录，再通过rsync同步到远程的WEB服务器上。

知识点：
1、SVN的hooks
```
# start-commit 提交前触发事务
# pre-commit 提交完成前触发事务
# post-commit 提交完成时触发事务
# pre-revprop-change 版本属性修改前触发事务
# post-revprop-change 版本属性修改后触发事务
通过上面这些名称编写的脚本就就可以实现多种功能了，相当强大。
```
2、同步命令rsync的具体参数使用
3、具有基个语言的编程能力bash python perl都可以实现

```
post-commit具体实现细节
post-commit脚本

编辑文件：sudo vim /home/svn/robots/hooks/post-commit

注意：编辑完成post-commit后，执行：sudo chmod 755 post-commit

内容：

#!/bin/sh
export LANG=zh_CN.UTF-8
sudo /usr/bin/svn update /var/www/www --username mirze --password 123456

或

#Set variable
SVN=/usr/bin/svn
WEB=/home/test_nokia/
RSYNC=/usr/bin/rsync
LOG=/tmp/rsync_test_nokia.log
WEBIP="192.168.0.23"
export LANG=en_US.UTF-8

#update the code from the SVN
$SVN update $WEB --username user --password password
#If the previous command completed successfully, to continue the following
if [ $? == 0 ]
then
 echo "" >> $LOG
 echo `date` >> $LOG
 echo "##############################" >> $LOG
 chown -R nobody:nobody /home/test_nokia/
 #Synchronization code from the SVN server to the WEB server, notes:by the key
 $RSYNC -vaztpH --timeout=90 --exclude-from=/home/svn/exclude.list $WEB root@$WEBIP:/www/ >> $LOG
fi

```
以上是具体的post-commit程序
注意事项：
```
1、一定要定义变量，主要是用过的命令的路径。因为SVN的考虑的安全问题，没有调用系统变量，如果手动执行是没有问题，但SVN自动执行就会无法执行了。
2、SVN update 之前一定要先手动checkout一份出来，还有这里一定要添加用户和密码如果只是手动一样会更新，但自动一样的不行。
3、加上了对前一个命令的判断，如果update的时候出了问题，程序没有退出的话还会继续同步代码到WEB服务器上，这样会造成代码有问题
4、记得要设置所属用户，因为rsync可以同步文件属性，而且我们的WEB服务器一般都不是root用户，用户不正确会造成WEB程序无法正常工作。
5、建议最好记录日志，出错的时候可以很快的排错
6、最后最关键的数据同步，rsync的相关参数一定要清楚，这个就不说了。注意几个场景：
这里的环境是SVN服务器与WEB服务器是开的
把SVN服务器定义为源服务器 WEB服务器为目的服务器
场景一、如果目的WEB服务器为综合的混杂的，像只有一个WEB静态资源，用户提交的，自动生成的都在WEB的一个目录下，建议不要用–delete这个参数
上面这个程序就是这样，实现的是源服务器到目的服务器的更新和添加，而没有删除操作，WEB服务器的内容会多于源SVN的服务器的
场景二、实现镜像，即目的WEB服务器与源SVN服务器一样的数据，SVN上任何变化WEB上一样的变化，就需要–delete参数
场景三、不需要同步某些子目录，可能有些目录是缓存的临时垃圾目录，或者是专用的图片目录（而不是样式或者排版的）要用exclude这个参数
注意：这个参数的使用不用写绝对路径，只要目录名称就行 aa代表文件 aa/ 代表目录 ，缺点就是如果有多个子目录都是一样的名称那么这些名称就都不会被同步
建议用–exclude-from=/home/svn/exclude.list 用文件的形式可以方便的添加和删除
exclude.list

.svn/
.DS_Store
images/

利用SVN的钩子还可以写出很多的程序来控制SVN 如代码提交前查看是否有写日志，是否有tab，有将换成空格，是否有不允许上传的文件，是否有超过限制大小的文件等等。
```

 






 

 



