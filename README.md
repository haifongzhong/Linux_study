# Linux学习笔记

日期：2021/11/7

## 序言

### 1）安装准备

​	虚拟机使用过两款，这里只是做简单的了解：①VMware Workstation Pro 虚拟机/②Order VM VirtualBox

​		两款软件各有差别，开始接触Linux使用的是②，觉得特别像安装了一台Linux主机在本地，但是用了几天就放弃了，主要是快捷键特别不顺手，后面改用①，发现用起来真不错，主要是配合Xshell远程登录，同时VMware可以在后台运行，感觉前面觉得丑啥的一点问题都不是，所以推荐使用①。

​	安装时需要好好研究一下，多重装几次，最开始的时候纠结要不要安装图形化界面，这个看个人，因为我的电脑配置不高，所以给虚拟机的配置比较低，安装图像化界面特别影响性能，所以干脆又不安装了，尤其是后面使用Xshell访问根本不会去想要登录图形界面了。

### 2）安装虚拟机

​	本次使用的时CentOS7和VMware 15，步骤推荐使用自定义的步骤，参考网上教程安装，启动前把网络配置中的模式改成“nat模式”，同时在高级哪里随机获取一个mac地址。

​	启动虚拟机，在安装位置上手动分区 （/boot、swap、/）。其中/boot存放引导文件，一般只需要200MB就够了，系统默认为1024MB所以要改，swap为暂时文件给2048MB就够，剩余的全部给/也行，同时文件系统格式改为xfs、swap、xfs。

​	软件选择选用最小安装，勾选开发工具和兼容性程序即可

​	KDUMP直接去掉，默认勾选，但是我们学习的化就不要了，一般在正式运维的服务器上一定要有，用来记录运行的日志。

​	其余设置默认。。。

​	安装、配置ROOT密码。

## 第一章	网络配置

​	在安装完虚拟机后第一步应给是配置网络，同时把虚拟机和主机网卡的关系捋一遍：

### 1）察看ip地址

![image-20211107201601953](https://gitee.com/zhong-haifeng/drawing-bed/raw/master/blogImage/202302261527871.png)

Linux参看IP地址的配置命令：nmcli、IP add ，ifconfig（centos7基本使用nmcli，需要的话自己再安装i服从fig的网络插件）。。。

![image-20211107202225187](https://gitee.com/zhong-haifeng/drawing-bed/raw/master/blogImage/202302261527909.png)

Windows下查看ip的命令：ipconfig、ipconfig /all。。。

### 2）设置Windows主机虚拟网卡的静态地址

打开网络和Internet设置——更改适配器选项——选择VMnet8——Internet协议版本4（TCP/IPv4）属性——选择使用下面的IP地址，这里参考上图个人主机上分配的ip地址，DNS设置一样就行（百度一下114.114.114.114、8.8.8.8的区别），VMnet1也可以如此设置，当然不用主机模式也可以不设置。

![image-20211107202840656](https://gitee.com/zhong-haifeng/drawing-bed/raw/master/blogImage/202302261527939.png)

### 2）查看nat模式配置

​	打开Vmware——点击编辑——点击虚拟网络编辑器——点击更改设置，可观察到有三个设置好的网络，选中VMnet8 查看net设置和DHCP设置，这里有两个参数是我们在设置Linux主机的静态IP需要使用的，一个是在net设置中看到的网关：192.168.26.2，还有一个是在DHCP设置中看到的地址池：192.168.26.128-192.168.26.254

![image-20211107203956597](https://gitee.com/zhong-haifeng/drawing-bed/raw/master/blogImage/202302261527974.png)

**<u>到这一步，因为改动了网卡，所以最好重启一下你的Windows电脑</u>**

### 3）虚拟机静态IP地址设置

​	root用户登录Linux，打开网卡的配置文件

`#cd /etc/sysconfig/network-scripts/`

​	使用vi（vim）打开ifcfg-ens33

`#vi ifcfg-ens33`

```
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"			//注意这里改成static
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="733016d4-7b44-4309-95ad-814f44e4d0cd"
DEVICE="ens33"
ONBOOT="yes"            	//开机启动设置为yes
IPV6_PRIVACY="no"

IPADDR="192.168.26.141"		//地址设置为地址池范围中的任意一个
NETMASK="255.255.255.0"		//地址掩码
PREFIX=24
GATEWAY="192.168.26.2"		//网关，让大家注意记下来的
DNS1="114.114.114.114"		//dns服务器1 中国
DNS2="8.8.8.8"				//dns服务器2 谷歌

```

根据注释内容修改自己的配置，修改完保存重启service，命令如下

`service network restart`

### 4）测试

​	查看Linux的地址：nmcli

​	ping网关，虚拟网卡VMnet8：

![image-20211107210952169](https://gitee.com/zhong-haifeng/drawing-bed/raw/master/blogImage/202302261527007.png)

Windows主机ping虚拟机IP地址：

![image-20211107211124298](https://gitee.com/zhong-haifeng/drawing-bed/raw/master/blogImage/202302261527049.png)

Ok到这了基本上实现的地址的配置，没有ping通的强烈建议重启电脑（你的Windows电脑）

## 第二章 Xshell远程登录

使用Xshell远程登录，可以解决许多Vmware本地登录的许多不友好的问题，比如快捷键重复，中文输入输出问题，以及必要的复制粘贴问题。

进入Xshell[官网下载](https://www.netsarang.com/zh/free-for-home-school/)，我们可以填写学生信息免费使用。

下载安装，打开Xshell，配置远程访问

![image-20211108112741348](https://gitee.com/zhong-haifeng/drawing-bed/raw/master/blogImage/202302261527081.png)

## 第三章 yum的配置

### 1）什么是yum

​	yum使用来安装rpm

## 第四章 docker的使用

### 1)docker安装MySQL5.7

> 命令配置：

```
docker run -p 3306:3306 --name mysql5.7  -v /usr/local/dockermysql/log:/var/log/mysql -v /usr/local/dockermysql/data:/var/lib/mysql -v /usr/local/dockermysql/conf:/etc/mysql -e MYSQL_ROOT_PASSWORD=@18370435674  -d mysql:5.7
```

​	在“usr/local/”目录下创建“dockermysql/data、log、conf”分别挂载MySQL中的数据、日志、配置文件目录

### 2）dockerfile配置tomcat8.5+jdk11

> 使用dockerfile配置镜像

```
#指定一个基础镜像

FROM centos:7

#作者信息

MAINTAINER hafongzhong<haifongzhong288@gmail.com>
WORKDIR /usr

# 创建两个新目录来存储jdk文件和tomcat文件

RUN mkdir /usr/local/java /usr/local/tomcat

#将jdk压缩文件复制到镜像中，它将自动解压缩tar文件

ADD jdk-11.0.13_linux-x64_bin.tar.gz /usr/local/java/
ADD apache-tomcat-8.5.73.tar.gz /usr/local/tomcat/

#配置环境变量

ENV JAVA_HOME /usr/local/java/jdk-11.0.13 
#ENV JRE_HOME $JAVA_HOME/jre
#ENV CLASSPATH $JAVA_HOME/bin/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CALSSPATH
ENV CATALINA_HOME /usr/local/tomcat/apache-tomcat-8.5.73
ENV CATALINA_BASE /usr/local/tomcat/apache-tomcat-8.5.73
ENV PATH $JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin:$PATH

#容器运行时的监听端口8080
EXPOSE 8080

#配置容器启动后执行的命令
ENTRYPOINT /usr/local/tomcat/apache-tomcat-8.5.73/bin/startup.sh && tail -f /usr/local/tomcat/apache-tomcat-8.5.73/logs/catalina.out
#启动时运行tomcat
CMD ["/usr/local/tomcat/apache-tomcat-8.5.73/bin/catalina.sh","run"]

# VOLUME 指定了临时文件目录为/tmp
# 其效果是在主机 /var/lib/docker 目录下创建了一个临时文件，并链接到容器的/tmp
VOLUME /tmp
```

​	编译镜像：docker build -t='haifongzhong/tomcat8.5-jdk11' .   //最后面还有一个“.”

> 创建镜像容器

```
docker run -id --name=mytomcat -p 9000:8080  -v /usr/local/webapps:/usr/local/tomcat/apache-tomcat-8.5.73/webapps haifongzhong/tomcat8.5-jdk11
```

​	在运行前创建一个目录用来挂载webapps目录：`mkdir -p /usr/local/webapps/  `

​	将war包放到该目录下自动解压。
