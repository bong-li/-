# zookeeper


[toc]

### 概述

#### 1.用途
用来开发分布式程序

#### 2.核心功能
* 分布式一致性
* 有一个leader节点
* 数据树（key-value的形式存储数据）
  * key是有层级的，跟linux中的目录一样（/为根）

#### 3.特点
* 读速度 比 写速度更快
  * 读 可以通过任意一个节点
  * 写 会先将写请求发送到leader，由leader完成写操作
* 数据存储在内存中

#### 4.应用场景

* 配置一致
比如几个程序监听 zookeeper中的某个key，如果这个key变化了，则程序采取相应操作

* HA
主程序，创建一个key，如果该程序挂掉，则该key会自动清除
备份程序监听这个key，如果该key清楚了，会自动创建，如果该key存在，则继续监听着

* 服务发现
* 负载均衡
* 分布式锁

***

### 配置
```shell
#server.<ID>=主机名:2888:3888[:observer]
#<ID>号不重复进行，有多少个节点，就写多少个server
server.1=192.168.1.1:2888:3888
server.2=192.168.1.2:2888:3888
server.3=192.168.1.3:2888:3888
```

* 需要在zookeeper的数据目录下设置该节点的id
```shell
mkdir <DATA_DIR>

echo <ID> > <DATA_DIR>/myid
```
* 启动
```shell

```