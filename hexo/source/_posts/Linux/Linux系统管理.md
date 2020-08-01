---
title: Linux系统管理
date: 2020-04-21 10:07:38
tags:
categories:
---

## linux系统管理

<!--more-->

```bash
参数替换xargs
[root@localhost ~]# echo {1..100}|xargs -n1
[root@localhost ~]# echo user{1..10}|xargs -n1 useradd

取ip地址
[root@localhost ~]# ifconfig|sed -rn '2s/[^0-9]+([0-9.]+).*/\1/p'
[root@localhost ~]# ifconfig eth0 |sed -r '2!d; s@(.*inet )(.*)(netmask.*)@\2@'

扫描系统新硬盘设备
[root@localhost ~]# echo "- - -" > /sys/class/scsi_host/host2/scan

查看该目录是否被访问
[root@localhost ~]# lsof /mnt/cdrom
COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
bash    49866   root  cwd    DIR   11,0     2048 1856 /mnt/cdrom
bash    49949 xingbo  cwd    DIR   11,0     2048 1856 /mnt/cdrom

给pts/1客户端发送消息
[root@localhost ~]# echo please exit /mnt/cdrom > /dev/pts/1

查看该目录被哪些用户访问
[root@localhost ~]# fuser -v /mnt/cdrom
                     USER        PID ACCESS COMMAND
/mnt/cdrom:          root     kernel mount /mnt/cdrom
                     root      49866 ..c.. bash
                     xingbo    49949 ..c.. bash

剔除访问该目录的用户
[root@localhost ~]# fuser -km /mnt/cdrom
/mnt/cdrom:          49866c 49949c

查看该目录是否处于挂载状态
[root@localhost ~]# findmnt /mnt/cdrom
TARGET SOURCE FSTYPE OPTIONS
/mnt/cdrom /dev/sr0 iso9660 ro,relatime

查看支持的文件系统
[root@localhost ~]# ls /lib/modules/$(uname -r)/kernel/fs

创建文件系统并挂载
[root@localhost ~]# dd if=/dev/zero of=/tmp/hello.img bs=1M count=100
[root@localhost ~]# mkfs.xfs /tmp/hello.img
[root@localhost ~]# blkid /tmp/hello.img

弹出光驱
[root@localhost ~]# eject
弹入光驱
[root@localhost ~]# eject -t

dd命令
dd if=读取文件，of=写入文件，bs=指定块大小，ibs（一次读入大小），obs(一次写入大小)，
skip=跳过ibs大小的块开始读取，seek=跳过obs大小的块开始写入，count=数量；    
conv=用指定的参数转换文件，lcase把大写字符转换为小写字符，ucase把小写字符转换为大写字符>
（例：dd if=/etc/fstab of=FSTAB conv=ucase）,
fdatasync 直接写入物理磁盘（dd if=/dev/zero of=/tmp/f1 bs=1M count=100 conv=fdatasync 
测试硬盘写入速度），
notrunc不截断输出的文件，dd if=/dev/sd |gzip >/.gz

逻辑卷 
pvcreate /dev/sdc            #生成物理卷
pvs、pvdisplay                #查看物理卷
vgcreate vg0 /dev/sd{a,b}    #创建卷组，并把物理卷加入卷组
vgs、vgdisplay                #查看卷组信息
lvcreate -n mysql -L 4G vg0   #创建一个逻辑卷，-n卷名
lvs、lvdisplay                #查看逻辑卷
mkfs.xfs /dev/vg0/mysql       #创建文件系统
vgextend vg0 /dev/sdd         #扩展卷组容量
lvextend -l -r +100%free /dev/sde    #增加逻辑卷大小，-r自动同步文件系统，
                              # resize2fs ext系列同步，xfs_growfs xfs系列同步
pvmove /dev/sda               #物理卷搬家
vgreduce vg1 /dev/sdc         #将sdc移除卷组
pvremove /dev/sdc             #将物理卷变成普通磁盘
vgrename                      #改名

跨主机迁移卷组
umount 、vgchange -an         #改为非活动
vgexport                    #导出
vgimport                    #导入新机器
vgchange -ay  、 mount      #激活

查看端口占用情况
[root@localhost ~]# netstat -nltp|grep 22
[root@localhost ~]# lsof -i:22

TAB的补全包
bash-completion

查看本机DNS
[root@localhost ~]# cat /etc/resolv.conf

抓包
[root@localhost ~]# tcpdump -i eth0 -nnX icmp
[root@localhost ~]# tcpdump -i eth0 -nnX port 22

系统管理工具
pstree、ps、pidof、top、htop、glances、pmap、vmstat、dstat、nohub、iotop、nload、lsof

查看所有网络连接
[root@localhost ~]# lsof -i -n
[root@localhost ~]# lsof -i @192.168.0.20
shell脚本练习
[root@localhost ~]# for i in a b c d;do echo $i;done
[root@localhost ~]# for i in {a..z};do echo $i;done
[root@localhost ~]# for i in {1..10..2};do echo $i;done
[root@localhost ~]# seq 1 3 10
[root@localhost ~]# for i in *.sh;do echo i is $i;done

更改文件名
[root@localhost ~]# vim rename.sh
#!/bin/bash
DIR=/root/testdir
cd $DIR
for i in *;do
        j=`echo $i|sed -rn 's/(.*)\..*/\1/p'`
        mv $i ${j}.log
done

批量创建用户
[root@localhost ~]# vim users.sh
#!/bin/bash
. /etc/init.d/functions
if [ "$1" = '-c' ];then
        for USER in `cat /tmp/name.txt`;do
                if id $USER &> /dev/null;then
                        action "$USER is exist" false
                else
                        useradd $USER
                        echo $USER | passwd --stdin $USER &> /dev/null
                        action "$USER is created" true
                fi
        done
elif [ "$1" = '-r' ];then
        for USER in `cat /tmp/name.txt`;do
                userdel -r $USER &> /dev/null
                action "$USER is remove" true
        done
else
        echo "Usage: `basename $0` -c | -r "
fi
[root@localhost ~]# bash users.sh -c
[root@localhost ~]# bash users.sh -r

棋盘
[root@localhost ~]# vim chess1_while.sh
#!/bin/bash
i=1
BEGIN='\e[1;'
RED=41m
YELL=44m
while [ $i -le 8 ];do
        j=1
        while [ $j -le 4 ]; do
                if [ $((i%2)) -eq 0 ];then
                        echo -e "${BEGIN}${RED}  ${BEGIN}${YELL}  \c"
                else
                        echo -e "${BEGIN}${YELL}  ${BEGIN}${RED}  \c"
                fi

                let j++
        done
        echo -e "\e[0m"
        let i++
done

[root@localhost ~]# vim chess2_while.sh
#!/bin/bash
i=1
BEGIN='\e[1;'
RED=41m
YELL=44m
flag=true
while [ $i -le 8 ];do
        if $flag;then
                j=1
        while [ $j -le 4 ]; do
                        echo -e "${BEGIN}${RED}  ${BEGIN}${YELL}  \c"
                let j++
                done
                flag=false
        else
                j=1
                while [ $j -le 4 ];do
                        echo -e "${BEGIN}${YELL}  ${BEGIN}${RED}  \c"
                let j++
                done
                flag=true
        fi
        echo -e "\e[0m"
        let i++
done

[root@localhost ~]# vim chess3_while.sh
#!/bin/bash
red(){
        echo -e "\033[41m        \033[0m\c"
}
yel(){
        echo -e "\033[43m        \033[0m\c"     
}
redyel(){
        for ((i=1;i<=4;i++));do
                for ((j=1;j<=4;j++));do
                        red;yel
                done
                echo
        done
}
yelred(){
        for ((i=1;i<=4;i++));do
                for ((j=1;j<=4;j++));do
                        yel;red
                done
                echo 
        done
}
for ((line=1;line<=8;line++));do
        [ $[$line%2] -eq 0 ] && redyel || yelred
done


[root@localhost ~]# vim chess4_while.sh
#!/bin/bash
red(){
        echo -e "\033[41m        \033[0m\c"
}
yel(){
        echo -e "\033[43m        \033[0m\c"     
}
redyel(){
        for ((i=1;i<=4;i++));do
                for ((j=1;j<=4;j++));do
                        [ "$1" = "-r" ] && { yel;red; } || { red;yel; }
                done
                echo
        done
}
for ((line=1;line<=8;line++));do
        [ $[$line%2] -eq 0 ] && redyel || redyel -r
done

监控httpd
[root@localhost ~]# yum -y install mailx sendmail
[root@localhost ~]# vim monitor_httpd_while.sh
#!/bin/bash
SLEEPTIME=30
SERVICE=httpd
LOG=/var/log/monitor_$SERVICE.log
while true;do
    if killall -0 $SERVICE &>/dev/null;then
        true
    else
        systemctl restart $SERVICE
        echo "AT `date +'%F %T'` $SERVICE is restart" | tee -a  $LOG | mail  -s warning root
    fi
    sleep $SLEEPTIME
done

10位随机数字挑出最大和最小值
#!/bin/bash
for ((i=0;i<10;i++));do
        N=$RANDOM
        echo -e "$N \c"
        if [ $i -eq 0 ];then
                MAX=$N
                MIN=$N
        else
                [ $N -gt $MAX ]&& MAX=$N
                [ $N -lt $MIN ]&& MIN=$N
        fi
done
echo 
echo MAX=$MAX,MIN=$MIN


使用shift创建用户
#!/bin/bash
if [ -z "$1" ];then
        echo "Usage: `basename $0` username ..."
        exit
fi
while [ "$1" ];do
        if id $1 &>/dev/null;then
                echo $1 is exist
        else
                useradd $1
                echo $1 is created
        fi
        shift
done

使用while创建用户
#!/bin/bash
while read name;do
        useradd $name
        echo $name is created
done < /tmp/name.txt

#!/bin/bash
cat /tmp/name.txt|while read name;do                            
        useradd $name
        echo $name is create
done 

判断分区使用率是否大于80
[root@localhost ~]# vim partition.sh
#!/bin/bash
df| while read line ;do
        DEVICE=`sed -rn '/^\/dev\/sd/s/(^[^ ]+).* ([0-9]+)%.*/\1/p'`
        USE=`df|sed -rn '/^\/dev\/sd/s/(^[^ ]+).* ([0-9]+)%.*/\2/p'`
        if [ $USE -gt 80 ];then
                wall $DEVICE will be full,USE:$USE
        fi
done

扫描网络中主机
[root@localhost ~]# vim scan_host.sh
#!/bin/bash
NET=192.168.0
for HOST in {1..254};do
        { ping -c1 -W1 $NET.$HOST &> /dev/null && echo $NET.$HOST is up || echo $NET.$HOST is down; } &

done
wait
echo "scan host is finished"

#!/bin/bash
NET=172.22
for SUBNET in {0..255};do
        for HOST in {1..254};do
        { ping -c1 -W1 $NET.$SUBNET.$HOST &> /dev/null && echo $NET.$SUBNET.$HOST is up; } &

        done
        wait
done
echo "scan host is finished"


拒绝频繁访问
#!/bin/bash
sed -rn '/^ESTAB/s/.*([0-9.]+):[0-9]+.*$/\1/p'ss.log|sort|uniq -c|while read times ip;do
        if [ $time -gt 10 ];then
                iptables -A INPUT -s $ip -j REJECT
        fi

#!/bin/bash
PS3="please choose a number: "
select menu in back clean config quit;do
        case $menu in
        back)
                ./backup.sh
                ;;
        clean)
                ./clean.sh
                ;;
        config)
                ./config.sh
                ;;
        quit)
                break
                ;;
        *)
                echo input false
        esac
done
echo "完成"


使用数组查找随机数字中最大和最小值
#!/bin/bash
declare -a ARR
for((i=0;i<10;i++));do
        ARR[$i]=$RANDOM
        if [ $i -eq 0 ];then
                MAX=${ARR[$i]}
                MIN=${ARR[$i]}
        else
                [ ${ARR[$i]} -gt $MAX ] && MAX=${ARR[$i]}
                [ ${ARR[$i]} -lt $MIN ] && MIN=${ARR[$i]}

        fi
done
echo all random are ${ARR[@]}
echo MAX=$MAX,MIN=$MIN
/var/log/btmp登录失败的信息              lastb命令查看     lastb -f 查看非本机的文件

创建临时文件mktemp
mktemp file.nameXXX（3个XXX以上）    创建目录mktemp -d

安装复制文件install 命令
install /etc/issue /data/issue.bak   -d复制目录
install -m 770 -u xing -g bin /etc/issue /data/issue.bak

expect自动化交互式操作
yum -y install expect   expect -c 从命令中执行expect脚本，默认expect是交互执行的，-d可以输出调试信息

expect中相关命令：spawn启动新的进程、send用于向进程发送字符串、expect从进程接受字符串、interact允许用户交互
exp_continue匹配多个字符串在执行动作后加此命令

[root@localhost ~]# vim expect.sh
#!/bin/bash
ip=$1
user=$2
password=$3
expect <<EOF
set timeout 20
spawn ssh $user@$ip
expect {
"yes/no" { send "yes\n";exp_continue }
"password" { send "$password\n" }
}
expect "]#" { send "useradd hehe\n" }
expect "]#" { send "echo magedu |passwd --stdin hehe\n" }
expect "]#" { send "exit\n" }
expect eof
EOF


centos6启动流程
  加载BIOS的硬件信息，获取启动设备
  读取第一个启动设备MBR的引导加载程序grub的启动信息
  加载内核，读取分区驱动模块
  启动第一个进程程序init
  init启动程序执行/etc/rc.d/rc/sysinit文件
  启动核心的外挂模式
  init执行运行各个批处理文件scripts
  init执行/etc/rc.d/rc.local
  执行/bin/login程序，等待登录

centos7启动流程
  引导顺序UEFI或BIOS初始化，运行POST开机自检 选择启动设备
  引导装载程序，grub2
  加载装载程序的配置文件：/etc/grub.d/ /etc/default/grub /boot/gtub2/grub.cfg
  加载initramfs驱动模块
  加载内核选项
  内核初始化，systemd
  执行initrd。target所有单元，包括/etc/fstab
  从initramfs根文件系统切换到磁盘根目录
  systemd执行默认target配置，配置文件/etc/systemd/system/default.target
  systemd执行sysinit.target初始化系统及basic.target准备操作系统
  systemd启动multi-user.target下的本机与服务器服务
  systemd执行multi-user.target下的/etc/rc.d/rc.local 
  Systemd执行multi-user.target下的getty.target及登录服务 
  systemd执行graphical需要的

内核参数
sysctl -a    显示当前所有可用的内核参数
sysctl kernel.hostname      读取kernel.hostname参数
sysctl -w kernel.hostname=abc      把hostname改为abc
ls -Z              查看文件的selinux安全标签

破解口令
centos7：启动项e键修改 ---> 找到linux16内核这一项 ---> 加入rd.break,ctrl+x启动 --->
         mount -o rw,remount /sysroot 
         chroot /sysroot passwd 修改密码    
         如果selinux启动状态，则需要touch /.autorelabel exit reboot

centos6：启动时按a输入1运行级别 进入系统修改密码
判断内部外部
type COMMAND

取路径基名
[root@localhost ~]# basename /etc/sysconfig/network-scripts/
network-scripts

取路径名
[root@localhost ~]# dirname /etc/sysconfig/network-scripts/
/etc/sysconfig

查看文件状态
[root@localhost ~]# stat /etc/passwd
  File: ‘/etc/passwd’
  Size: 2241      	Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d	Inode: 33981052    Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2019-08-16 20:37:24.256360238 +0800
Modify: 2019-08-16 20:37:24.247360238 +0800
Change: 2019-08-16 20:37:24.247360238 +0800
 Birth: -

rename
rename test good cttest    #将ct开头的所有test字符替换成good

符号链接和硬链接的区别
软连接：
  1、一个符号链接指向另一个文件
  2、一个符号链接的内容是它引用文件的名称
  3、可以对目录进行
  4、可以跨分区
  5、指向的是另一个文件的路径
  6、其大小为指向的路径字符串的长度
  7、不增加或减少目标文件inode的引用计数
硬链接：
  1、创建硬链接会增加额外的记录项以引用文件
  2、对应于同一文件系统上一个物理文件
  3、每个目录引用相同的inode号，同步更新
  4、创建时链接数递增
  5、删除文件时，rm命令递减计数的连接 文件要存在，至少有一个连接数，当连接数为零0，改文件被删除
  6、不能跨分区，不能针对目录

接入新硬盘
1、fdisk格式化分区
2、mkfs.xfs制作文件系统
3、mount挂载
4、编辑fstab

标准I/O和管道
1、将/etc/issue文件中的内容转换为大写后保存至/tmp/issue.out文件中
[root@localhost ~]# cat /etc/issue|tr 'a-z' 'A-Z' > /tmp/issue.out

2、将当前系统登录用户的信息转换为大写后保存至/tmp/who.out文件中
[root@localhost ~]# who|tr 'a-z' 'A-Z' >/tmp/who.out
[root@localhost ~]# who|tr '[:lower:]' '[:upper:]' >/tmp/who.out

3、发送系统邮件
[root@localhost ~]# mail -s hello root <<EOF
> Hello,I am `whoami`.
> EOF

4、保存linux历史命令并显示命令操作时间
/etc/profile
export HISTTIMEFORMAT="%F %T"


创建⽤户gentoo，附加组为bin和root，默认shell为/bin/csh，注释信息为 "Gentoo Distribution"
[root@localhost ~]# useradd -G bin,root -c "Gentoo Distribution" -s /bin/csh gentoo

nmap扫描
[root@localhost ~]# nmap -sP 192.168.0.0/21
 
过滤出IP
[root@localhost ~]# ip a|grep -E -o "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}"


简述Raid0、Raid1、Raid5、Raid10的区别
Raid0：
  连续的分割数据并并行地读写于多个磁盘上。因此具有很高的数据传输，但没有数据冗余，没有提供数据可靠性
如果一个磁盘失效，将影响整个数据。因此RAID0不可应用于需要数据高可用性的关键应用。
最少由2块磁盘组成，每块都能存储数据。

Raid1：
  通过数据镜像实现数据冗余，在两队分离的磁盘上产生互为备份的数据。可以提高读的性能，是磁盘阵列中费用
最高的，但是提供了最好的数据可用性。当一个磁盘失效，系统可以自动地交换到镜像磁盘上。最少由2块磁盘组成，
只有一半的磁盘能存储数据。

Raid5：
  所有磁盘轮流充当校验盘，有容错能力，最多准许1块磁盘损坏。最少由3块磁盘组成，充当校验盘的不分磁盘
不能存储数据，实际容量是n-1/n

Raid10：
  先两两组成RAID1，再组合成RAID0.每组镜像最多损坏一个磁盘，有容错能力。最少由4块磁盘组成，只有一半
的磁盘能存储数据。

简述Linux下常用的文件系统有哪些，有什么区别
EXT3：
  1、最多只能支持32TB的文件系统和2TB的文件，实际只能容纳2TB的文件系统和16GB的文件
  2、EXT3目前只支持32000个子目录
  3、EXT3文件系统使用32位空间记录块数量和i-节点数量
  4、当数据写入到EXT3文件系统中时，Ext3的数据块分配器每次只能分配一个4KB的块
 
EXT4：
  1、EXT4是Linux系统下的日志文件系统，是EXT3文件系统的后继版本
  2、EXT4的文件系统容量达到1EB，而文件容量则达到16TB
  3、理论上支持无限数量的子目录
  4、EXT4文件系统使用64位空间记录块数量和i-节点数量
  5、EXT4的多块分配器支持一次调用分配多个数据库

XFS：
  1、根据所记录的日志在很短的时间内迅速恢复磁盘文件内容
  2、采用优化算法，日志记录对整体文件操作影响非常小
  3、是一个全64-bit的文件系统，他可以支持上百万T字节的存储空间
  4、能以接近裸设备I/O的性能存储数据

Read-only File system
系统没有正常关机，导致虚拟磁盘出现文件系统错误
解决方法：
  重启系统后使用root进入单用户模式，运行fsck.ext4 -y /dev/sda

Inode耗尽导致故障
find /tmp -type f -exec rm -rf {} \;
find /home -type f -size 0 -exec rm -rf {} \;

检测并修复/dev/hda5
e2fsck -p /dev/hda5

修复文件系统
fsck -y /home

/etc/fstab的字段含义
分区的标签
设备的挂载点
磁盘文件系统的格式
文件系统的参数
代表每天进行备份的操作
以fsck检验我们的系统是否为完整，1级别检验完成之后进行检验
交换机的端口模式有几种？
Access类型端口：只能属于1个VLAN，一般用于连接计算机端口
Trunk类型端口：可以允许多个VLAN通过，可以接受和发送多个VLAN报文，一般用于交换机与交换机相关的接口
Hybrid类型端口：可以允许多个VLAN通过，可以接受和发送多个VLAN报文，可以用于交换机间连接，也可以用于连接用户计算机

简述有类于无类路由选择协议的区别？
1、有类的路由不会识别子网的信息
2、无类的路由协议不会根据A B C类来识别，根据子网掩码的长度来区分网段
3、有类的路由协议只会传送网络前缀（网络地址），但是不会包含子网掩码。
4、无类路由协议传输网络前缀（网络地址）的同时也会传输子网掩码，所以它支持VLSM
从管理距离上看，无类的路由协议一般在子网中使用，所以距离较小

简述stp的作用和工作原理
STP（Spanning Tree Protocol）是生成树协议的英文缩写。该协议可应用于在网络中建立树形拓扑，消除网络中的二层环路，
并可以通过一定的方法实现路径冗余，但不是一定可以实现路径冗余。
原理：通过在交换机之间传递一种特殊的协议报文，网桥协议数据单元，来确定网络拓扑结构。

tcp协议和udp协议对比的优缺点
1、TCP面向连接；UDP是无连接，即发送数据之前不需要建立连接
2、TCP提供可靠的服务。也就是说，通过TCP连接传送的数据，无差错、不丢失、不重复，且按序到达；
   UDP尽最大努力交付，不保证可靠交付
3、UDP具有较好的实时性，工作效率比TCP高，适用于对高速传输和实时性有较高的通信或广播通信。
4、每一条TCP连接只能是点到点；UDP支持一对一，一对多，多对一和多对多的交互同喜。
5、TCP对系统资源要求较多，UDP对系统资源要求较少

UDP以其简单、传输快的优势，在越来越多场景下取代了TCP
1、网速的提升给UDP的稳定性提供可靠网络保障，丢包率很低，如果使用应用层重传，能够确保传输的可靠性
2、TCP为了实现网络通信的可靠性，使用了复杂的拥塞控制算法，建立了繁琐的握手过程，由于TCP内置的系
统协议栈中，极难对其进行改进。

简述tcp三次握手和四次挥手过程及各过程中客户端和服务器端的状态
三次握手：
客户端向服务器端发送SYN包，客户端进入SYN_SEND状态。
服务器端收到客户端发送的包返回ACK+SYN包，服务器端进入SYN_RECV状态。
客户端收到服务器端返回的包再发回ACK包，客户端进入ESTABLISHED状态，服务器端收到包也进入ESTABLISHED状态。

四次挥手：
客户端发送FIN包询问服务器端是否能断开，客户端进入FIN_WAIT_1状态。
服务器端收到客户端发送的包并返回ACK包，服务器端进入CLOSE_WAIT状态
服务器端准备好断开后，发送FIN包给客户端，服务器端进入LAST_ACK状态
客户端收到服务器端发送的包后返回ACK包，客户端进入TIME_WAIT状态，服务器端收到包后进入CLOSED状态

端口转发
iptables -t nat -A PREROUTING -d 192.168.16.1 -p tcp --dport 80 -j DNAT --todestination
192.168.16.1:8080;

判断是否有服务器被黑客入侵，如何排查？如何防护
1、检查系统组及用户。如果发现administrators组内添加了一个admin$或类似的用户
2、检查管理员账户是否存在异常登录和注销记录
3、检查服务器是否存在异常启动项

网站服务器遭受入侵
1、临时关闭网站
2、分析网站受损情况
3、检测漏洞并打补丁
4、木马病毒清除
5、经常备份数据

如何防止DDOS攻击？如被攻击需要做哪些处理
1、定期扫描，要定期扫描现有网络主节点，清查可能存在的安全漏洞，对新出现的漏洞及时清理
2、在骨干节点配置专业的抗拒服务设备。这类产品在国内应用较为广泛有绿盟、中新金盾。
3、用足够的机器承受黑客攻击。如果用户拥有足够的容量和足够的资金给黑客攻击，在它不断访问用户
、夺取用户资源之时，自己的能量也在逐渐耗失，或许未等用户被攻死，黑客已无力只招了。
4、充分利用网络设备保护网络资源。所谓网络设备是指路由器、防火墙等负载均衡设备，他们可将网络有效的保护起来
5、过滤不必要的服务和端口
6、检查访问者的来源
7、过滤所有RFC1918 IP地址。RFC1918 IP地址是内部网的IP地址，像10.0.0.0、192.168.0.0和
172.16.。。。。

显示全部进程
ps aux

查看tomcat的进程状态
ps -ef|grep tomcat

查看端口是否已经启动
netstat -anp|grep 端口

查看80端口是什么进程占用的
lsof -i:80

简述nslookup、dig、top、traceroute命令各自的作用
nslookup：测网络中DNS服务器是否能正确实现域名解析的命令行工具
dig：dns查询工具
top：实时查看系统资源占用情况
traceroute：追踪数据包在网络上的传输时的全部路径

查看cpu、内存、io使用情况
top dstat sar
free
iostat
top动态实时显示系统性能
vmstat静态显示系统性能

统计establish状态的连接数
#netstat -an|grep ESTABLISHED|wc -l

常用的查看服务器性能命令
top htop iotop uptime free iostat dstat vmstat mpstat glances

 crontab任务计划
 周三 7-9点每5分钟执行一次a.sh
crontab -e
*/5 7-9 * * 3 /bin/bash a.sh

每周一、三、五凌晨1点自动重启
crontab -e
00 1 * * 1,3,5 /sbin/reboot

每天早上6点到12点，每隔2小时执行一次/usr/bin/httpd.sh
crontab -e
00 6-12/2 * * * /usr/bin/httpd.sh

/var/spoll/cron/root   #root制定完计划任务后会生成此文件，备份此文件就是备份定时任务

每周六 23点同步时间
crontab -e
00 23 * * 6 ntpdate ntp3.aliyun.com

glances命令获取远程主机的系统信息
1、在远程主机上开启glances代理   host:192.168.3.10
[root@localhost ~]# glances -sB 192.168.3.20

2、在本地监控远程主机的系统信息  host:192.168.3.20
[root@localhost ~]# glances -C 192.168.3.10

每周工作日1:30，执行
30 1 * * 1-5 /root/

每两小时执行
0 */2 * * * /bin/

工作日时间，每十分钟执行一次
*/10 * * * 1-5 /bin/bash
判断/var/⽬录下所有⽂件的类型
root@centos7 ~]# vim filetype.sh
#!/bin/sh
for i in $(find /var) ;do
        if [ -b $i ];then
                echo "$i是块设备"
        elif [ -c $i ] ;then
                echo "$i是字符设备"
        elif [ -f $i ];then
                echo "$i是普通文件"
        elif [ -h $i ];then
                echo "$i是符号链接文件"
        elif [ -p $i ];then
                echo "$i是管道文件"
        elif [ -S $i ];then
                echo "$i是套接字文件"
        elif [ -d $i ];then
                echo "$i是目录文件"
        else
                echo "文件或目录不存在"
        fi
done
exit 0

添加10个用户user1-user10，密码为8位随机字符
[root@centos7 ~]# vim useradd.sh
#!/bin/bash
for n in $(seq 1 10);do
        name=user$n
        useradd "$name"
        echo "$(tr -cd [[:alnum:]\!_#@] < /dev/urandom |head -c 8)"|passwd --stdin $name &> /dev/null
        echo "user creation"
done
exit 0

⽬录下分别有多个以K开头和以S开头的⽂件；分别读取每个⽂件， 
以K开头的输出为⽂件加stop，以S开头的输出为⽂件名加start，
[root@centos7 ~]# vim k_s.sh
#!/bin/sh
for i in `ls /data/` ;do
        [[ "$i" =~ ^k.* ]] && echo "$i stop"
        [[ "$i" =~ ^s.* ]] && echo "$i start"
done

#!/bin/sh
for i in `ls /data/|egrep -i "^k.*"`;do
        echo "$i stop"
done

for j in `ls /data/|egrep -i "^s.*"`;do
        echo "$j start"
done

编写脚本，提示输入正整数n的值，计算总和
[root@centos7 ~]# vim numbersum.sh
#!/bin/bash
read -p "please input a positive integer:" n
i=1
sum=0
for i in `seq 1 $n`;do
        let sum+=i
done
        echo "sum is $sum"
lsmod         查看已加载的模块
modinfo       显示kernel模块的信息
modprobe      加载某个模块
biosdecode    BIOS版本
dmidecode | grep 'Product Name'        #查看服务器型号
dmidecode |grep 'Serial Number'        #查看主板的序列号
ethtool -i eth0                        #网卡驱动版本

Linux运行级别
0： 系统停机（关机）模式，系统默认运行级别不能设置为0，否则不能正常启动，一开机就自动关机。
1：单用户模式，root权限，用于系统维护，禁止远程登陆，就像Windows下的安全模式登录。
2：多用户模式，没有NFS网络支持。
3：完整的多用户文本模式，有NFS，登陆后进入控制台命令行模式。
4：系统未使用，保留一般不用，在一些特殊情况下可以用它来做一些事情。例如在笔记本电脑的电池
用尽时，可以切换到这个模式来做一些设置。
5：图形化模式，登陆后进入图形GUI模式或GNOME、KDE图形化界面，如X Window系统。
6：重启模式，默认运行级别不能设为6，否则不能正常启动，就会一直开机重启开机重启。

MBR
MBR(Master Boot Recorder)，我们称之为主引导记录。BIOS是怎样寻找可启动设备的呢？我们知
道在分区时，硬盘的第一块扇区512个字节就是存放的MBR，如果该设备是可启动设备，那么该扇区的
最后两个字节肯定是55/AA，所以此时在寻找可启动设备时，如果发现该设备的最后两个字节是这个，
那么该设备就是可启动设备。BIOS在找到可启动设备以后就会执行其引导代码，因为MBR占据了第一块
扇区的512字节，分区表占用了 16*4 =64字节，再加上最后两个标志字节，所以MBR的引导代码就是
MBR的前446个字节，当然这446个字节太小了，并不能完成整个操作系统的引导程序，所以这446个字
节里面可能存放的就是启动引导程序的一些代码。

GRUB
GRUB是引导加载程序，将引导操作系统，启动引导器是计算机启动过程中运行的第一个真正的软件，
计算机启动时在通过BISO自检后读取并运行引导介质上最前面的扇区即硬盘主引导扇区(MBR)中的启
动引导器程序， 这里的扇区中：MBR占据了第一块扇区的512字节，但其实际只占用了其中的446个字
节，另外的64个字节交给了DPT（Disk Partition Table硬盘分区表），最后两个字节“55，AA”是
分区的结束标志。启动引导器在负责加载启动硬盘分区中的操作系统。如果启动引导器不能正常工
作，将导致操作系统不能正常启动，从而整个计算机瘫痪。通常，每个操作系统在安装过程中都要将
自带的启动器写在硬盘(MBR)，以便能过进行自身的引导。
```