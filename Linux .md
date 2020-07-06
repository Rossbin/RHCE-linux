# 存储结构

* 文件系统层次化标准（FHS，Filesystem Hierarchy Standard）
* Linux系统中最常见的Ext3、Ext4与XFS文件系统

| 目录名称    | 应放置文件的内容                                             |
| ----------- | :----------------------------------------------------------- |
| /boot       | 开机所需文件—内核、开机菜单以及所需配置文件等                |
| /dev        | 以文件形式存放任何设备与接口                                 |
| /etc        | 配置文件                                                     |
| /home       | 用户主目录                                                   |
| /bin        | 存放单用户模式下还可以操作的[命令](https://www.linuxcool.com/) |
| /lib        | 开机时用到的函数库，以及/bin与/sbin下面的命令要调用的函数    |
| /sbin       | 开机过程中需要的命令                                         |
| /media      | 用于挂载设备文件的目录                                       |
| /opt        | 放置第三方的软件                                             |
| /root       | 系统管理员的家目录                                           |
| /srv        | 一些网络服务的数据文件目录                                   |
| /tmp        | 任何人均可使用的“共享”临时目录                               |
| /proc       | 虚拟文件系统，例如系统内核、进程、外部设备及网络状态等       |
| /usr/local  | 用户自行安装的软件                                           |
| /usr/sbin   | Linux系统开机时不会使用到的软件/命令/[脚本](https://www.linuxcool.com/) |
| /usr/share  | 帮助与说明文件，也可放置共享文件                             |
| /var        | 主要存放经常变化的文件，如日志                               |
| /lost+found | 当文件系统发生错误时，将一些丢失的文件片段存放在这里         |







# RAID(磁盘冗余阵列)

### raid 10创建:

mdadm -Cv /dev/md0 -n 4 -l 10 /dev/sd[b-e]
查看raid:
mdadm -Q /dev/md0
查看细节：
mdadm -D /dev/md0
格式化为xfs:
mkfs.xfs /dev/md0
挂在到/haha目录:
mount /dev/md0 /haha
查看挂在信息:
df -h
永久挂载:
vim /etc/fstab
/dev/md0 /haha xfs default 0 0
将/etc目录下的所有递归复制到当前目录:
cp -rf /etc/* ./
追加备份盘:
umount /haha
mdadm /dev/md0 -a /dev/sdb
建立一个热备盘：
mdadm -Cv /dev/md0 -n 3 -l 5 -x 1 /dev/sd[b-e]



mdadm -Cv /dev/md0 -n 4 -l 10 /dev/sd[b-e]
mdadm -Q /dev/md0
mdadm -D /dev/md0
mkfs.xfs /dev/md0
mount /dev/md0 /haha
df -haha
vim /etc/fstab
/dev/md0 /haha xfs default 0 0 
cp -rf /etc/* ./

-----------------------------------------------------------



# LVM（逻辑卷管理）

### LVM（考扩大操作）：

pvcreate /dev/sdb
pvcreate /dev/sdc
vgcreate xiaoguo /dev/sdb /dev/sdc
lvcreate -n haha -l (小写为个数，大写为总大小) 100 xioaguo 
mkfs.ext4 /dev/xioaguo/haha 
mkdir /haha
vim /etc/fstab
/dev/xiaoguo/haha /haha ext4 default 0 0
mount -a
df -h

### 扩容：

umount /haha
lvextend -L 800M /dev/xiaoguo/haha
e2fsck -f /dev/xiaoguo/haha  (检查数据是否丢失)
resize2fs /dev/xiaoguo/haha  (通知文件系统)
mount -a
df -h

### 缩小：

umount /haha
e2fsck -f /dev/xiaoguo/haha
resize2fs /dev/xiaoguo/haha 300M
lvreduce -L 300M /dev/xiaoguo/haha 

### 快照(单次有效)：

cp -rf /etc/* ./
lvcreate -L 300M -s -n SNAP /dev/xiaoguo/haha
lvdisplay 
rm -rf *
cd ~
umount /haha
lvconvert --merge /dev/xiaoguo/SNAP

### 卸载LVM：

cd ~
umount /haha
vim /etc/fstab (删除文件中的挂载信息)
lvremove /dev/xiaoguo/haha 
vgremove xioaguo
pvremove /dev/sdb
pvremove /dev/sdc

--------------------------------------------------------

# 配置网络
* 1. vim /etc/sysconfig/network-scripts/ifcfg-eno16777728
     IPADDR0=192.168.74.5
     systemctl restart network

* 2. nmtui

* 3. nm-connection-editor

# 防火墙
### iptables(real 7之前版本)
* 1. ACCEPT              接受

* 2. REJECT               拒绝

* 3. DROP                 丢包

* 4. LOG                    日志

```
1.iptables -L         显示当前已有的策略
2.iptables -F         清空现有的已有的策略
3.iptables -P INPUT DROP   限制所有用户为丢包
4.iptables -I INPUT -s 192.168.74.1 -p icmp -j ACCEPT  允许该地址ip对主机进行icmp
5.iptables -I INPUT -s 192.168.74.1 -p tcp --dport 22 -j ACCEPT 允许该地址对主机进行ssh访问
5.iptables -P INPUT ACCEPT  允许所有
6. service iptables save      保存iptables的状态

```
### firewall(real 7)
* 1、firewall-cmd

* 2、firwall-config(图形化界面)

#### firewall-cmd
```
1.firewall-cmd --get-default-zone    查看默认的区域（模板）
2.firewall-cmd --get-zone-of-interface=eno16777728   (查看网卡接口的区域)
3.firewall-cmd --set-default-zone=drop   设置当前区域为丢包状态
4.firewall-cmd --zone=public --query-service=http    查询是否有HTTP服务
5.firewall-cmd --zone=public --add-service=http       将HTTP服务加入进去
6.firewall-cmd --permanent --zone=public --query-service=https   查询永久策略是否允许https服务
7.firewall-cmd --permanent --zone=public --add-service=https  设置为永久生效，需重启
8.firewall-cmd --zone=public --add-port=8000-8888/tcp  放行端口号
9.firewall-cmd --list-ports                            查询端口号
10.firewall-cmd --zone=public --remove-service=https   禁止该服务
11.firewall-cmd --permanent --zone=public --add-forward-port=port=888:proto=tcp:toport=22:toaddr=192.168.74.5   转发端口修改
12.firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.74.0/24" service name="ssh" reject"     配置一条富规则，禁止ssh



runtime      暂时模式
premanent    永久名称（需要reboot）

`firewall-cmd`
`--panic-on   紧急模式(断截所有连接)`        
`--reload     立即生效参数`

```


+ ping 

  - 1. ping x.x.x.x -t  (持续ping)

  - 2. ping -c n(次数) x.x.x.x   (n为具体次数)

#### firewall-config

#### 应用层过滤（TCP Wrappers）
`/etc/hosts.allow`
* 格式:
sshd : 192.168.74.5 (允许的源地址) 

`/etc/hosts.deny`

* 格式:
  sshd(服务名) : 192.168.74. (被限制的网段)  

  

* 优先级：
`iptales (内核级别) > firewall > tcp wrappers`





# 配置网卡服务

### 创建网络会话(nmcli)

* 创建两个网络会话
  * company  192.168.10.88/24
  * home 192.168.74.0/24 dhcp

```
* nmcli connection add con-name company ifname eno16777728 autoconnect no type ethernet ip4 192.168.10.88/24 gw4 192.168.10.1   #添加公司网络会话
* nmcli connection add con-name home type ethernet ifname eno16777728   #添加家庭网络会话

* nmcli connection show      #查看网络会话
* nmcli connection up home   #启用home网络会话
```



### 绑定两块网卡（类似于RAID）

* 打开虚拟机编辑设置添加一块新网卡（仅主机模式）

* vim /etc/sysconfig/network-scripts/ifcfg-eno16777728   (编辑被绑定的网卡信息)

  * ```
    TYPE(上网类型)=Ethernet
    BOOTPROTO(分配协议)=none  (有static dhcp none(默认))
    ONBOOT(重启有效)=yes
    USERCTL(用户命令行管理)=no
    DEVICE(设备名)=eno16777728
    MASTER(主设备名称)=bond0
    SLAVE(从设备)=yes
    ```

* vim /etc/sysconfig/network-scripts/ifcfg-eno33554968   (编辑被绑定的网卡信息)

  * ```
    TYPE=Ethernet
    BOOTPROTO=none
    ONBOOT=yes
    USERCTL=no
    DEVICE=eno33554968
    MASTER=bond0
    SLAVE=yes
    ```

* vim /etc/sysconfig/network-scripts/ifcfg-bond0    (编辑绑定后的网卡信息)

  * ```
    TYPE=Ethernet
    BOOTPROTO=none
    ONBOOT=yes
    USERCTL=no
    DEVICE=bond0
    IPADDR=192.168.10.88
    PREFIX=24
    DNS=192.168.10.1
    NM_CONTROLLED=no
    ```

* vim /etc/modprobe.d/bond.conf     (创建网卡绑定的驱动文件，让Linux内核支持网卡绑定驱动)

  * ```
    alias bond0 bonding  (已绑定的网卡)
    options bond0 miimon(延迟毫秒)=100 mode=6
    ```

  * mode0 (平衡负载模式)： 两块网卡均工作，且自动备援，但需要在与服务器本地网卡相连的交换机上进行端口聚合来支持绑定技术

  * mode1 (自动备援模式)：只有一块网卡工作，但它故障后另一块会自动替换

  * mode6 (平衡负载模式)：两块网卡均工作，且自动备援，无需交换机提供辅助支持

* systemctl restart network   (重启服务)







# 远程控制服务

### 配置文件

* 主配置文件：最重要的配置参数
  * `/etc/服务名称/服务名称.conf/`
* 普通配置文件：主要被调用，常规参数

#### SSH的Windows工具

* xshell
* putty
* xmanager
* securecrt
* SSH Secure Shell



### 配置ssd服务

* vim /etc/ssh/sshd_config

  * ```
    若要修改sshd的配置就将文件中的#号去出就可修改
    例：
    port=22 可以改为 port=3389
    ```

* setenforce 0  (关闭SELinux)

* iptables -F  (iptables防火墙规则清零，防止干扰实验)

* systemctl enable sshd  (加入开机启动)

* systemctl restart sshd  (重启服务)



###  安全密钥认证

#### 客户端上生成，传送到服务器

##### 密钥生成

* ssh-keygen    (生成密钥文件)

  * (回车)
  * (回车)
  * (回车)

* cd ~

* cat .ssh/id_rsa.pub    (公钥信息)

* cat .ssh/id_rsa    (私钥信息,2048bit)

* ssh-copy-id 192.168.10.10   (传到服务器中)

  * ```
    服务器：192.168.10.10
    客户端：192.168.10.20
    ```

  * yes

  * 输入服务器端root用户的密码

* ssh 192.168.10.10    (连接服务器)

* touch xiaoli

* vim xiaoli

  * ```
    chuan shu wen jian zhong de zi fu 
    ```



#### 服务器端

##### 密钥设置

* cat .ssh/authorized_keys

* vim /etc/ssh/sshd_config   (编辑ssh配置信息)

  * 78行：PasswordAuthentication no   (去除密码认证)
  
* systemctl restart sshd   (重启服务)
  
##### 传输文件（scp）

* scp xiaoguo 192.168.10.20:/root   (`scp 目标文件 用户名@(可以不指定用户)对方IP:路径`)
  * yes
  * (输入对方服务器的密码)

  ##### 下载文件  (scp)

* scp 192.168.10.20:/root/xiaoli /root   (`scp 用户名@(可以不指定用户)对方IP:文件名 本地路径`)
  
  * 输入对方服务器的密码



  

  # 不间断会话服务

### yum仓库配置

###### 1. 搞定你的镜像 /dev/cdrom /media/cdrom

###### 2. 搞定yum仓库的配置文件

###### 3. 执行yum install

###### 1. 

* mkdir -p /media/cdrom      (创建cdrom目录)

* mount /dev/cdrom /media/cdrom   (将cdrom目录挂载到media下)

* ls /media/cdrom     (查看cdrom内的内容)

* vim /etc/fstab     (编辑，永久生效)

  * ```
    /dev/cdrom /media/cdrom iso9660 default 0 0
    ```

* mount -a     (自动挂载)

  ###### 2. 

* vim /etc/yum.repos.d/xiaoguo.repo   (创建yum仓库的配置)

  * ```
    (模板)
    [] (yum仓库的标识符)
    name=    (文件命名)
    baseurl= file:// 、http://   (挂载路径)
    enabled=    (是否开启仓库)
    gpgcheck=	(是否校验)
    ```

  * ```
    [haha]
    name=hoho
    baseurl=file:///media/cdrom
    enabled=1
    gpgcheck=0
    ```

###### 3. 

* yum install screen    (安装screen软件)

  * y

  



### 管理远程会话  (screen)

* ssh 192.168.10.10 
  * 输入密码
* screen -S haha      (新建会话)
  * (在此界面进行的所有操作会保存在本地的服务器中)
  * 在意外中断连接时可以被恢复
* screen -ls         (查看之前保存的会话)
* screen -r haha      (恢复保存的会话)
  * (进入了之前中断·了的会话，可以继续操作)

###### 写文件时被中断：

* screen vim xiaoshuo

  * ``` 
    fjaeiffweoafnehwioghwei    (未保存)
    ```

* screen -ls

* screen -r 会话编号   （恢复到中断前）

  

###### 屏幕实时共享

* 两个终端同时登录一个服务器  （ssh）
  * 服务器A
    * screen -S haha
  * 服务器B
    * screen -x (加入编号，可以先回车查看编号)
  * 会话同步环境完成





# Apache部署静态网站

#### Apache  (美国，老牌)

#### Nginx  (俄罗斯研发,新版)

### 网站服务程序httpd

* 安装
  * yum install httpd    (daemon,后面的d为守护进程的意思)
    * y
  * systemctl restart httpd
  * systemctl enable httpd
  * 访问192.168.10.10，但是网站权限不足，网站中没有文件
  * cd /var/www/html  (默认目录)
  * vim index.html    （首页文件）
    * hahah,the first web site
* 修改权限，移动默认目录
  * mkdir /home/wwwroot
  * vim /etc/httpd/conf/httpd.conf
    * 在120行是存放网站的路径
      * 修改为 "/home/wwwroot"
    * 124行也得修改为："/home/wwwroot"
  * systemctl restart httpd
  * systemctl enable httpd
  * vim index.html
    * xie ru nei rong
  * 访问192.168.10.10
    * SELinux权限不足



### SELinux安全子系统

##### SELinux 域   (限制软件、服务程序的行为)

##### SELinux 安全上下文    (限制文件的访问人员)

* ls -dlZ     /var/www/html   (查看安全上下文的值)
* ls -dlZ     /hom/wwwroot
* setenforce 0      (暂时关闭SELinux，一般用来排除故障)
* setenforce 1       (暂时开启SELinux)
* semanage fcontext  -a  -t (安全上下文的值) /home/wwwroot        `设置SELinux,-a表示设置，-t表示设置的值`
*  semanage fcontext -a -t  httpd_sys_content_t(查到的上下文的值)  /home/wwwroot/*   `设置SELinux 网站下的所有文件`
* restorecon -Rv /home/wwwroot    `让已设置的SELinux的值生效`





### 网站用户的功能

##### （为每一位用户提供独立的网站主页）

* vim /etc/http/conf.d/userdir.conf     `编辑个人用户主页功能`
  * UserDir xxxx 修改为  pubic_html
* systemctl restart httpd
* systemctl enable httpd
* su - linuxprobe
  * mkdir -p public_html
  * cd public_html/
  * echo "linuxprobe" > index.html
  * cd ..
  * chmod -Rf 755   /home/linuxprobe    `权限修改`
  * exit
* 访问192.168.10.10/~linuxprobe

 

##### getsebool、setsebool     `设置SELinux域的值`

* getsebool  -a     `查看selinux域`

* getsebool  -a  |  grep  http    (查看值跟网站相关的条目)

* setsebool   -P  httpd_enable_homedirs=on     (-P表示永久生效,开启一条条目)  

* vim /etc/httpd/conf.d/userdir.conf     (开启加密功能)

  * ```
    <Directory "/home/*/public_html">
    allowoverride all   (伪静态技术)
    authuserfile "/etc/httpd/passwd"   (认证文件)
    authname "hahahaah"       (用户提示信息)
    authtype basic     (认证方式)
    require user linuxprobe 
    </Directory>
    ```

* htpasswd -c /etc/httpd/passwd  linuxprobe     (-c表示创建,linuxprobe表示验证用户名)

  * 输入密码
  
* systemctl restart httpd

* 访问192.168.10.10/~linuxprobe





### 虚拟主机功能

#### 基于IP 地址

* mkdir -p /home/wwwroot

*  mkdir -p /home/wwwroot/10

* mkdir -p /home/wwwroot/20

* mkdir -p /home/wwwroot/30

* vim /etc/sysconfig/network-scripts/ifcfg-eno16777728

  * ```
    IPADDR0=192.168.10.10
    IPADDR1=192.168.10.20
    IPADDR2=192.168.10.30
    ```

* systemctl restart network

* ping 192.168.10.10   (测试主机地址的连通性)

* ping 192.168.10.20

* ping 192.168.10.30

* cd /home/wwwroot     （添加网站的数据）

  * mkdir -p 10 20 30
  * echo "<h1>101010101010</h1>" > 10/index.html
  * echo "<h1>202020202020</h1>" > 20/index.html
  * echo "<h1>303030303030</h1>" > 30/index.html

* vim /etc/httpd/conf/httpd.conf   (修改主配置文件)

  * :113

  * ```
    <virtualhost 192.168.10.10>
    documentroot /home/wwwroot/10
    servername www.linuxprobe.com
    <directory /home/wwwroot/10>
    allowoverride none
    require all granted
    </directory>
    </virtualhost>
    <virtualhost 192.168.10.20>
    documentroot /home/wwwroot/20
    servername www.linuxprobe.com
    <directory /home/wwwroot/20>
    allowoverride none
    require all granted
    </directory>
    </virtualhost>
    <virtualhost 192.168.10.30>
    documentroot /home/wwwroot/30
    servername www.linuxprobe.com
    <directory /home/wwwroot/30>
    allowoverride none
    require all granted
    </directory>
    </virtualhost>
    ```

* systemctl status httpd  (查看http服务的状态)
* (设置SELinux安全上下文的值，给予网站获取文件的权限)
* ls -ldZ /var/www/html
* ls -ldZ /home/wwwroot
* semanage fcontext -a -t [安全上下文的值] /home/wwwroot
* semanage fcontext -a -t [安全上下文的值] /home/wwwroot/10
* semanage fcontext -a -t [安全上下文的值] /home/wwwroot/10/*
* semanage fcontext -a -t [安全上下文的值] /home/wwwroot/20
* semanage fcontext -a -t [安全上下文的值] /home/wwwroot/20/*
* semanage fcontext -a -t [安全上下文的值] /home/wwwroot/30
* semanage fcontext -a -t [安全上下文的值] /home/wwwroot/30/*
* restorecon -Rv /home/wwwroot   (使修改的SELinux立即生效)
* 分别访问这三个网站 192.168.10.10/20/30   (访问成功)





#### 基于主机域名  （实验需要还原虚拟机）

* yum install -y httpd  (前提为搞定了 yum 仓库)

* vim /etc/hosts     (强制做域名和IP地址解析对应)

  * ```
    (在最后一行写入)
    192.168.10.10 www.linuxprobe.com bbs.linuxprobe.com tech.linuxprobe.com
    ```

* ping www.linuxprobe.com   (测试域名解析)

* cd /home/wwwroot

  * mkdir -p /home/wwwroot/www
  * mkdir -p /home/wwwroot/bbs
  * mkdir -p /home/wwwroot/tech

* cd /home/wwwroot

  * echo "<h1>wwwwwwwwww</h1>" > www/index.html
  * echo "<h1>bbsbbsbbsbbsbb</h1>" > /bbs/index.html
  * echo "<h1>techtechtechtech</h1>" > /tech/index.html

* vim /etc/httpd/conf/httpd.conf     (修改网站的主配置文件)

  * ```
    :113
    <virtualhost 192.168.10.10>
    documentroot /home/wwwroot/www
    servername www.linuxprobe.com
    <directory /home/wwwroot/www>
    allowoverride none
    require all granted
    </directory>
    </virtualhost>
    <virtualhost 192.168.10.10>
    documentroot /home/wwwroot/bbs
    servername bbs.linuxprobe.com
    <directory /home/wwwroot/bbs>
    allowoverride none
    require all granted
    </directory>
    </virtualhost>
    <virtualhost 192.168.10.10>
    documentroot /home/wwwroot/tech
    servername tech.linuxprobe.com
    <directory /home/wwwroot/tech>
    allowoverride none
    require all granted
    </directory>
    </virtualhost>
    
    ```

* systemctl restart httpd    (重启服务)

* systemctl enable httpd

* ls -ldZ /home/wwwroot

* ls -ldZ /var/www/html

* semanage fcontext -a -t [SELinux安全上下文的值] /home/wwwroot

* semanage fcontext -a -t [SELinux安全上下文的值] /home/wwwroot/www

* semanage fcontext -a -t [SELinux安全上下文的值] /home/wwwroot/www/*

* semanage fcontext -a -t [SELinux安全上下文的值] /home/wwwroot/bbs

* semanage fcontext -a -t [SELinux安全上下文的值] /home/wwwroot/bbs/*

* semanage fcontext -a -t [SELinux安全上下文的值] /home/wwwroot/tech

* semanage fcontext -a -t [SELinux安全上下文的值] /home/wwwroot/tech/*

* restorecon -Rv /home/wwwroot   (让SELinux立即生效)

* 以域名形式访问网站  (成功)



#### 基于端口号  (还原镜像)

* yum install -y httpd

* mkdir -p /home/wwwrooot/6111

* mkdir -p /home/wwwrooot/6222

* cd /home/wwwroot

  * echo "<h1>611111111111</h1>" > 6111/index.html
  * echo "<h1>622222222222</h1>" > 6222/index.html

* vim /etc/httpd/conf/httpd.conf

  * ```
    (开启监听端口号)
    Listen 6111
    Listen 6222
    ```

  * ```
    (配置虚拟主机)
    <virtualhost 192.168.10.10:6111>
    documentroot /home/wwwroot/6111
    servername www.linuxprobe.com
    <directory /home/wwwroot/6111>
    allowoverride none
    require all granted
    </directory>
    </virtualhost>
    <virtualhost 192.168.10.10:6222>
    documentroot /home/wwwroot/6222
    servername www.linuxprobe.com
    <directory /home/wwwroot/6222>
    allowoverride none
    require all granted
    </directory>
    </virtualhost>
    ```

* ls -ldZ /var/www/html

* ls -ldZ /home/wwwroot

* semanage fcontext -a -t [安全上下文的值] /home/wwwroot

* semanage fcontext -a -t [安全上下文的值] /home/wwwroot/6111

* semanage fcontext -a -t [安全上下文的值] /home/wwwroot/6111/*

* semanage fcontext -a -t [安全上下文的值] /home/wwwroot/6222

* semanage fcontext -a -t [安全上下文的值] /home/wwwroot/6222/*

* restorecon -Rv /home/wwwroot   (立即生效SELinux上下文)

* semanage port -l   (查询selinux支持的端口号)	

  * semanage port -l | grep http   (过滤网站服务程序允许的端口号)
  * semanage port -a -t http_port_t -p tcp 6111
  * semanage port -a -t http_port_t -p tcp 6222

* systemctl restart httpd

* systemctl enable httpd









#### Apache的访问控制列表   (acl,还原虚拟机)

* yum install -y httpd    (搞定yum仓库)

* mkdir /var/www/html/server

* vim /etc/httpd/conf/httpd.conf

  * ```
    <directory /var/www/html/server>
    setenvif User-Agent "Firefox" ff=1
    order allow,deny
    allow from env=ff
    </directory>
    ```

* systemctl restart httpd

* systemctl enable httpd

* iptables -F        (清空防火墙)

* service iptables save     （可以在原机上的浏览器上访问）

* 访问192.168.10.10/server    (成功)

* echo "<h1>i am a server</h1>" > /var/www/html/server/index.html

















# Vsftpd 服务传输文件

### FTP    20 21     (20为传输，21为控制文件)

### ssh  22

### telnet 23

### TFTP 69

### vsftpd 表示提供一个安全的文件传输进程

* 三个模式的访问路径

* ```
  匿名用户      /var/ftp
  本地用户      对应用户的家目录
  虚拟用户      映射用户的家目录
  ```

* yum install vsftpd    （提前搞定你的yum仓库）

* cd /etc/vsftpd     #进入vsftpd的配置目录

  * vim vsftpd.conf

    * ```
      :127
      ```

* mv vsftpd.conf vsftpd_bak

* grep "#" vsftpd_bak      (过滤出有#号信息内容)

* grep -v "#" vsftp_bak      （-v反选，过滤出不带#号的信息内容）

* grep -v "#" vsftp_bak > vsftp.conf     (重新写回到原配置文件中)

* vim vsftpd.conf

  * ```
    anonymous_enable=YES  (是否允许匿名用户直接访问)
    local_enable=YES   (是否允许本地账户访问)
    write_enble=YES    (是否允许本地账户写入)
    local_umask=022    (写入后，文件权限的反掩码)
    （umask 022）
    （文件：666-umask=）
    （目录：777-umask=）
    dirmessage_enable=YES     (进入目录后的提示信息)
    xferlog_enable=YES     (是否开启日志功能	)
    connect_from_port_20=YES   (是否同过20端口传输文件)
    xferlog_std_format=YES    (是否以格式化的形式写入日志)
    listen=NO     (是否以独立的形式运行vsftpd)
    listen_ipv6=YES   (是否支持ipv6)
    pam_service_name=vsftpd   (定义pam模块的使用mingc)
    （pam模块为可插拔模块）
    userlist_enable=YES   (黑名单)
    （ftpusers、user_list 用户出现有这两个文件中，默认不能登录至服务器中）
    tcp_wrappers=YES    (是否开启这个防火墙限制)
    ```

  * ```
    （匿名用户访问，开始配置）
    anonymous_enable=YES
    anon_umask=022            
    anon_upload_enable=YES     (是否允许用户上传文件)
    anon_mkdir_write_enable=YES   (写入)
    anon_other_write_enable=YES  (重命名、删除等权限)
    ...
    ```

* systemctl restart vsftpd

* systemctl enable vsftpd

* yum install -y ftp

* Windows 远程访问工具flashfxp



##### 匿名访问模式：

* ftp 192.168..10.10
  * anonymous
  * (回车)
  * (访问的目录为： /var/ftp/pub)
    * cd pub
    * mkdir haha
* chmod -Rv  777 /var/ftp/pub    (设置权限)
* getsebool  -a | grep ftp
* setseboot -P ftpd_full_access=on    (开启selinux中ftpd)
* ftp 192.168.10.10
  * mkdir haha
  * rmdir haha



##### 本地账户模式:

* (本地账户登录)

* vim /etc/vsftpd/vsftpd.conf

  * ```
    删除匿名用户的3条  （以anon开头的）
    ```

* systemctl restart vsftpd

* ftp 192.168.10.10

  * root  (失败，在黑名单中有记录)

* cd /etc/vsftpd

  * vim ftpusers
    * (删除root)
  * vim user_list
    * （删除root）

* systemctl restart vsftpd

* ftp 192.168.10.10

  * root
  * 密码
  * mkdir  haha
  * rmdir   haha





##### 虚拟用户模式(最安全)：

* pam模块 ------- 可插拔式认证模块

* cd /etc/vsftpd     (过滤成可是用的配置)

  * mv vsftpd.conf vsftpd_bak

  * grep -v "#" vsftpd_bak

  * grep -v "#" vsftpd_bak > vsftpd.conf

  * vim vsftpd.conf

  * vim vuser.list    (账户和密码的信息)

    * ```
      (单数行为账号，偶数行为密码)
      zhangsan
      redhat
      lisi
      redhat
      ```

  * db_load -T -t hash -f vuser.list vuser.db    (加密文件)

  * file vuser.list

  * file vuser.db

  * rm vuser.list   (删除明文文件)

  * useradd -d /var/ftproot -s /sbin/nologin virtual   (映射的虚拟用户)

  * id virtual

  * chmod -Rf 755 /var/ftproot   (设置虚拟账户的权限)

###### pam模块

* （是为了让用户调取pam模块进行自动验证）

* vim /etc/pam.d/vsftpd. vu   （建立pam模块）

  * ```
    auth required pam_userdb.so db=/etc/vsftpd/vuser
    account required pam_userdb.so db=/etc/vsftpd/vuser
    ```

* cd /etc/vsftpd

* vim vsftpd.conf     (修改主配置文件中的pam)

  * ```
    pam_service_name=vsftpd.vu
    ```

  * ```
    guest_enable=YES  (虚拟用户开启)
    guest_username=virtual    (映射的虚拟用户)
    allow_writeable_chroot=YES    
    (writeable----可写操作)
    (chroot----牢笼机制)
    user_config_dir=/etc/vsftpd/vusers_dir  (权限列表目录)
    ```

* mkdir -p /etc/vsftpd/vusers_dir 

* cd /etc/vsftpd/vusers_dir 

  * touch zhangsan

  * touch lisi

  * vim zhangsan 

    * ```
      anon_upload_enable=YES
      anon_mkdir_write_enable=YES
      anon_other_write_enable=YES
      ```

* getsebool -a | grep ftp

* setsebool -P ftpd_full_access=on     (-P为永久生效)

* systemctl restart vsftpd

* systemctl enable vsftpd

* yum install -y ftp

* ftp 192.168.10.10

  * zhangsan 
  * redhat
  * mkdir haha
  * rename haha hoho
  * rmdir hoho
  * exit

* ftp 192.168.10.10

  * lisi
  * redhat
  * ls
  * mkdir haha (失败,因为没有足够的权限)



### TFTP (简单文件传输协议)

#### 采用UDP 端口号为69

* yum install tftp-server tftp   (安装)
###### xinetd (服务程序)

```
在RHEL 7系统中，TFTP服务是使用xinetd服务程序来管理的。xinetd服务可以用来管理多种轻量级的网络服务，而且具有强大的日志功能。在安装TFTP软件包后，还需要在xinetd服务程序中将其开启，把默认的禁用（disable）参数修改为no
```

* vim /etc/xinetd.d/tftp
  * disable = no
* systemctl restart xinetd
* systemctl enable xinetd
* cd /var/lib/tftpboot/
  * vim xiaoguo
    * (写入信息，模拟文件)

###### 用户端（另起一个终端）

* tftp 192.168.10.10
  * get xiaoguo
  * (获取成功)
  * q (退出)











# Samba或NFS实现文件共享

### Samba:

```
Linux  -  Linux
Linux  -  Windows
```

* yum install -y samba    (安装服务，前提搞定镜像)

* cd /etc/samba

  * vim smb.conf   (主配置文件)

* mv smb.conf  smb.bak

* ```
  grep -v "#" smb.bak | grep -v ";" | grep -v "^$" > smb.conf 
  ("^$"表示空行)
  (将带#、带;和空行都过滤掉)
  ```

* vim smb.conf

  * ```
    14~20是关于打印机的信息（可以删除）
    10~13是有关共享的家目录（可以删除）
    8~9  是有关打印机的信息（可以删除）
    
    [global]      (共享名称)
    workgroup = MYGROUP    (网络邻居里的共享组)
    server string = Samba Server Version %v   (samba服务器的标识，显示版本信息)
    log file = /var/log/samba/log.%m   (系统日志保存的路径)
    
    (%m表示以月份保存)
    
    max log size = 50    (系统日志保存的大小)
    security = user    (验证模式)
    (user share server domain)
    passdb backend = tdbsam    
    ```
    
  * ```
    [global]
    workgroup = MYGROUP
    server string = Samba Server Version %v
    log file = /var/log/samba/log.%m
    max log size = 50
    security = user
    passdb backend = tdbsam
    
    [rick]    (表示名)  
    comment =  zhongyaodewenjina  (描述)
    path = /rick  (共享的文件夹的路径)
    public = no   (是否公开访问该目录)
    writable  = yes   
    ```
  
* mkdir -p /rick  (新建目录)

* chown -Rf linuxprobe:linuxprobe /rick

* chmod -Rf 777 /rick

* id linuxprobe

* pdbedit -a -u linuxprobe (-a 用户修改密码第一次时会使用，-u 用户的名称)

* getsebool -a | grep samba

* setsebool -P samba_export_all_rw=on

* systemctl restart smb

* systemctl enable smb

* iptables -F



#### Windows访问

* (从外部访问samba，再搜索栏输入 //192.168.10.10 )

  * 账号 linuxprobe
  * 密码 redhat
* vim readme.txt
    * i can write by windows




#### linux访问

* yum install -y cifs-utils    (安装，用于挂载samba共享文件)

* vim auth.smb  (登录验证,避免每次的登录都需要输入账户密码)

  * ```
    username=linuxprobe
    password=redhat
    domain=MYGROUP
    ```

* mkdir /rick    (创建挂在目录)

* vim /etc/fstab

  * ```
    //192.168.10.10/rick(目标设备挂载名) /rick(本地挂载名) cifs(协议名) credential=/root/auth.smb 0 0 
    ```

* mount -a (挂载全部)

* df -h  (查看挂载信息)

* cd /rick

  * ls
  * vim readme.txt
    * i can also by linux



#### redhat Linux

* cd /rick
* cat readme.txt









### NFS  (need for speed):

### (网络文件系统)

```
Linux  -  Linux     (简单)
Linux  -  Windows   (但配置繁杂，不如Samba好)
```



##### 服务端

* iptables -F

* mkdir /xiaoguo

* cd /xiaoguo

  * vim readme.txt
    * i am xiaoguo by redhat

* chmod -Rf 777 /xiaoguo

* iptables -F

* (红帽6、7版本默认已安装该服务)

* vim /etc/exports   (NFS的主配置文件)

  * ```
    /xiaoguo 192.168.10.*(rw,sync,root_squash)  
    (共享的目录名称，*表示共享给所有人,如上为该网段)
    (sync 为数据同步，root_squash 为将root账户映射为本地的虚拟账户)
    ```

* systemctl restart rpcbind    (nfs协议的基础协议)

* systemctl enable rpcbind

* systemctl restart nfs-server

* systemctl enable nfs-server



##### 客户端

* ping 192.168.10.10
* showmount -e 192.168.10.10  (查看目标主机可用的共享)
* mkdir /xiaoguo  (创建挂载目录)
* vim /etc/fstab    (修改挂载镜像配置)
  * 192.168.10.10:/xiaoguo /xiaoguo nfs default 0 0 
* mount -a
* df -h   (查看挂载镜像)
* cd /xiaoguo
  * ls
  * cat readme.txt
* 可以用 Cerberus (加密)



















# AutoFS （自动挂载）

* yum install autofs   (安装)

* vim /etc/auto.master

  * ```
    /media /etc/iso.misc   (建议以misc结尾)
    ```

* vim /etc/iso.misc

  * ```
    iso -fstype=iso9660,ro :/dev/cdrom
    ```

* umount /dev/cdrom

* systemctl restart autofs

* systemctl enable autofs

* cd /media

  * ls (空白)
  * df -h
  * cd iso
    * pwd  
    * df -h (已自动挂载)













# Bind   (提供域名解析服务)

` CDN   -----  分离解析技术`

`TSIG加密技术  （主服务器 <-----> 从服务器）`

## DNS  （域名解析）

#### 两种工作模式：

##### 正向解析模式： 域名 → IP

##### 反向解析模式： IP → 域名

###### 主服务器

* yum install bind-chroot  (安装)

`chroot  ---  牢笼机制`

* vim /etc/named.conf    (DNS的主配置文件)

  * ```
    11行修改监听的地址
    listen-on port 53 {any;};     
    allow-query   {any;};      //使用服务的网段
    ```

* vim /etc/named.rfc1912.zones   (域名的区域信息块文件)

  * ```
    100 dd  //删除所有
    zone "linuxprobe.com" IN {     //对该域名进行说明
    type master;    //类型为主服务器
    file "linuxprobe.com.zone";     //存储的文件
    allow-update {none;};    //允许可以同步的从属服务器
    };
    zone "10.168.192.in-addr.arpa" IN {      //192.168.10.0/24
    type master;
    file "192.168.10.arpa";  //加载的文件
    allow-update {none;};
    };
    ```

* cd /var/named

  * ls

  * `cp -a --- 复制文件的所有属性到该目录`

  * cp -a named.localhost linuxprobe.com.zone

  * cp -a named.loopback 192.168.10.arpa

  * ls -l 

  * vim linuxprobe.com.zone  (正向解析)

    * ```
      @  IN SOA linuxprobe.com.  root.linuxprobe.com. (   //第二个为通知邮箱
      
      
           NS  ns.linuxprobe.com.
      ns   IN A 192.168.10.10
      www  IN A 192.168.10.10
      ```

  * vim 192.168.10.arpa  （反向解析）

    * ```
      @      IN SOA linuxprobe.com. root.linuxprobe.com. (
      
      
      
            NS     ns.linuxprobe.com.
      ns    IN  A  192.168.10.10
      10    PTR    www.linuxprobe.com.
      ```

  * systemctl restart named

  * systemctl enable named

* iptables -F

* nmtui
  
  * DNS servers 192.168.10.10
* systemctl restart network
* ping www.linuxprobe.com
* nslookup
  * www.linuxprobe.com
  * 192.168.10.10



###### 从属服务器

* yum install bind-chroot

* vim /etc/named.conf

  * ```
    修改第11行和第17行
    { any; }
    { any; }
    ```

* vim /etc/named.rfc1912.zones

  * ```
    100dd
    zone "linuxprobe.com" IN {
    type slave;
    masters { 192.168.10.10; };
    file "slave/linuxprobe.com.zone";
    };
    zone "10.168.192.in-addr.arpa" IN {
    type slave;
    masters { 192.168.10.10;};
    file "slaves/192.168.10.10.arpa";
    };
    ```

* systemctl restart named

* systectl enable named

*  nm-connection-editor

  * DNS 192.168.10.20

* cd /var/named

  * ls   (只同步了一个文件，是虚拟机的模拟问题，多重启服务)
  * cd slaves
    * ls

* systemctl restart network

* systemctl restart named

* nslookup

  * www.linuxprobe.com
  * 192.168.10.10



##### TSIG   (加密)

###### 主服务端

* cd /var/named

  * dnssec-keygen -a HMAC-MD5 -b 128 -n HOST master-slaves

  * `-a 加密方式   -b  加密位数   -n  主机类型  加密的密钥名称`

  * ls

  * cat Kmaster-slave.+157+27176.private

    * 复制Key中的内容

  * cd chroot/etc

    * vim transfer.key

      + ```
        key "master-slave" {
        algorithm hamc-md5;
        secret "刚复制的key中的内容"
        };
        ```

    * chown root:named transfer.key

    * chmod 640 transfer.key

    * ln transfer.key /etc  (做链接方便调用)

    * vim /etc/named.conf

      + ```
        include "/etc/transfer.key";   //第一行
        allow-transfer {key master-slave;};
        ```

      + systemctl restart named

* iptables -F







###### 从服务端

* cd /var/named/salve

  * ls
  * rm -rf * 
  * systemctl restart named
  * ls   (因为主服务器已加密，所以不会同步文件)

* cd /var/named/chroot/etc

  * ls

  * vim transfer.key

    * ```
      key "master-slave" {
      algorithm hmac-md5;
      secret "刚复制的加密的key的内容";
      };
      ```

  * chmod 640 transfer.key

  * chown root:named transfer.key

  * ln transfer.key /etc

  * vim /etc/named.conf

    * ```
      include "/etc/transfer.key";
      server 192.168.10.10
      {
      keys { master-slave; };
      };
      ```

* cd /var/named/slave
  * ls
  * systemctl restart named
  * ls   (已同步过来了)
* nslookup
  
  * www.linuxprobe.com











#### 缓存服务器

* 关机添加网卡，模式为桥接模式

* 开机

* 打开网卡服务

* vim /etc/sysctl.conf

* yum install bind-chroot

* vim /etc/named.conf

  * ```
    第11行,第17行
    {any;}
    {any;}
    forwarders {8.8.8.8;};   转发地址
    ```

* iptables -F

* systemctl restart named

* systemctl enable named

* 去测试客户端ping服务器

* 因为不成功，使用代理

* vim /etc/sysctl.conf

  * net.ipv4.ip_forward=1

* sysctl -p    (使之生效)

* systemctl restart named

* 去客户端ping测试



###### 客户端

* nmtui

  * ```
    Gateway 192.168.10.10
    DNS servers 192.168.10.10   (指向主服务器)
    ```

* systemctl restart network

* ping 192.168.10.10

* nslookup

  * www.baidu.com

* (因为不成功，采用第16章节的代理，在服务端配置)

* systemctl restart network

* nslookup

  * ping www.linuxprobe.com
  * (还是未成功)













#### CDN   (分离解析技术)

`使用win 7来模拟客户端`

###### 服务端

* yum install bind-chroot

* vim /etc/named.conf

  * ```
    第11行,第17行
    { any; }
    { any; }
    注释第51-54行
    ```

* cd /etc

  * vim named.rfc1912.zones

    * ```
      100dd   //清空
      acl "china" { 122.71.115.0/24; };
      acl "american" { 106.185.25.0/24; };
      view "china" {
      match-clients { "china"; };
      zone "linuxprobe.com" {
      type master;
      file "linuxprobe.com.china";
      };
      };
      view "american" {
      match-clients { "american"; };
      zone "linuxprobe.com" {
      type master;
      file "linuxprobe.com.american";
      };
      };
      ```

* cd  /var/named

  * ls

  * cp -a named.localhost linuxprobe.com.china

  * cp -a named.localhost linuxprobe.com.american

  * vim linuxprobe.com.china

    * ```
      @       IN SOA linuxprobe.com. root.linuxpronbe.com.
      
      
               NS     ns.linuxprobe.com.
      ns       IN  A  122.71.115.10
      www      IN  A  122.71.115.10
      ```

* nmtui    (配置IP地址)

  * Addresses  122.71.115.10/24
  * Addresses  106.185.25.10/24

* systemctl restart named

* vim /etc/named/linuxprobe.com.american

  * ```
    @           IN SOA   linuxprobe.com. root.linuxprobe.com.
    
    			NS        ns.linuxprobe.com.
    ns          IN   A    106.185.25.10
    www         IN   A    106.185.25.10
    ```

* systemctl status named.service    (排错)

* systemctl restart named

* systemctl enable named

* systemctl restart network

* iptables -F

* 去配置win 7的ip地址







###### Win 7

* IP : 122.71.115.30/24
* DNS : 122.71.115.10
* ping www.linuxprobe.com
* 再改一下
* IP : 106.185.25.10
* DNS : 106.185.25.10
* systemctl restart network







# 使用DHCP动态管理主机地址

##### 作用域：声明的范围，并不是真实分配的

##### 超级作用域：可以跨网段的声明，也并不是真实分配的

##### 地址池：真实为用户分配的范围

##### 排除范围：除了真实分配的，剩余的声明

##### 租约：①defaults（默认租约时间，硬）  ②max（最大租约时间，软）

##### 预约：绑定

*  yum install -y dhcp   (前提线搞定镜像)

* vim /etc/dhcp/dhcpd.conf     (修改配置文件)

  * ```
    ddns-update-style none;   //动态dns更新协议
    ignore  client-updates;   //忽略客户端更新
    subnet 192.168.10.0 netmask 255.255.255.0 {  //作用域
    range 192.168.10.50 192.168.10.100;  //地址池
    option subnet-mask 255.255.255.0;   //为用户分配的子网掩码
    option routers 192.168.10.10;  //用户网关地址
    option domain-name-servers 192.168.10.10; //为用户分配的DNS信息
    default-lease-time 21600;   //租约时间
    max-lease-time 43200;  //最大租约时间
    }
    
    ```

* systemctl  restart dhcpd

* systemctl enable dhcpd

* iptables -F

* service iptables save

* systemctl status dhcpd 







##### 客户端

* dhcp
* 编辑
  * 关闭本地DHCP服务将IP地址分配给虚拟机
  * 选择仅主机模式







# 邮件传输协议

* 邮件传输示意图：

![image-20200705092432741](C:\Users\31650\AppData\Roaming\Typora\typora-user-images\image-20200705092432741.png)

* 邮件系统的工作流程：

![image-20200705092832844](C:\Users\31650\AppData\Roaming\Typora\typora-user-images\image-20200705092832844.png)



* 配置主机名称

* vim /etc/hostname

  * ```
    mail.linuxprobe.com    //定义主机名称
    ```

* iptables -F

* service iptables save

* yum install -y bind-chroot    //安装DNS服务

* vim /etc/named.conf    //域名配置文件

  * ```
    listen-on port 53 {any;};
    allow-query       {any;};
    ```

* vim /etc/named.rfc1912.zones   //域名数据文件

  * ```
    zone "linuxprobe.com" IN {   //定义根域
    type master;    //信息类型为主服务器
    file "linuxprobe.com.zone";  //本地文件路径
    allow-update {none;};  //从属服务器更新
    };
    ```

* cd /var/named

  * ls

  * cp -a named.localhost  linuxprobe.com.zone

  * vim linuxprobe.com.zone

    * ```
      @    IN SOA  linuxprobe.com. root.linuxprobe.com. 
      
           NS       ns.linuxprobe.com.
      ns   IN A     192.168.10.10
      @    IN MX    10 mail.linuxprobe.com.
      mail IN A     192.168.10.10
      ```

  * systemctl restart named

  * systemctl enable named

  * vim /etc/postfix/main.cf     //发送邮件服务配置

    * ```
      第75行
      myhostname = mail.linuxprobe.com
      
      第83行
      mydomain = linuxprobe.com
      
      第99行
      myorigin = $mydomain
      
      第116
      inet_interfaces = all
      
      第164行
      mydestination = $myhostname, $mydomain //接受的邮件服务器
      ```

  * systemctl restart postfix

  * systemctl enable postfix

  * useradd xiaoguo

  * passwd xiaoguo

    * redhat

  * yum install -y dovecot    //收取邮件并保存用户信息的服务

  * vim /etc/dovecot/dovecot.conf

    * ```
      第24行
      protocols = imap pop3 lmtp
      第25行
      disable_plaintext_auth = no  //禁用明文认证
      第48行
      login_trusted_networks = 192.168.10.0/24  //只允许内网用户访问
      
      ```

  * vim /etc/dovecot/conf.d/10-mail.conf

    * ```
      mail_location = mbox:~/mail:INBOX=/var/mail/%u
      ```

  * su - xiaoguo

  * mkdir -p mail/.imap/INBOX

  * exit

  * systemctl restart  dovecot

  * systemctl enable  dovecot

  * iptables -F

  * 



###### 客户端（windows）

* 配置IP，保证设备的连通性
* 安装outlook
* 登录
  * 账号 xiaogu
  * 密码  redhat
  * 第二次非加密会成功
  * 发送邮件
* 别名尝试发送







###### 服务器端

* mail    //登录邮箱查看

* mail xiaoguo@linuxprobe.com    //回复邮件

  * 发送内容
  * .      //英文.表示结束发送

* vim /etc/aliases      //用户的别名文件

  * ```
    xiaolang:      root     //root用户的别名
    ```

* newaliases

* systemctl restart postfix

* systemctl restart dovecot













# Squid   （和CDN相似）

* 正向代理模式

![image-20200705110210099](C:\Users\31650\AppData\Roaming\Typora\typora-user-images\image-20200705110210099.png)

* 反向代理模式

![image-20200705110613404](C:\Users\31650\AppData\Roaming\Typora\typora-user-images\image-20200705110613404.png)



#### 正向解析    （标准）

###### 服务端

* iptables -F
* service iptables save
* yum install -y squid     //安装服务
* 添加网卡
* ![image-20200705114039122](C:\Users\31650\AppData\Roaming\Typora\typora-user-images\image-20200705114039122.png)
  * 桥接模式
* 配置网卡
* nm-connection-editor
  * DHCP
* systemctl restart network
* systemctl restart squid
* systemctl enable squid
* iptables -F
* service iptables save
* 去客户端设置









###### 客户端（windows）

* 打开ie

* 工具

  * Internet选项

  * 连接

    *  局域网的设置

    * 代理服务器

      ```
      192.168.10.10    3128   //squid的端口号为3128
      ```

* 访问www.baidu.com



#### 反向解析    （透明）











