
> TrueChain因为其组合的共识机制，除了挖矿还能挖水果，并且不管是挖矿还是挖水果，其难度要远远小于比特币的难度。那么，这给我们普通人带来更公平的机会，比特币的共识机制是公平的，但是随着难度的提高，能挖到矿的机会只有几家大型的矿场，普通人已经毫无机会。
> TrueChain在挖水果的机制上，减弱了对机器性能的依赖，让机会更加均等的分布到我们每一个人，那么有必要让我们参与到TrueChain的挖水果或者挖矿中去，这篇文章告诉你怎样让你的机器自动化持续化的挖矿。

## 一、前言

### 1.1 导读
区块链公链的项目都是开源的，你可以在任何时间获取到最新的项目源码，我们部署的矿机就是从拉取开源的项目代码进行部署的。由于项目团队会不定期的对项目进行迭代或者维护，随时有可能更新开源项目的代码。在区块链的网络里面，你运行的机器必须保持和大部分人的运行的规则是一样的，不然你就是“不合规矩的人”，不合规矩的人在整个网络里面是不会被其他区块链节点所认可的。要保整你是一个规矩的遵守者，就必须保证你运行的程序的共识机制是和大多数人一致的。
这时候，我们就需要一种技术方案来保证我机器上的程序（挖矿程序）是最新的TrueChain代码。持续化集成方案

### 1.2 jenkins 介绍
Jenkins是一个开源软件项目，是基于Java开发的一种持续集成工具，用于监控持续重复的工作，旨在提供一个开放易用的软件平台，使软件的持续集成变成可能
Jenkins功能包括：
1、持续的软件版本发布/测试项目。
2、监控外部调用执行的工作。
Jenkins特性：
* 开源的java语言开发持续集成工具，支持CI，CD。
* 易于安装部署配置，它需要有专门的集成服务器来执行集成构建；
* 分布式构建，支持Jenkins能够让多台计算机一起构建/测试
* 支持主流的代码托管工具以及拆件，比如Git、SVN、maven、docker等；

官网地址地址：https://jenkins.io



## 二、环境搭建
挖机部署环境linux较为常用，此文部署在CentOS系统中，其他Linux版本部署类似。
2.1安装准备
| 软件 | 版本 | 描述 |
| ------ | ------ | ------ |
|Java JDK| 1.8.0_181 | yum安装，也可以其他方式安装 |
|ant|  |  |
|Jenkins|  |  |

### 2.2 安装Java JDK
采用yum方式安装，如果已经安装则忽略安装JavaJDK这一步骤
~~~~shell
#  查看JDK软件包列表
$ yum search java | grep -i --color JDK
# 安装javaJDK,采用带*号安装，直接安装java-1.8.0-openjdk所有相关的程序包（如果有错误，请参考5.1)
$ yum install java-1.8.0-openjdk*
# 查看java版本安装是否成功
$ java -version
# 显示如下命令，表示安装成功
openjdk version "1.8.0_181"
OpenJDK Runtime Environment (build 1.8.0_181-b13)
OpenJDK 64-Bit Server VM (build 25.181-b13, mixed mode)

~~~~~~

### 2.3 安装ant
Ant是一个Apache基金会下的跨平台的构建工具，它是一个将软件编译、测试、部署等步骤联系在一起加以自动化的一个工具。
在本文中，我主要让介绍Ant的命令、构建文件、最后部分以一个实例概述在持续集成工具jenkins中的应用。
安装ant使用手动安装，具体安装步骤如下：
~~~~shell

# 下载ant安装包
$ wget http://mirror.bit.edu.cn/apache//ant/binaries/apache-ant-1.10.5-bin.tar.gz
# 解压
$ tar -zxvf apache-ant-1.10.5-bin.tar.gz
# 移动到/usr/share下
$ cp -r apache-ant-1.10.5 /usr/share
# 切换到/usr/share目录下，重命名
$ cd /usr/share
$ mv apache-ant-1.10.5 ant
# 配置环境变量
$ vim /etc/profile
# set Ant enviroment
export PATH=$PATH:/usr/share/ant/bin
# 立刻将配置生效
$ source /etc/proifle
# 检查是否安装成功
$ ant -version
# 提示如下信息，表示ant安装成功
Apache Ant(TM) version 1.10.5 compiled on July 10 2018



~~~~

### 2.4 安装jenkins

jenkins下载地址：http://pkg.jenkins-ci.org/redhat/
本例中使用最新的版本，如果已经安装则可以跳过此步骤，升级版本则需要删除原有的版本之后再安装

~~~~~shell
# 检查是否已经安装了jenkins
$ rpm -qa|grep jenkins
# 卸载原先版本的jenkins
rpm -e nodeps jenkins-2.58-1.1.noarch
~~~~~~

安装最新版的jenkins
~~~~~~shell

# 下载jenkins
$ wget http://pkg.jenkins-ci.org/redhat/jenkins-2.142-1.1.noarch.rpm
# 安装jenkins-2.142-1.1.noarch.rpm
$ sudo rpm -ih jenkins-2.142-1.1.noarch.rpm
# 出现如下信息则表示安装成功
warning: jenkins-2.142-1.1.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID d50582e6: NOKEY
################################# [100%]
Updating / installing...
################################# [100%]

# 启动jenkins,需要前置安装java虚拟环境，否则报错，安装步骤参考2.2
$ sudo service jenkins start
# 出现如下信息，表示启动成功
Starting jenkins (via systemctl):                          [  OK  ]
# 注意，如果是后置安装java的虚拟环境，需要在“vi /etc/init.d/jenkins”，把jdk的路径添加进去
~~~~~~

## 三、配置
到这一步，环境的安装环境已经基本完成，接下来要进行相关环境的配置

### 3.1 修改jenkins的配置文件
~~~~~~shell
# 打开/etc/sysconfig/jenkins文件
$ vim /etc/sysconfig/jenkins
#修改为18080，默认是8080
JENKINS_PORT="18080"
#内存设置，我这里设置成如下配置
JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true -Xms512m -Xmx1024m -XX:MaxNewSize=512m -XX:MaxPermSize=1024m"
~~~~~~

### 3.2 初始化软件环境
在浏览器中输入 121.42.31.210:8080/jenkins （默认是使用8080端口）
打开Jenkins控制页面之后，根据页面提示在服务指定目中找到密码
![解锁Jenkins](https://raw.githubusercontent.com/zouwei/resource/master/images/jenkins/jenkins-01.png)

登录成功后回让你选择插件的安装，可以选择建议的安装也可以自己进行选择，不清楚的话可以使用建议的安装
由于建议安装的插件比较多，安装的过程有点慢，多等待一会
![插件安装]()

### 3.3 初始化工作配置
**修改工作空间**
从主页面直接到“系统管理>系统配置”，点击右边“高级”按钮



# 查看java 的安装位置
whereis javac
javac: /usr/bin/javac /usr/share/man/man1/javac.1.gz

## 四、自动化持续集成项目


## 五、常见错误处理


### 5.1 解决YUM下Loaded plugins: fastestmirror Determining fastest mirrors 的错误问题

~~~~~~shell
# 修改fastestmirror.conf配置文件
$ vim  /etc/yum/pluginconf.d/fastestmirror.conf

# 修改如下配置，保存
enabled=0  #把1改为0  

# 修改yum.conf配置文件
$ vim /etc/yum.conf
# 修改如下配置，保存
plugins=1                 #将plugins的值修改为0

# 更改如上两个配置文件之后，再执行yum带*安装就OK了
~~~~~~



### 5.2 修改/etc/proifle文件，改错了导致所有命令无法执行
~~~~~~shell



~~~~~~

