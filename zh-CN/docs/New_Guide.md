---
order: 1
---

# 新手指南
## 概要
本文假设用户已经注册拥有了NiceScale帐号，通过NiceScale部署一个PHP+MySQL集群服务，演示了整个产品的使用过程。

### 第一步 添加云供应商
NiceScale本身并不提供IaaS资源，如果你还没有任何IaaS的帐号，那么你需要先去对应的云供应商进行注册，注册完后，再进入NiceScale。

目前NiceScale支持的云供应商列表：

- 青云QingCloud
- AWS

拥有了服务商的帐号后，点击NiceScale控制台的右上"添加供应商"按钮，将API Key加入到NiceScale中。

### 第二步 部署PHP+MySQL
进入NiceScale的控制台，点击左侧导航的“软件市场”，可以看到市场中有很多服务。先选择"Apache_PHP"服务，开始部署之旅。

![PHP在NiceScale中的部署截图](/assets/php-deploy.png "PHP5 Deploy")

NiceScale和一般PaaS平台不同的一点是，提供高度灵活和定制，且使用简单。因此你可以在部署过程中看到各种配置选择，从云供应商、地区、操作系统、虚拟机类型、磁盘大小、软件参数等，都可以在线选择修改。NiceScale对软件服务的部署配置提供了默认设置，你可以不做任何修改即可运行良好。

选择"集群"模式，配置使用默认，点击“开始部署“按钮，进入到部署进度页面。

部署完成后，系统会为本次部署创建一个项目名，用于日后的管理。

#### 添加MySQL服务
进入刚才部署的项目，在左侧栏看到有一个"+添加服务"按钮，点击并选择MySQL进入MySQL部署。
![MySQL部署截图](/assets/mysql-deploy.png "MySQL Deploy")

完成后，您的PHP和MySQL已经部署到同一个项目中了。

### 第三步 项目和服务管理
软件一旦部署完成，这些服务就已经开始运行了。你可以点击左侧导航的项目，能够看到刚才部署的服务，出现在里面。点击进入该项目详情页面。

![项目概览截图](/assets/prj-overview.png "Project Overview")

从上面这张图可以看到该项目运行了哪些服务，每个服务运行在哪些虚拟机节点上，服务的目前状态如何等。

我们还可以点击php或mysql服务进入服务页面

![php服务详情截图](/assets/php-detail.png "PHP Detail")


![mysql服务详情截图](/assets/mysql-detail.png "MySQL Detail")

您还可以针对每个服务进行各种管理操作，包括:

- 启动、停止、重启  你可以批量操作所有节点上的服务
- 配置  某些参数想调整了，可以通过这里修改


### 第四步 登录机器
NiceScale提供了服务的可视化面板管理，一般而言，你是不需要登录机器的。但如果你很好奇，想了解NiceScale在你的机器上做了哪些事情，你仍然拥有机器的完全管理权限。

声明：NiceScale官方不保存你的SSH密码或者密钥，无法登录你的机器，你的密码或密钥请自己妥善保管

在刚才部署的项目概览页面中，机器信息中有一列是"SSH登录"，使用您的SSH Key和这里显示的IP:Port即可登录了。


```
ssh -i privatekey.pem -p port root@public_ip (AWS机器上Ubuntu系统用户名是ubuntu)
```

### 第五步 检查服务运行状况
NiceScale所有服务运行于Docker容器中，并且在/opt/nicescale目录下放置了各种管理脚本工具。

##### 我们登录刚才部署了php服务的一台机器
```
ssh -i ~/.ssh/sshgo.pem root@121.201.7.110 -p 60003
     Welcome to use NiceScale.
  You have the following services running on this server:

  +----------------+-----------------------------+------------------------------------+---------+---------------------------+
  | name           | service                     | path                               | status  | created_at                |
  +----------------+-----------------------------+------------------------------------+---------+---------------------------+
  | PHP_Apache_all | nicescale/apache_php:latest | /services/53e190c43733650b31020000 | running | 2014-08-06 10:20:44 +0800 |
  +----------------+-----------------------------+------------------------------------+---------+---------------------------+
root@i-eeuwn0rf:~# 
```

##### 接下来我们查看PHP服务部署在哪里，运行如何
```
root@i-eeuwn0rf:~# ls -l /services
total 4
drwxr-xr-x 7 root root 4096 Aug  6 10:20 53e190c43733650b31020000
root@i-eeuwn0rf:~# cd /services/53e190c43733650b31020000/
root@i-eeuwn0rf:/services/53e190c43733650b31020000# ls -l
total 20
drwxr-xr-x 2 root root 4096 Aug  6 10:20 admin
drwxr-xr-x 8 root root 4096 Aug  6 10:31 conf
drwxr-xr-x 3 root root 4096 Jul 28 12:00 data
drwxr-x--- 2 root adm  4096 Jul 28 12:01 log
drwxr-xr-x 2 root root 4096 Jul 28 17:28 scripts
root@i-eeuwn0rf:/services/53e190c43733650b31020000# cat conf/env
MYSQL_MASTER_HOSTS=192.168.35.4:3306
MYSQL_SLAVE_HOSTS=192.168.35.5:3306
root@i-eeuwn0rf:/services/53e190c43733650b31020000# docker ps
CONTAINER ID        IMAGE                                                        COMMAND                CREATED             STATUS              PORTS               NAMES
c2d7f1f7ae64        gd1-qingcloud.nicedocker.com:5000/nicescale/apache_php:5.5   /opt/nicedocker/run.   About an hour ago   Up About an hour                        53e190c43733650b31020000
```
从上面的目录可以看出，NiceScale将服务的配置(conf)、日志(log)、数据(data)这三个目录放在Host进行管理，并在启动docker时，通过docker的volume绑定到容器内部的对应目录。同时conf目录下的配置文件，是通过配置管理工具来保障配置的一致性和收敛性，通过NiceScale面板对该服务的配置进行的任何变更都将会自动同步到这些节点上。env文件存放在conf目录下，在该项目中添加服务时，将会自动生成相关环境变量保存到该文件中，这样用户在编写php脚本连接相关服务时，可以直接从$CONF_DIR/env中直接读取。

##### 登录到容器内部
```
root@i-eeuwn0rf:~# nicedocker login 53e190c43733650b31020000
root@PHP_Apache_all:/# ps ax
  PID TTY      STAT   TIME COMMAND
    1 ?        Ss     0:00 /usr/sbin/apache2 -DFOREGROUND
   10 ?        Ss     0:00 /usr/sbin/cron
   12 ?        S      0:00 /usr/sbin/apache2 -DFOREGROUND
   13 ?        S      0:00 /usr/sbin/apache2 -DFOREGROUND
   14 ?        S      0:00 /usr/sbin/apache2 -DFOREGROUND
   15 ?        S      0:00 /usr/sbin/apache2 -DFOREGROUND
   16 ?        S      0:00 /usr/sbin/apache2 -DFOREGROUND
```

更多Docker了解，请访问http://docker.com

