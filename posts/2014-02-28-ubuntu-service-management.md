---
layout: post
title: ubuntu服务管理
published: true
tags: [ubuntu linux system service 服务 服务管理]
---

## System V

linux的启动过程会经过如下几步:

1. 运行BIOS，进行自检等操作
2. 查找引导程序(MBR)
3. 执行引导程序：grub等(用于加载内核)
4. 加载内核
5. 执行init程序(init是linux运行的第一个程序，pid为1)
6. 进入相应的runlevel，并启动runlevel对应的服务

也就是说从第5步开始才真正进入到linux的世界。init位于/sbin/init，
是初始化所有的设备驱动程序和数据结构之后，由内核启动的一个用户级程序，
由init程序进而完成系统的启动过程。

init启动后，它会去启动其他服务，那么怎么判断需要启动哪些服务呢。这引出了另外一个
概念"运行级别"(runlevel)。系统默认定义了7个runlevel，他们是:

* 0-关机
* 1-单用户模式
* 2-不带网络的多用户模式
* 3-带网络的多用户模式，纯文本界面
* 4-未使用
* 5-带网络的多用户模式，图形界面
* 6-重启

ubuntu下的runlevel略有差异:

* 0-关闭系统
* 1-单用户模式
* 2~5-完整的多用户模式
* 6-重启

每个runlevel下都定义了需要启动和不需要启动的服务。通过<code>runlevel</code>命令
可以查看当前运行的级别。

接下来的问题就是init如何判断运行哪个runlevel的服务。init会读取<code>/etc/inittab</code>文件，
此文件中设置了默认runlevel。ubuntu下略有差异，默认情况下<code>/etc/inittab</code>文件
是不存在的，默认runlevel定义在<code>/etc/init/rc-sysinit.conf</code>中，文件中定义
<code>env DEFAULT_RUNLEVEL=2</code>，这表明ubunut默认进入的runlevel是2。
另外，在此文件中还有一段以<code>if [ -r /etc/inittab ]</code>开始的代码，
这里保留了使用inittab指定系统默认运行级别的功能，也就是说，如果用户手动创建了
<code>/etc/inittab</code>，那么init将以<code>/etc/inittab</code>中指定的默认runlevel进行系统的启动。
比如说用户希望系统以级别3为默认运行级别，则只需在inittab文件中加入如下一行：<code>id:3:initdefault:</code>

确定runlevel后，init就会启动/关闭relevel对应目录下定义的服务。这些目录是
<code>/etc/rc[?].d</code>，例如ubuntu的默认runlevel是2，就会启动/etc/rc2.d下的服务。

打开/etc/rc[?].d目录，会发现这些目录下的文件都是形如<code>Snnxxxx</code>或
<code>Knnxxxx</code>的符号链接，而且都是指向/etc/init.d。
也就是说不同runlevel下服务的启动或关闭脚本均是放在/etc/init.d下，只不过根据不同
级别的需要，在对应/etc /rc[?].d目录下放一个链接，不同的级别会需要不同的服务，
因此不同/etc/rc[?].d目录下的链接文件也不尽相同。

其中链接文件中以S开头的表示在调用/etc/init.d目录中对应脚本的时候会传递一个start
参数，也就是启动对应服务，而以K开头的则是传递一个stop参数，由此关闭此服务，
此处的K表示kill。S和K后面的nn是一个数字，表示本脚本被执行的先后顺序，小号在前
大号在后，这样以解决不同服务之间可能存在的先后依赖关系。比如说ftp服务依赖于网络
服务的启动，所以ftp服务的编号就要大于网络服务的编号，在网络服务启动后再行启动。
最后的xxxx则是服务的名字。

另外，除了/etc/rc[0~6].d文件外，还有一个/etc/rcS.d目录，这个目录下的服务脚本与
/etc/rc[0~6].d格式类似，也为指向/etc/init.d中的脚本的链接，但是会在
/etc/rc[0~6].d中的脚本执行前被执行。所有runlevel都需要的服务就可以定义在
/etc/rcS.d中。

安装服务的过程就明确了：

1. 在/etc/init.d中添加一个服务的启动脚本
2. 在需要启动服务的runlevel的/etc/rc[0~6].d中按照文件名格式添加一个符号链接，指向/etc/init.d中的脚本

以在ubunut下安装apache2为例，

1. 把apache2的启动脚本<code>/usr/local/apache2/bin/apachectl</code>拷贝到/etc/init.d下，<code>sudo cp /usr/local/apache2/bin/apachectl /etc/init.d/httpd</code>
2. 创建链接，假设启动序号是80，<code>sudo ln -s /etc/init.d/httpd /etc/rc2.d/S80httpd</code>

服务的启动、关闭、重启如下:
<pre>
service httpd start
service httpd stop
service httpd restart
</pre>
如果想将httpd服务改成开机不启动，只需<code>sudo mv /etc/rc2.d/S80httpd /etc/rc2.d/K80httpd</code>

以上linux管理服务的框架叫做System V(罗马数字5)。

除了手动管理外，还有很多工具可以对服务进行管理，比如常用的有update-rc.d、chkconfig和sysv-rc-conf等。

## Upstart

从上面的介绍可以看出system V的启动是线性的，S80必须要等S20启动后才能启动，不管S80是不是依赖S20。
Ubuntu从6.10开始逐步用Upstart代替原来的system V。RHEL(CentOS)也都从版本6开始转用Upstart。

Upstart(Upstart init daemon)是基于事件的启动系统，它使用事件来启动和关闭系统服务。
Upstart是是并行的，只要事件发生,服务可以并发启动。这种方式无疑要优越得多，
因为它可以充分利用现在计算机多核的特点，大大减少启动所需的时间。基于时间的思路
相信所有前端都不陌生。

ubuntu下/etc/init.d里很多服务都是链接到/lib/init/upstart-job的软连接，也就是使用Upstart进行的启动。
这就是为什么有时候使用System V的方法将服务关闭了，但是服务依然开机启动，比如vfstpd。
<pre>
    cd /etc/rc2.d
    sudo mv S80vsftpd K80vsftpd
</pre>

查看/etc/init/vsftpd.conf
<pre>
    vim /etc/init/vsftpd.conf

    start on runlevel [2345] or net-device-up IFACE!=lo
    stop on runlevel [!2345]
</pre>

可以看到在2,3,4,5的级别下都是启动的，要修改成开机不启动，可以这样：
<pre>
    stop on runlevel [2345] or net-device-up IFACE!=lo
    start on runlevel [!2345]
</pre>

这是一个简单介绍，供参考。

## 参考资料

[http://upstart.ubuntu.com/](http://upstart.ubuntu.com/)    
[http://www.mike.org.cn/articles/understand-upstart/](http://www.mike.org.cn/articles/understand-upstart/)
[默认安装的vsftpd取消开机启动](http://hi.baidu.com/pengjunlong/item/e3e98e175bff7c9e99ce339c)
