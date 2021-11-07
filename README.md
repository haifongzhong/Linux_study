# Linux学习笔记

日期：2021/11/7

## 序言

##### 1）安装准备

​	虚拟机使用过两款，这里只是做简单的了解：①VMware Workstation Pro 虚拟机/②Order VM VirtualBox

​		两款软件各有差别，开始接触Linux使用的是②，觉得特别像安装了一台Linux主机在本地，但是用了几天就放弃了，主要是快捷键特别不顺手，后面改用①，发现用起来真不错，主要是配合Xshell远程登录，同时VMware可以在后台运行，感觉前面觉得丑啥的一点问题都不是，所以推荐使用①。

​	安装时需要好好研究一下，多重装几次，最开始的时候纠结要不要安装图形化界面，这个看个人，因为我的电脑配置不高，所以给虚拟机的配置比较低，安装图像化界面特别影响性能，所以干脆又不安装了，尤其是后面使用Xshell访问根本不会去想要登录图形界面了。

##### 2）安装虚拟机

​	本次使用的时CentOS7和VMware 15，步骤推荐使用自定义的步骤，参考网上教程安装，启动前把网络配置中的模式改成“nat模式”，同时在高级哪里随机获取一个mac地址。

​	启动虚拟机，在安装位置上手动分区 （/boot、swap、/）。其中/boot存放引导文件，一般只需要200MB就够了，系统默认为1024MB所以要改，swap为暂时文件给2048MB就够，剩余的全部给/也行，同时文件系统格式改为xfs、swap、xfs。

​	软件选择选用最小安装，勾选开发工具和兼容性程序即可

​	KDUMP直接去掉，默认勾选，但是我们学习的化就不要了，一般在正式运维的服务器上一定要有，用来记录运行的日志。

​	其余设置默认。。。

​	安装、配置ROOT密码。

## 第一章	网络配置

​	在安装完虚拟机后第一步应给是配置网络，同时把虚拟机和主机网卡的关系捋一遍：

##### 1）察看ip地址

![image-20211107201601953](C:\Users\haifongzhong\AppData\Roaming\Typora\typora-user-images\image-20211107201601953.png)

Linux参看IP地址的配置命令：nmcli、IP add ，ifconfig（centos7基本使用nmcli，需要的话自己再安装i服从fig的网络插件）。。。

![image-20211107202225187](C:\Users\haifongzhong\AppData\Roaming\Typora\typora-user-images\image-20211107202225187.png)

Windows下查看ip的命令：ipconfig、ipconfig /all。。。

##### 2）设置Windows主机虚拟网卡的静态地址

打开网络和Internet设置——更改适配器选项——选择VMnet8——Internet协议版本4（TCP/IPv4）属性——选择使用下面的IP地址，这里参考上图个人主机上分配的ip地址，DNS设置一样就行（百度一下114.114.114.114、8.8.8.8的区别），VMnet1也可以如此设置，当然不用主机模式也可以不设置。

![image-20211107202840656](C:\Users\haifongzhong\AppData\Roaming\Typora\typora-user-images\image-20211107202840656.png)

##### 2）查看nat模式配置

​	打开Vmware——点击编辑——点击虚拟网络编辑器——点击更改设置，可观察到有三个设置好的网络，选中VMnet8 查看net设置和DHCP设置，这里有两个参数是我们在设置Linux主机的静态IP需要使用的，一个是在net设置中看到的网关：192.168.26.2，还有一个是在DHCP设置中看到的地址池：192.168.26.128-192.168.26.254

![image-20211107203956597](C:\Users\haifongzhong\AppData\Roaming\Typora\typora-user-images\image-20211107203956597.png)

##### 3）虚拟机网卡设置



##### 4）测试

