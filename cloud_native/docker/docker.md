# docker
[toc]

### 基础概念

#### 1.docker隔离6种namespace
|namespace|说明|
|-|-|
|uts|unix timesharing system，主机名和域名的隔离|
|user|用户的隔离|
|mnt|mount，文件系统的隔离|
|pid|进程的隔离|
|ipc|进程间通信的隔离|
|net|网络的隔离|


#### 2.使用docker的前提
* 需要关闭selinux

#### 3.有两种存储卷
（1）Dokcer-managed volume
由docker自动设置宿主机目录，目录为`/var/lib/docker/volumes/容器id/_data`
```shell
  -v 容器目录
```

（2）Bind-mount volume
```shell
  -v 宿主机目录:容器目录
```
#### 4.注意事项
* 在容器中跑任何程序都不能运行在后台，否则容器会认为程序终止了，容器也就会结束
***
### 相关操作和概念
#### 1.容器相关
（1）查看容器与真机绑定的端口号
```shell
docker port xx        #xx为容器ip
```

（2）查看容器使用的存储卷
```shell
docker inspect -f {{.Mounts}} xx		
```

（3）在容器外执行命令
```shell
docker exec -it xx /bin/bash -c '命令'
```
（4）复制使用其它容器的卷
```shell
  --volumes-from
```

（5）对容器资源限制的相关参数（docker run时使用）

* 对内存资源限制
```shell
  -m 1g                       #表示最多使用1g内存
  --memory-swap xx            #使用的swap的限制，当此值和-m设置的一样，则不允许使用swap（默认使用2*M）
  --oom-kill-disable          #表示当宿主机内存满了，需要kill进程时，禁止kill该容器
```

* 对cpu资源限制
```shell
  --cpus xx                   #设置该容器最多只能使用宿主机的几核cpu，比如：1.5，只能使用1.5核
  --cpuset-cpus  xx           #设置只能运行在指定cpu上（从0开始编号），比如：0,2，只能跑在0号和2号cpu上
  --cpu-shares  xx            #设置一个数值，比如宿主机上只运行了两个容器一个设为1024，一个设为512，当两个都需要cpu时，则按2:1进行分配
```

（6）使用`-v`挂载存储卷时，不管宿主机还是容器，只要该目录不存在，会自动创建

（7）运行容器，赋予相关权限
```shell
  --privileged        #能够修改系统参数（即宿主机的，因为容器用的宿主机内核）
  --user=0            #以容器内uid为0的用户权限执行（即root权限）
```

#### 2.镜像相关
（1）批量打包镜像，简单且压缩体积的方法
```shell
tar -Pzcf docker.tar.gz /var/lib/docker/overlay2/*
```
（2）docker镜像加速

docker cn 或者 阿里云加速器 或者 中国科技大学
```shell
#vim /etc/docker/daemon.json
  {
      "registry-mirrors": ["http://registry.docker-cn.com"]
  }
```

#### 3.网络相关
（1）启动容器时利用`--net=xx`指定使用的网络模式

（2）修改docker进程监听的端口，可以连接远程的docker服务端
```shell
#vim /etc/dokcer/daemon.json
  {
    "hosts": ["tcp://0.0.0.0:2222","unix:///var/run/dokcer.sock"]         //默认只监听本机的docker.sock套接字
  }

  docker -H xx:2222 ps
```

（3）当容器以host网络模式启动，会复制宿主机的/etc/hosts文件

（4）如何恢复iptables rules：重启docker
#### 4.其他
（1)查看网卡是否处于混杂模式
```shell
cat /sys/class/net/xx/flags
#0x100 表示没有开启
#0x110 表示开启了
```
（2）docker-ce的配置文件
```shell
/etc/docker/daemon.json
```
***
### docker镜像
#### 1.image
是一个只读模板
#### 2.container
容器是镜像的可运行实例
#### 3.镜像采用分层构建机制
![](./imgs/docker_01.png)
##### 3.1 概述
* 最底层为bootfs
```
用于系统引导的文件系统，包括bootloader和kernel，
容器启动后会被卸载以节省内存
```
* 上面的所有层整体称为rootfs
```
表现为容器的根文件系统，
在docker中rootfs只挂载为"只读"模式，
而后通过"联合挂载"技术额外挂载一个"可写"层
```
* 最上层为"可写"层
```
所有的写操作，只能在这一层实现，
如果删除容器，这个层也会被删除，即数据不会发生变化
```
##### 3.2 分层的优点
某些层可以进行共享，所以减轻了分发的压力（比如，底层是一个centos系统，可以将底层与tomcat层或nginx层联合，提供相应的服务）

#### 4.docker支持的存储驱动
|storage driver|支持的后端文件系统|
|-|-|
|overlay2，overlay|xfs，ext4|
|aufs|xfs，ext4|
|devicemapper|direct-lvm|
|btrfs|btrfs|
|zfs|zfs|
|vfs|所有文件系统|

#### 5.overlay2驱动
##### 5.1 特点
* image的每一层，都会在`/var/lib/docker/overlay2/`目录下生成一个目录
* 启动容器后，会在`/var/lib/docker/overlay2/`目录下生成一个目录（即**可写层**），会将**overlay文件系统** **挂载** 在该目录下的`merged`目录上

##### 5.2`/var/lib/docker/overlay2/<id>/`目录下可能有的内容
|目录或文件|说明|
|-|-|
|diff（目录）|包含该层的内容|
|lower（文件）|记录该层下面的层的信息|
|merged（目录，可写层才有）|将底层和可写层合并后的内容|

#### 6.docker registry
一个registry可以存在多个repository（一般一个软件有一个repository，里面存储该软件不通版本的镜像）
repository可分为"顶层仓库"和"用户仓库"
用户仓库名称格式：用户名/仓库名

#### 7.打包镜像
可以打包多个镜像
打包镜像只指定仓库，会将该仓库所有的镜像都打包
打包镜像指定仓库和标签，只会打包指定镜像
打包镜像指定镜像id，只会打包该镜像，不会包括该镜像的仓库和标签

***
### docker网络模式
#### 1. 虚拟交换机
* 如果使用虚拟交换机，虚拟网卡会创建一对，一个在宿主机中，一个在虚拟交换机上
* 如果使用nat桥，可以在宿主机上查看nat的规则：iptables -t nat -vnL
* 桥接模式和nat模式区别：
都是bridge模式
桥接模式不能与不同网段的通信(需要虚拟一个路由器），nat可以

#### 2.bridge模式（是一个nat桥）
有一个虚拟交换机dokcer0，所有的容器都与docker0相连，通过宿主机路由出去
```shell
#修改bridge所在的网段：vim /etc/docker/daemon.json
  {
    "bip": "192.168.0.11/24"          #指定docker0桥的ip地址和其掩码
  }
```
#### 3.host模式
  新创建的容器与宿主机共享一个netns
```shell
  --network host
```

#### 4.container模式
  新创建的容器和已经存在的一个容器共享一个netns
```shell
  --network container:xx         //xx为已经存在的容器的名字
```

#### 5.none模式
  使用none模式，Docker容器拥有自己的Network Namespace，但是，并不为Docker容器进行任何网络配置。
  也就是说，这个Docker容器没有网卡、IP、路由等信息。需要我们自己为Docker容器添加网卡、配置IP等

#### 6.overlay模式
  可以实现容器的跨主机通信，host模式也可以

（1）工作方式：
* 需要创建一个consul的服务容器，提供ip地址池，比如10.0.9.0/24之类的
* 容器的从consul获取ip
* 获取完了后，会通过eth1进行通信

（2）工作原理
* 通过隧道的方式，源地址和目标地址不变
* 再封装一层四层和三层报文，源地址为源宿主机地址，目标地址为目标宿主机地址
* 然后再进行二层等报文的封装

#### 6.macvlan模式
**一定要允许混杂模式，如果是vm：真机 -> 虚拟交换机 -> 允许混杂模式**
* 允许在同一个物理网卡上配置多个 MAC 地址，即多个 interface，每个 interface 可以配置自己的 IP
* 一个网卡只能用于一个macvlan网络（不能用于其他，也不能有ip），一个macvlan网络可以分配多个ip地址
```shell
docker network create -d macvlan --subnet=10.0.36.0/24 \
  --ip-range=10.0.36.115/32 --gateway=10.0.36.254  \
  -o parent=ens34 xx
```

#### 7.macvlan通信     
子网掩码越长，路由优先级越高
这个方法只能解决宿主机和容器通信的问题，所以没什么用

（1）创建macvlan网络
```shell
docker  network create -d macvlan --subnet 10.0.36.0/24 \
  --ip-range 10.0.36.192/27 --aux-address 'host=10.0.36.223' \         #aux-address就是排除的ip地址，即不会进行分配
  --gateway 10.0.36.254  -o parent=ens34 mynet
```
（2）创建一个新的macvlan接口，用于连接host和container
```shell
  ip link add mynet-shim link ens34 type macvlan mode bridge     
  ip addr add 10.0.36.223/32 dev mynet-shim
  ip link set mynet-shim up
```

（3）添加路由信息
```shell
  ip route add 10.0.36.192/27 dev mynet-shim
```
#### 8.创建docker网络
```shell
  docker network create -d 驱动 --subnet "xx" --gateway "xx" 网络的名字
```