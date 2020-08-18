#### 网络问题排查
```
ping 检测是否连接到主机
    ping www.baidu.com
traceroute  跟踪当前主机到目标主机的网络状态，-w 1超时最多等1秒
    traceroute -w 1 www.baidu.com

mtr  显示自己主机的网络状态

nslookup  域名解析成ip
    nslooup www.baidu.com

telnet  检测端口
    telnet www.baidu.com 80

tcdump  网络抓包   -i any 抓取所有网卡里的数据包，-n 把域名解析成 ip ，port 80 抓取指定端口  host 10.0.0.1 抓取当前主机到某个主机的数据包
    tcpdump -i any -n port 80
    tcpdump -i any -n host 10.0.0.1
    tcpdump -i any -n host 10.0.0.1 and port 80
    tcpdump -i any -n host 10.0.0.1 and port 80 -w /tmp/filename 捕获并且保存

netstat 监听地址 -n 域名转换，-t 显示tcp ，-p 进程 ，-l tcp状态 listen
    netstat -ntpl

ss 跟netstat一样，参数也一样，显示的格式不一样
```



#### kernel related
```
rpm 格式内核
    查看内核版本
        uname -r
    升级内核版本
        yum install kernel-3.10.0   这种方式一般不能升级到最新
        epel软件仓库会有较高的软件版本。yum install epel-release -y
    升级已安装的其他软件包和补丁
        yum update    除了升级内核，还会升级软件包。正常不要使用。


源代码编译安装内核
yum install gcc gcc-c++ make ncurses-devel openssl-devel elfutils-libelf-devel

下载并解压缩内核
https://www.kernel.org
tar xvf linux-5.1.10.tar.xz -C /usr/src/kernels

配置内核编译参数
cd /usr/src/kernels/linux-5.1.10/
make menuconfig | allyesconfig | allnoconfig
make allyesconfig （无脑全选）

使用当前系统内核配置
cp /boot/config-kernelversion.platform /usr/src/kernels/linux-5.1.10/.config


查看cpu
lscpu

编译
make j2 all

安装内核
make modules_install
make install
```


#### 进程
```
进程-运行中的程序，从程序开始运行到终止的整个生命周期是可管理的

查看命令
ps
    -e 表示所有的终端运行的进程
    -f  显示更多信息，比如 UID、PPID（父进程）、CMD（命令的完整路径）
    -L  多显示 LWP ，线程信息
    ps -eLf  常用命令

pstree  查看进程树

top  动态查看进程信息top -p 进程号， CPU usage, memory usage

CPU 100%?? 为什么？？ 单核利用率100%

结论：
进程也是树形结构
进程和权限有着密不可分的关系
```


####磁盘分区和挂载
```
常用命令
fdisk
mkfs
parted
mount


常见配置文件
/etc/fstab

用fdisk创建分区（一个硬盘设备可以创建多个分区，也可以创建一个）
1：fdisk -l 查看有几个硬盘设备及分区
2：fdisk /dev/sdc    （比如有设备sdc，则可以针对sdc进行分区）
3：之后 m 键是帮助
4：n 表示新建一个分区
5：新建分区时，需要选择主分区和扩展分区，其中 p表示主分区，最多有4个。e表示扩展分区（里面可以建立逻辑分区）。一般把一块硬盘划分为一个主分区。使用扩展分区时，只能建立3个主分区。
6：选择区分编号1-4
7：指定分区扇区大小，默认2048
8：指定分区大小。默认全部。可以 + 20G等可以选择分区大小
9：q 表示退出，分区不生效。w 表示生效

建立完分区后，需要对分区进行格式化。

mkfs.ext4  mkfs.xfs等命令
mkfs.ext4 /dev/sdc1

然后要进行操作，linux里都是文件级别的操作，需要挂载到某个目录下
mkdir /mnt/sdc1
mount /dev/sdc1 /mnt/sdc1 挂载上去
对/mnt/sdc1的读写就会落入sdc1设备上

1、一个硬盘
2、进行分区
3、格式化
4、挂载
5、对指定目录进行操作


需要注意的事情：
如果一个硬盘大于 2T ，不能使用 fdisk 进行分区，需要使用 parted
parted /dev/sdd
help 获取帮助

mount 进行挂载是临时的，不是固化的
vim /etc/fstab
在文件中新增下面一句话
/dev/sdc1 /mnt/sdc1 ext4 defaults（表示权限）0 0
```



####nginx
https://zhuanlan.zhihu.com/p/31196264
```
Nginx主要是用于Http服务器，反向代理服务器，邮件服务器
Nginx由多个模块组成，每个请求的完成都是由一个或多个模块共同完成的。
Nginx 默认采用守护模式启动，守护模式让master进程启动后在后台运行。在Nginx运行期间主要由一个master主进程和多个worker进程（数目一般与cpu数目相同）
master主进程主要是管理worker进程，对网络事件进行收集和分发：
接收来自外界的信号
向各worker进程发送信号
监控worker进程的运行状态，当worker进程退出后(异常情况下)，会自动重新启动新的worker进程
nginx用一个独立的worker进程来处理一个请求，一个worker进程可以处理多个请求：
当一个worker进程在accept这个连接之后，就开始读取请求，解析请求，处理请求，产生数据后，再返回给客户端，最后才断开连接。
一个请求，完全由worker进程来处理，而且只在一个worker进程中处理。采用这种方式的好处：
节省锁带来的开销。对于每个worker进程来说，独立的进程，不需要加锁，所以省掉了锁带来的开销，同时在编程以及问题查上时，也会方便很多
独立进程，减少风险。
采用独立的进程，可以让互相之间不会影响，一个进程退出后，其它进程还在工作，服务不会中断，master进程则很快重新启动新的worker进程。
在一次请求里无需进程切换
```


#### 高并发和高可用常用策略
https://www.cnblogs.com/aspirant/p/11451066.html
缓存、降级和限流。缓存的目的是提升系统访问速度和增大系统能处理的容量，可谓是抗高并发流量的银弹；而降级是当服务出问题或者影响到核心流程的性能则需要暂时屏蔽掉，待高峰或者问题解决后再打开；而有些场景并不能用缓存和降级来解决，比如稀缺资源（秒杀、抢购）、写服务（如评论、下单）、频繁的复杂查询（评论的最后几页），因此需有一种手段来限制这些场景的并发/请求量，即限流。

* 数据库系统：数据库系统可以采取`集群策略`以保证某台数据库服务器的宕机不会影响整个系统，并且通过`负载均衡`策略来降低每一台数据库服务器的压力,另外采取`读/写分离`的方法降低数据库负载，再加上分库和分表进一步降低数据库负载，从而可以从容地应对高并发问题。当然成本会比较高，毕竟要这么多服务器。


* 分布式缓存系统：建立分布式缓存系统是至关重要的，所有的读写都先进入缓存系统，然后由缓存系统来安排从数据库系统/源服务器的读写。比如读的话，要是找不到，则把数据从数据库中读出放入缓存系统里（同样的，把页面从源服务器中获取并保存在缓存服务器中），再读取给客户端；写的话，则先写入缓存，由缓存系统把数据异步提交到消息队列中，然后写入数据库里。缓存分为硬盘缓存和内存缓存，硬盘缓存更多地用来缓存页面和页面资源例如多媒体资源，而内存缓存更多地用来缓存数据库的数据和应用中的一些状态。硬盘缓存有Squid，内存缓存有Memcached，微软在.Net Framework 4.0推了个Velocity也是内存缓存。

* 监控系统：这么多服务器和系统，总归是需要监控的，不然出了问题排查起来会非常麻烦，所以上述系统在开发时也要考虑到监控这一块，做好日志记录，然后配合监控系统可以一下子查到问题根源。

* 前端系统：和数据库系统一样可以采用服务器集群和负载均衡，可以把各个服务细分然后注册到服务中心再分别部署到不同的服务器上即采用分布式服务的方式，还可以使用多线程，另外为了更好地用户体验，可以多用异步方式和客户端操作来显示数据或者执行操作，ajax，js等等可以派不少用处，此外，还可以使用网页静态化（这样不但降低了开销还会提高网页被搜索到的概率，.Net有URLRewrite可以做到，只需要引入dll并注册然后设置Rewrite的规则即可），还有图片等多媒体资源单独设置服务器与页面分离，使用镜像网站或者CDN技术（Content Delivery Network，智能镜像网站+缓存技术，让用户可以访问自己最近的镜像网站的缓存服务器中的缓存页面）

* 服务器CPU和IO的平衡：对于所有的服务器，都需要保证它们的CPU和IO保持平衡，如果失衡，需要查找原因，更改部署和配置以达到平衡。最佳线程数=CPU核的个数*2 + 2，当然这个只是经验之谈，实际公式比较复杂，为最佳线程数=((线程等待时间+线程cpu时间)/线程cpu时间) * cpu核的数量。


#### 前后端交互
http://xbhong.top/2018/02/28/ajax/
