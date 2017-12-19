# Linux-Server
Recording a set of operations of Linux servers
# 一、查看及管理当前登录用户

### 1、使用w命令查看登录用户正在使用的进程信息，w命令用于显示已经登录系统的用户的名称，以及他们正在做的事。该命令所使用的信息来源于/var/run/utmp文件。w命令输出的信息包括：

- 用户名称
- 用户的机器名称或tty号
- 远程主机地址
- 用户登录系统的时间
- 空闲时间（作用不大）
- 附加到tty（终端）的进程所用的时间（JCPU时间）
- 当前进程所用时间（PCPU时间）
- 用户当前正在使用的命令

> $ w

```
23:04:27 up 29 days,  7:51,  3 users,  load average: 0.04, 0.06, 0.02
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
ramesh   pts/0    10.1.80.56        22:57    8.00s  0.05s  0.01s sshd: ramesh [priv]
jason    pts/1    10.20.48          23:01    2:53   0.01s  0.01s -bash
john     pts/2    10.1.80.7         23:04    0.00s  0.00s  0.00s w
```

此外，可以使用who am i查看使用该命令的用户及进程，使用who查看所有登录用户进程信息，这些查看命令大同小异；

### 2、使用pkill强制退出登录的用户

使用pkill可以结束当前登录用户的进程，从而强制退出用户登录，具体使用可以结合w命令；

首先：使用w查看当前登录的用户，注意TTY所示登录进程终端号

其次：使用pkill –9 -t pts/1 结束pts/1进程所对应用户登录(可根据FROM的IP地址或主机号来判断）

# 二、查看所有登录用户的操作历史

在linux系统的环境下，不管是root用户还是其它的用户只有登陆系统后用进入操作我们都可以通过命令history来查看历史记录，可是假如一台服务器多人登陆，一天因为某人误操作了删除了重要的数据。这时候通过查看历史记录（命令：history）是没有什么意义了（因为history只针对登录用户下执行有效，即使root用户也无法得到其它用户histotry历史）。那有没有什么办法实现通过记录登陆后的IP地址和某用户名所操作的历史记录呢？答案：有的。

通过在/etc/profile里面加入以下代码就可以实现：

```
PS1="`whoami`@`hostname`:"'[$PWD]'
history
USER_IP=`who -u am i 2>/dev/null| awk '{print $NF}'|sed -e 's/[()]//g'`
if [ "$USER_IP" = "" ]
then
USER_IP=`hostname`
fi
if [ ! -d /tmp/dbasky ]
then
mkdir /tmp/dbasky
chmod 777 /tmp/dbasky
fi
if [ ! -d /tmp/dbasky/${LOGNAME} ]
then
mkdir /tmp/dbasky/${LOGNAME}
chmod 300 /tmp/dbasky/${LOGNAME}
fi
export HISTSIZE=4096
DT=`date "+%Y-%m-%d_%H:%M:%S"`
export HISTFILE="/tmp/dbasky/${LOGNAME}/${USER_IP} dbasky.$DT"
chmod 600 /tmp/dbasky/${LOGNAME}/*dbasky* 2>/dev/null
```

source /etc/profile 使用脚本生效


退出用户，重新登录

上面脚本在系统的/tmp新建个dbasky目录，记录所有登陆过系统的用户和IP地址（文件名），每当用户登录/退出会创建相应的文件，该文件保存这段用户登录时期内操作历史，可以用这个方法来监测系统的安全性。
```
root@zsc6:[/tmp/dbasky/root]$ls 
10.1.80.47 dbasky.2013-10-24_12:53:08 
root@zsc6:[/tmp/dbasky/root]cat 10.1.80.47 dbasky.2013-10-24_12:53:08
```
查看在12:53:08从10.1.80.47登录的root用户操作命令历史
