title: 在CentOS6中升级openSSH到6.7版
date: 2015-11-08 09:16:21
tags:
- centos
- ssh
- linux
categories: linux
---
# 缘起

机房的例行漏洞扫描显示openSSH6.6及以下版本存在以下漏洞:
> "verify_host_key函数 SSHFP DNS RR 检查绕过漏洞(CVE-2014-2653),建议升级OpenSSH至6.6之后版本."

实际上通过yum是无法直接升级到6.6以后版本的(CentOS7好像可以直接升级到更高版本),而rpm包能成功安装的最新版本也只是6.6p1版本,唯一的办法只能通过编译安装了,而这一过程比较痛苦,因为是ssh,所以远程安装时必须先启用telnet备用,待补好后再关闭,事实证明这一步骤__非常非常必要!__

经过三四天的折腾,成功完成了20几台服务器的升级,在此记录下:

<!-- more -->
# 使用yum安装需要的telnet服务以及编译openssh需要的库
登录后先跳转root权限,执行以下命令:

```
cd ~
mkdir sshUpdate && cd sshUpdate \
&& yum install -y xinetd telnet-server make perl pam pam-devel zlib zlib-devel openssl-devel \
&& yum install -y gcc* \
&& vim /etc/xinetd.d/telnet
```
在这里将/etc/xinetd.d/telnet(这里telnet文件名也有可能为krb5-telnet) 中的disable=yes改为no,保存退出.

# 启动telnet服务

```
/etc/init.d/xinetd restart && vim /etc/sysconfig/iptables
```
启动telnet服务并编辑iptables打开23端口,也就是telnet端口,打开后重启iptables

```
service iptables restart
```
此时,应该可以另开一个terminal使用telnet <ip>方式尝试登录服务器了,如果有问题,需要检查上述步骤是否有误,特别要注意23端口除了iptables限制外,还有没有上层的路由或者云主机的管理系统没有打开.

# 下载并编译安装openssh6.7p1
这里推荐先用yum update openssl将openssl升级,至少是1.0.1版本.
执行以下命令:

```
cd ~/sshUpdate \
&& wget http://ftp.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-6.7p1.tar.gz && mv /etc/ssh /etc/ssh.bak \
&& rpm -e `rpm -qa | grep openssh` --nodeps \
&& tar zxf openssh-6.7p1.tar.gz && cd openssh-6.7p1 \
&& ./configure --prefix=/usr --sysconfdir=/etc/ssh --with-pam --with-zlib --with-md5-passwords \
&& make && make install \
&& ssh -V
```
以上指令会先下载6.7的源码,然后备份ssh并卸载当前的openssh,随后将源码进行编译,这里如果执行出错,请检查第一步的相关依赖包没有通过yum安装完整,如果没有,建议更换yum源,比如用阿里的源.
最后一句会显示当前ssh的版本号,如果成功显示openSSH6.7p1,那基本上算是接近成功了.

# 启用ssh服务

```
cp contrib/redhat/sshd.init /etc/init.d/sshd && chkconfig --add sshd \
&& service sshd start && vim /etc/ssh/sshd_config
```
这里将ssh服务做成开机启动并启动了sshd,打开sshd_config后,如果需要连接xshell4,则按下图进行以下修改:
![sshd](/images/openssh_sshd_config.jpg)
保存退出后,重启sshd并查一下sshd服务是否是自启动状态:

```
service sshd restart && chkconfig --list sshd
```

# 关闭telnet
停止telnet服务,并修改配置:

```
/etc/init.d/xinetd stop && vi /etc/xinetd.d/telnet
```
将之前改过的disable=yes又改回去成no.
随后再将修改iptables将23端口关闭,并重启iptables服务.
至此,可以再开ssh登录,用ssh -V查看版本号.


