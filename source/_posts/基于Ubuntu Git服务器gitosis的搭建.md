title: "基于Ubuntu Git服务器（gitosis）的搭建"
date: 2015-3-13 16:58:02
tags: [git服务器,代码同步,git]
categories: git
---

 首先从整体上介绍git服务器的工作原理：多个客户端，其中可以包括仓库管理员，通过将自己的ssh公钥上传到服务器仓库keydir目录，统一调用Git专用账号git进行访问git仓库，不同的用户可以根据不同的ssh公钥校验登录，进行项目版本的各种操作，包括clone，commit，push，pull等等。

    下面具体介绍Git服务器的搭建，机器环境：ubuntu12.04，Git服务器软件采用gitosis(https://github.com/res0nat0r/gitosis )。

<!--more-->


一、软件安装
    
    1.1 安装ssh的服务端和客户端：sudo apt-get install openssh-server openssh-client
    1.2 安装git-core软件，这个是git服务的基础：sudo apt-get install git-core
    1.3 安装 Gitosis，这个是git服务器软件
``` bash
     sudo apt-get install python-setuptools
```
``` bash
		   cd /tmp
```
``` bash
				 git clone https://github.com/res0nat0r/gitosis.git
```
``` bash
					cd gitosis
```
``` bash
					sudo python setup.py install
```
   

    ok，软件的安装都此结束。

二、创建Git专用账号git
    sudo useradd -m -s /bin/bash git    //创建git账号，用户家目录默认为/home/git，shell为/bin/bash
    sudo passwd git    //设置git用户的密码

三、初始化Git仓库
    1、初始化Git仓库需要一个管理员账号，如上图所示，管理员也是一个客户端用户，所以需要在客户端主机（我的客户端机器是Win7系统）生成一个用户，并且生成ssh-key。具体操作如下：
        
        在客户端安装ssh服务，包括客户端和服务端。如果你的客户端系统是：
            （1）Windows机器，建议直接安装git bash软件，其中包括了ssh这个服务；
            （2）Linux系统，可以按照先前的命令apt-get install openssh-server openssh-client安装ssh服务。
        
        在客户端机器我的账号是huixiao200068，在用户家目录中生成ssh-key：
            （1）Windows机器，打开git bash软件，输入命令
                        cd ~
                        ssh-keygen -t rsa
            （2）Linux系统，也执行以上相同的命令，即可以生成当前用户的ssh-key。
            执行完成之后，输入命令：ls -al，如果出现了.ssh目录，则表示ssh-key成功创建。

    2、把ssh的公钥上传到服务器，我这里假设上传到服务器的临时目录/tmp。
        scp  ~/.ssh/id_rsa.pub git@server:/tmp    //命令中的server改成你自己服务器的IP
    
    到此为止，仅仅只是做好了初始化仓库的准备工作，上面的2步操作都是在客户端操作的。下面是关键的第三步——初始化，该步骤在服务器端执行。

    3、初始化
        sudo -H -u git gitosis-init < /tmp/id_rsa.pub    //将git仓库目录初始化到了git用户家目录下
        此时，git用户的home目录中将出现repositories目录，该目录为git的仓库。
        
        修改目录权限：sudo chmod 755 /home/git/repositories/gitosis-admin.git/hooks/post-update，该步骤尚不清楚其具体的作用，修改了这个目录的权限有啥用呢？望各位指教。

恭喜你，到此为止，你已经创建了一个git仓库，而且账号huixiao200068是git仓库的管理员啦。下面要做的就是仓库配置啦，管理项目和用户。

四、下载仓库配置项目gitosis-admin到本地客户端
        因为git仓库的配置文件都是以git方式来管理的，所以你需要先下载一份到客户端本地 。
        在你的用户目录下面创建一个临时目录work，
        然后 进入到该目录：cd work，
        然后执行命令：
        git clone git@server:gitosis-admin.git    // 命令中的server改成你自己服务器的IP 
        执行完成之后，work目录下会生成gitosis-admin目录，目录下面有一个gitosis.conf文件和一个keydir目录，它们将是下面配置任务的主要操作对象，请牢记它们的位置。

五、新建项目
       1、修改配置文件gitosis.conf，增加如下内容。
            [group first-pro]    //用户组名
            members = huixiao200068    //成员名，多个成员可以用空格隔开
            writable = first-pro    //项目名及其用户对于此项目的权限，目前是可写
    
        2、创建项目目录 mkdir first-pro
             初始化该目录 cd first-pro;    git init
             添加远程仓库 git remote add origin git@SERVER:first-pro.git
             创建工程文件 touch 1.txt
             添加工程文件到本地仓库中 git add ./1.txt
             提交整个项目到本地仓库 git commit -m "comment"
             提交整个项目到远程仓库 git push origin master    //只有这样子，团队其他开发人员才能看到你修改的代码
             查看提交日志 git log
             查看本地库当前的状态，比如是否有文件需要提交。 git status

        新的项目仓库已经生成，其他开发人员可以git clone 命令从服务器上下载一份工程到自己的本地机器上，协同开发啦！

六、新建用户
        
        关于上一节最后提到的内容，“其他开发人员”，他们是需要管理员来增加和配置的，这一节主要讲怎么添加用户。
        
（1）客户端操作：
        首先要生成ssh-key，方法和上述说明的一样。
        cd ~
        ssh-keygen -t rsa
        然后一直回车，就OK。然后将生成的id_rsa.pub文件传给GIT服务器管理员

（2）服务器端操作：
        管理员将客户上传的id_rsa.pub文件移到gitosis-admin/keydir目录中，并且改名为CLIENT_NAME.pub。注意：如果客户端如下图所示
        ，则CLIENT_NAME为sean@bogon，否则后续操作会出现“Repository read access denied”的错误。
        
         给项目first-pro增加新的开发者，编辑gitosis.conf文件，vi gitosis.conf。
          [group first-pro]    //用户组名
          members = huixiao200068 sean@bogon    //成员名，多个成员可以用空格隔开
          writable = first-pro    //项目名及其用户对于此项目的权限，目前是可写

         提交修改的管理文件：
            git commit -a -m "add user sean@bogon"
            git push origin master

完成上述2步之后，即可以使用该账号共同开发项目first-pro啦！
        cd ~
        git clone git@SERVER:first-pro.git    //克隆项目到本地
        ……    //do anything you want to do
        commit -am "comment"
        commit push origin master






 

 



