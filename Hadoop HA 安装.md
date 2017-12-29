# Hadoop HA 安装

HA 的意思是 High Availability 高可用，指当当前工作中的机器宕机后，会自动处理这个异常，并将工作无缝地转移到其他备用机器上去，以来保证服务的高可用。

HA 方式安装部署才是最常见的生产环境上的安装部署方式。Hadoop HA 是 Hadoop 2.x 中新添加的特性，包括 NameNode HA 和 ResourceManager HA。

因为 DataNode 和 NodeManager 本身就是被设计为高可用的，所以不用对他们进行特殊的高可用处理。

**第九步  时间服务器搭建**

Hadoop 对集群中各个机器的时间同步要求比较高，要求各个机器的系统时间不能相差太多，不然会造成很多问题。

可以配置集群中各个机器和互联网的时间服务器进行时间同步，但是在实际生产环境中，集群中大部分服务器是不能连接外网的，这时候可以在内网搭建一个自己的时间服务器（NTP服务器），集群的各个机器与这个时间服务器进行时间同步。

三十三、配置NTP服务器

我们选择第三台机器（bigdata-senior03.chybinmy.com）为NTF服务器，其他机器和这台机器进行同步。

1. 检查 ntp 服务是否已经安装

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvayp7WDnNiaic3TzIJIf16aJmBhahYMkpWZWoHWglR32M2DgfsuzWMnkg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

显示已经安装过了ntp程序，其中`ntpdate-4.2.6p5-1.el6.centos.x86_64` 是用来和某台服务器进行同步的，`ntp-4.2.6p5-1.el6.centos.x86_64`是用来提供时间同步服务的。

2. 修改配置文件 ntp.conf

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvsCVicjGEibvafZicEsJDn4ODKDStofWL40oMAAGAOy89mYuoibHtX9LPuw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

**启用 restrice，修改网段**

restrict 192.168.100.0 mask 255.255.255.0 nomodify notrap
将这行的注释去掉，并且将网段改为集群的网段，我们这里是100网段。

**注释掉 server 域名配置**

![img](https://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvsiay01FptbDoRF02TIzic6lH8pVnLfsCaQ0BAhjyA4GMrS7JTf2z7xew/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&retryload=1)

是时间服务器的域名，这里不需要连接互联网，所以将他们注释掉。

**修改**

server 127.127.1.0

fudge 127.127.1.0 stratum 10

\3. 修改配置文件 ntpd

![img](https://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvsKYk1Ya01nbVia01LDoA4QfMwQSDGWJoauoYydyiauUG1jgazl9MCbxw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&retryload=1)

添加一行配置：SYNC_CLOCK=yes

![img](https://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvib1UXnT8CuEKIsojw4l5nW9icr7QonPEFk4GCcyDKGMtiaG6iaz13PcoqQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&retryload=1)

\4. 启动 ntp 服务

![img](https://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaPZfsIDmrMFibwO7lteFCeWDkos5kaico9Kcb9CPXTaAiaFxAYvzpeyx1rEVREcoTYpN47nO8TwMXEUA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&retryload=1)

这样每次机器启动时，ntp 服务都会自动启动。

三十四、配置其他机器的同步

切换到 root 用户进行配置通过 contab 进行定时同步：

![img](https://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvSw7X4YeaicJmsbiaAINYu6vPVqSNDUQwia4vfgw5z8YU0FV1H8vZdVWgA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&retryload=1)

三十五、 测试同步是否有效

1. 查看目前三台机器的时间

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEv3CbKwm5g0sQWY5Gu5ic3jeqU1iadibS95O7Giav3enWUcvKmE7hib8qpOiaQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

2. 修改 bigdata-senior01上的时间

将时间改为一个以前的时间：

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvWPnthOfboE7eVcYKEEd8RXTKNVFMFHVHGbEEKt24Y43Iw3cVo2NcwQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

等10分钟，看是否可以实现自动同步，将 bigdata-senior01 上的时间修改为和 bigdata-senior03 上的一致。

3. 查看是否自动同步时间

![img](https://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvbWcYcn8mmusDfyuZYoib7SfnibnkBBsVibsjHSRicVia33ic7uX2MrplyLzw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&retryload=1)

可以看到 bigdata-senior01 上的时间已经实现自动同步了。

#### 第十步  Zookeeper 分布式机器部署

三十六、zookeeper 说明

**Zookeeper 在Hadoop 集群中的作用。**

Zookeeper是分布式管理协作框架，Zookeeper集群用来保证Hadoop集群的高可用，（高可用的含义是：集群中就算有一部分服务器宕机，也能保证正常地对外提供服务。）

**Zookeeper 保证高可用的原理。**

Zookeeper 集群能够保证 NamaNode 服务高可用的原理是：Hadoop 集群中有两个 NameNode 服务，两个NaameNode都定时地给 Zookeeper 发送心跳，告诉 Zookeeper 我还活着，可以提供服务，单某一个时间只有一个是 Action 状态，另外一个是 Standby 状态，一旦 Zookeeper 检测不到 Action NameNode 发送来的心跳后，就切换到 Standby 状态的 NameNode 上，将它设置为 Action 状态，所以集群中总有一个可用的 NameNode，达到了 NameNode 的高可用目的。

**Zookeeper 的选举机制。**

Zookeeper 集群也能保证自身的高可用，保证自身高可用的原理是，Zookeeper 集群中的各个机器分为 Leader 和 Follower 两个角色，写入数据时，要先写入Leader，Leader 同意写入后，再通知 Follower 写入。客户端读取数时，因为数据都是一样的，可以从任意一台机器上读取数据。

这里 Leader 角色就存在单点故障的隐患，高可用就是解决单点故障隐患的。

Zookeeper 从机制上解决了 Leader 的单点故障问题，Leader 是哪一台机器是不固定的，Leader 是选举出来的。

选举流程是，集群中任何一台机器发现集群中没有 Leader 时，就推荐自己为 Leader，其他机器来同意，当超过一半数的机器同意它为 Leader 时，选举结束，所以 Zookeeper 集群中的机器数据必须是奇数。

这样就算当 Leader 机器宕机后，会很快选举出新的 Leader，保证了 Zookeeper 集群本身的高可用。

**写入高可用。**

集群中的写入操作都是先通知 Leader，Leader 再通知 Follower 写入，实际上当超过一半的机器写入成功后，就认为写入成功了，所以就算有些机器宕机，写入也是成功的。

**读取高可用。**

zookeeperk 客户端读取数据时，可以读取集群中的任何一个机器。所以部分机器的宕机并不影响读取。

zookeeper 服务器必须是奇数台，因为 zookeeper 有选举制度，角色有：领导者、跟随者、观察者，选举的目的是保证集群中数据的一致性。

三十七、安装 zookeeper

我们这里在 BigData01、BigData02、BigData03 三台机器上安装 zookeeper 集群。

\1. 解压安装包

在 BigData01上安装解压 zookeeper 安装包。

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaPZfsIDmrMFibwO7lteFCeWDXQibMFloy799dibvTOLoxXI9egUZZiafsRKRRE3v0Xo5ibNU0RgH5P6vqA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

\2. 修改配置

拷贝 conf 下的 zoo_sample.cfg 副本，改名为 zoo.cfg。zoo.cfg 是 zookeeper 的配置文件：

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaPZfsIDmrMFibwO7lteFCeWDAQqjOnNUWeCD7uVbxxWZNiaasZruMW0CB8uYVjRwE8mLicJRkibarSAkg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

dataDir 属性设置 zookeeper 的数据文件存放的目录：

dataDir=/opt/modules/zookeeper-3.4.8/data/zData

指定 zookeeper 集群中各个机器的信息：

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaPZfsIDmrMFibwO7lteFCeWDN9laWVGVlgtjicgUzPdlK3nBoDvfoeb0icKkRlJsxjN9D699pbkWVrRw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

server 后面的数字范围是1到255，所以一个 zookeeper 集群最多可以有255个机器。

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaPZfsIDmrMFibwO7lteFCeWDDlGDaZfE2qTTuvuc30zRCBXnLUjsuL4JHebBCmJbYXkGibsK3eXjCZg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

3. 创建 myid 文件

在 dataDir 所指定的目录下创一个名为 myid 的文件，文件内容为 server 点后面的数字。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

4. 分发到其他机器

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

5. 修改其他机器上的myid文件

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

6. 启动 zookeeper

需要在各个机器上分别启动 zookeeper。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

三十八、zookeeper 命令

**进入zookeeper Shell**

在zookeeper根目录下执行 bin/zkCli.sh进入zk shell模式。

zookeeper很像一个小型的文件系统，/是根目录，下面的所有节点都叫zNode。

**进入 zk shell 后输入任意字符，可以列出所有的 zookeeper 命令**

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaPZfsIDmrMFibwO7lteFCeWDq4d8va4tSDEib6fNUUBkLPGO3jSmO2yPCd9g3qGiaRqzeeBCIj9OJoxg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

**查询 zNode 上的数据：get /zookeeper**

**创建一个 zNode : create /znode1  “demodata “**

**列出所有子 zNode：ls /**

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaPZfsIDmrMFibwO7lteFCeWDAzKLC4aX3cPKXk3rs5QaKO1Sq91G9uiavicxCHE1aaPQqcPgEbCTP5rw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

**删除 znode : rmr /znode1**

**退出 shell 模式：quit**

#### 第十一步  Hadoop 2.x HDFS HA 部署

三十九、HDFS HA原理

单 NameNode 的缺陷存在单点故障的问题，如果 NameNode 不可用，则会导致整个 HDFS 文件系统不可用。

所以需要设计高可用的 HDFS（Hadoop HA）来解决 NameNode 单点故障的问题。解决的方法是在 HDFS 集群中设置多个 NameNode 节点。

但是一旦引入多个 NameNode，就有一些问题需要解决。

- HDFS HA 需要保证的四个问题：

- - 保证 NameNode 内存中元数据数据一致，并保证编辑日志文件的安全性。
  - 多个 NameNode 如何协作
  - 客户端如何能正确地访问到可用的那个 NameNode。
  - 怎么保证任意时刻只能有一个 NameNode 处于对外服务状态。

- 解决方法

- - 对于保证 NameNode 元数据的一致性和编辑日志的安全性，采用 Zookeeper 来存储编辑日志文件。
  - 两个 NameNode 一个是 Active 状态的，一个是 Standby 状态的，一个时间点只能有一个 Active 状态的
    NameNode 提供服务,两个 NameNode 上存储的元数据是实时同步的，当 Active 的 NameNode 出现问题时，通过 Zookeeper 实时切换到 Standby 的 NameNode 上，并将 Standby 改为 Active 状态。
  - 客户端通过连接一个 Zookeeper 的代理来确定当时哪个 NameNode 处于服务状态。

四十、HDFS HA 架构图

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

- HDFS HA 架构中有两台 NameNode 节点，一台是处于活动状态（Active）为客户端提供服务，另外一台处于热备份状态（Standby）。

- 元数据文件有两个文件：fsimage 和 edits，备份元数据就是备份这两个文件。JournalNode 用来实时从 Active NameNode 上拷贝 edits 文件，JournalNode 有三台也是为了实现高可用。

- Standby NameNode 不对外提供元数据的访问，它从 Active NameNode 上拷贝 fsimage 文件，从 JournalNode 上拷贝 edits 文件，然后负责合并 fsimage 和 edits 文件，相当于 SecondaryNameNode 的作用。

  最终目的是保证 Standby NameNode 上的元数据信息和 Active NameNode 上的元数据信息一致，以实现热备份。

- Zookeeper 来保证在 Active NameNode 失效时及时将 Standby NameNode 修改为 Active 状态。

- ZKFC（失效检测控制）是 Hadoop 里的一个 Zookeeper 客户端，在每一个 NameNode 节点上都启动一个 ZKFC 进程，来监控 NameNode 的状态，并把 NameNode 的状态信息汇报给 Zookeeper 集群，其实就是在 Zookeeper 上创建了一个 Znode 节点，节点里保存了 NameNode 状态信息。

  当 NameNode 失效后，ZKFC 检测到报告给 Zookeeper，Zookeeper把对应的 Znode 删除掉，Standby ZKFC 发现没有 Active 状态的 NameNode 时，就会用 shell 命令将自己监控的 NameNode 改为 Active 状态，并修改 Znode 上的数据。
  Znode 是个临时的节点，临时节点特征是客户端的连接断了后就会把 znode 删除，所以当 ZKFC 失效时，也会导致切换 NameNode。

- DataNode 会将心跳信息和 Block 汇报信息同时发给两台 NameNode， DataNode 只接受 Active NameNode 发来的文件读写操作指令。

四十一、搭建HDFS HA 环境

\1. 服务器角色规划

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在 bigdata01、bigdata02、bigdata03 三台机器上分别创建目录 /opt/modules/hadoopha/ 用来存放 Hadoop HA 环境。

\2. 创建 HDFS HA 版本 Hadoop 程序目录

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

3. 新解压 Hadoop 2.5.0

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

4. 配置 Hadoop JDK 路径

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

5. 配置 hdfs-site.xml

```
<?xml version="1.0" encoding="UTF-8"?><configuration>
  <property>
    <!-- 为namenode集群定义一个services name -->
    <name>dfs.nameservices</name>
    <value>ns1</value>
  </property>
  <property>
    <!-- nameservice 包含哪些namenode，为各个namenode起名 -->
    <name>dfs.ha.namenodes.ns1</name>
    <value>nn1,nn2</value>
  </property>
  <property>
    <!--  名为nn1的namenode 的rpc地址和端口号，rpc用来和datanode通讯 -->
    <name>dfs.namenode.rpc-address.ns1.nn1</name>
    <value>bigdata-senior01.chybinmy.com:8020</value>
  </property>
  <property>
    <!-- 名为nn2的namenode 的rpc地址和端口号，rpc用来和datanode通讯  -->
    <name>dfs.namenode.rpc-address.ns1.nn2</name>
    <value>bigdata-senior02.chybinmy.com:8020</value>
  </property>
  <property>
    <!--名为nn1的namenode 的http地址和端口号，web客户端 -->
    <name>dfs.namenode.http-address.ns1.nn1</name>
    <value>bigdata-senior01.chybinmy.com:50070</value>
  </property>
  <property>
    <!--名为nn2的namenode 的http地址和端口号，web客户端 -->
    <name>dfs.namenode.http-address.ns1.nn2</name>
    <value>bigdata-senior02.chybinmy.com:50070</value>
  </property>
  <property>
    <!--  namenode间用于共享编辑日志的journal节点列表 -->
    <name>dfs.namenode.shared.edits.dir</name>
    <value>qjournal://bigdata-senior01.chybinmy.com:8485;bigdata-senior02.chybinmy.com:8485;bigdata-senior03.chybinmy.com:8485/ns1</value>
  </property>
  <property>
    <!--  journalnode 上用于存放edits日志的目录 -->
    <name>dfs.journalnode.edits.dir</name>
    <value>/opt/modules/hadoopha/hadoop-2.5.0/tmp/data/dfs/jn</value>
  </property>
  <property>
    <!--  客户端连接可用状态的NameNode所用的代理类 -->
    <name>dfs.client.failover.proxy.provider.ns1</name>
    <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
  </property>
  <property>
    <!--   -->
    <name>dfs.ha.fencing.methods</name>
    <value>sshfence</value>
  </property>
  <property>
    <name>dfs.ha.fencing.ssh.private-key-files</name>
    <value>/home/hadoop/.ssh/id_rsa</value>
  </property></configuration>
```

\6. 配置 core-site.xml

```
<?xml version="1.0" encoding="UTF-8"?><configuration>
  <property>
    <!--  hdfs 地址，ha中是连接到nameservice -->
    <name>fs.defaultFS</name>
    <value>hdfs://ns1</value>
  </property>
  <property>
    <!--  -->
    <name>hadoop.tmp.dir</name>
    <value>/opt/modules/hadoopha/hadoop-2.5.0/data/tmp</value>
  </property></configuration>
```

`hadoop.tmp.dir`设置 hadoop 临时目录地址，默认时，NameNode 和 DataNode 的数据存在这个路径下。

\7. 配置 slaves 文件

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

\8. 分发到其他节点

分发之前先将 share/doc 目录删除，这个目录中是帮助文件，并且很大，可以删除。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

\9. 启动HDFS HA集群

三台机器分别启动Journalnode。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

jps 命令查看是否启动。

\10. 启动Zookeeper

在三台节点上启动Zookeeper：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

\11. 格式化 NameNode

在第一台上进行 NameNode 格式化：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在第二台 NameNode 上：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

12. 启动 NameNode

在第一台、第二台上启动 NameNode：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

查看 HDFS Web 页面，此时两个 NameNode 都是 standby 状态。

切换第一台为 active 状态：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

可以添加上 forcemanual 参数，强制将一个 NameNode 转换为 active 状态。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

此时从 web 页面就看到第一台已经是 active 状态了。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

\13. 配置故障自动转移

利用 zookeeper 集群实现故障自动转移，在配置故障自动转移之前，要先关闭集群，不能在 HDFS运行期间进行配置。

**关闭 NameNode、DataNode、JournalNode、zookeeper**

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaPZfsIDmrMFibwO7lteFCeWDWDVRia6VN9bqpKL23TkvnmaWwP0gUnj5eegzicsRxrw6zcZywdVM5zAg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

**修改 hdfs-site.xml**

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaPZfsIDmrMFibwO7lteFCeWDRuN3hjDWAcLibTPdlZGibiaTIyEQ5icwtBt4TZlHZDguo9rT4PqokycKCQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

**修改 core-site.xml**

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaPZfsIDmrMFibwO7lteFCeWDkKISIhqKmC6e5lfrS8Ln3UibZYsQibzyLgtWcEMrtQz7G4aE9rt8hIhg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

**将 hdfs-site.xml 和 core-site.xml 分发到其他机器**

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaPZfsIDmrMFibwO7lteFCeWDZfcTRIWYicaI25b92u9icBfmNopdSgVvombY1KFwD5IIYiaUj7sJOibNtA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

**启动 zookeeper**

三台机器启动 zookeeper

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**创建一个 zNode**

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在 Zookeeper 上创建一个存储 namenode 相关的节点。

14. 启动 HDFS、JournalNode、zkfc

启动 NameNode、DataNode、JournalNode、zkfc

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

zkfc只针对 NameNode 监听。

四十二、测试 HDFS HA

1. 测试故障自动转移和数据是否共享

**在 nn1 上上传文件**

目前 bigdata-senior01节点上的 NameNode 是 Active 状态的。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaPZfsIDmrMFibwO7lteFCeWDz7CLgMcnJpEK0DbZUdl7GYAvbI3Ro9mjibbtoackdib4xiaMlfcDvsT9Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

**将 nn1 上的 NodeNode 进程杀掉**

```
[hadoop@bigdata-senior01 hadoop-2.5.0]$ kill -9 3364
```

nn1 上的 namenode 已经无法访问了。

**查看 nn2 是否是 Active 状态**

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaPZfsIDmrMFibwO7lteFCeWDYW1QhaZnycRr4zxAn7cy4C3NzSUca7bkrwjlKR6nO9qCicmqiaIuWaPg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

**在nn2上查看是否看见文件**

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaPZfsIDmrMFibwO7lteFCeWD4tSqLgQWauzt77GphsAnBRRCIOoibdPj6PLB3TLa0u9ib7flOxCjWmyQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

经以上验证，已经实现了 nn1 和 nn2 之间的文件同步和故障自动转移。

#### 第十二步  Hadoop 2.x YARN HA 部署

四十三、YARN HA原理

Hadoop2.4 版本之前，ResourceManager 也存在单点故障的问题，也需要实现HA来保证 ResourceManger 的高可也用性。

ResouceManager 从记录着当前集群的资源分配情况和 JOB 的运行状态，YRAN HA 利用 Zookeeper 等共享存储介质来存储这些信息来达到高可用。另外利用 Zookeeper 来实现 ResourceManager 自动故障转移。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

- MasterHADaemon：控制RM的 Master的启动和停止，和RM运行在一个进程中，可以接收外部RPC命令。
- 共享存储：Active Master将信息写入共享存储，Standby Master读取共享存储信息以保持和Active Master同步。
- ZKFailoverController：基于 Zookeeper 实现的切换控制器，由 ActiveStandbyElector 和 HealthMonitor 组成，ActiveStandbyElector 负责与 Zookeeper 交互，判断所管理的 Master 是进入 Active 还是 Standby；HealthMonitor负责监控Master的活动健康情况，是个监视器。
- Zookeeper：核心功能是维护一把全局锁控制整个集群上只有一个 Active的ResourceManager。

四十四、搭建 YARN HA 环境

1. 服务器角色规划

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

2. 修改配置文件yarn-site.xml

```
<?xml version="1.0" encoding="UTF-8"?><configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
  </property>
  <property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>106800</value>
  </property>
  <property>
    <!--  启用resourcemanager的ha功能 -->
    <name>yarn.resourcemanager.ha.enabled</name>
    <value>true</value>
  </property>
  <property>
    <!--  为resourcemanage ha 集群起个id -->
    <name>yarn.resourcemanager.cluster-id</name>
    <value>yarn-cluster</value>
  </property>
  <property>
    <!--  指定resourcemanger ha 有哪些节点名 -->
    <name>yarn.resourcemanager.ha.rm-ids</name>
    <value>rm12,rm13</value>
  </property>
  <property>
    <!--  指定第一个节点的所在机器 -->
    <name>yarn.resourcemanager.hostname.rm12</name>
    <value>bigdata-senior02.chybinmy.com</value>
  </property>
  <property>
    <!--  指定第二个节点所在机器 -->
    <name>yarn.resourcemanager.hostname.rm13</name>
    <value>bigdata-senior03.chybinmy.com</value>
  </property>
  <property>
    <!--  指定resourcemanger ha 所用的zookeeper 节点 -->
    <name>yarn.resourcemanager.zk-address</name>
    <value>bigdata-senior01.chybinmy.com:2181,bigdata-senior02.chybinmy.com:2181,bigdata-senior03.chybinmy.com:2181</value>
  </property>
  <property>
    <!--  -->
    <name>yarn.resourcemanager.recovery.enabled</name>
    <value>true</value>
  </property>
  <property>
    <!--  -->
    <name>yarn.resourcemanager.store.class</name>
    <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
  </property></configuration>
```

3. 分发到其他机器

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

4. 启动

在 bigdata-senior01 上启动 yarn：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在 bigdata-senior02、bigdata-senior03 上启动 resourcemanager：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

启动后各个节点的进程。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

Web 客户端访问 bigdata02 机器上的 resourcemanager 正常，它是 active 状态的。

http://bigdata-senior02.chybinmy.com:8088/cluster

访问另外一个 resourcemanager，因为他是 standby，会自动跳转到 active 的resourcemanager。

http://bigdata-senior03.chybinmy.com:8088/cluster

四十五、测试 YARN HA

5. 运行一个mapreduce job

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

6. 在 job 运行过程中，将 Active 状态的 resourcemanager 进程杀掉。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

\7. 观察另外一个 resourcemanager 是否可以自动接替。

bigdata02 的 resourcemanage Web 客户端已经不能访问，bigdata03 的 resourcemanage 已经自动变为active状态。

8. 观察 job 是否可以顺利完成。

而 mapreduce job 也能顺利完成，没有因为 resourcemanager 的意外故障而影响运行。

经过以上测试，已经验证 YARN HA 已经搭建成功。

#### 第十三步  HDFS Federation 架构部署

四十六、HDFS Federation 的使用原因

1. 单个 NameNode 节点的局限性

命名空间的限制。

NameNode 上存储着整个 HDFS 上的文件的元数据，NameNode 是部署在一台机器上的，因为单个机器硬件的限制，必然会限制 NameNode 所能管理的文件个数，制约了数据量的增长。

数据隔离问题。

整个 HDFS 上的文件都由一个 NameNode 管理，所以一个程序很有可能会影响到整个 HDFS 上的程序，并且权限控制比较复杂。

性能瓶颈。

单个NameNode 时 HDFS文件系统的吞吐量受限于单个 NameNode 的吞吐量。因为 NameNode 是个 JVM 进程，JVM 进程所占用的内存很大时，性能会下降很多。

2. HDFS Federation介绍

HDFS Federation 是可以在 Hadoop 集群中设置多个 NameNode，不同于 HA 中多个 NameNode 是完全一样的，是多个备份，Federation 中的多个 NameNode 是不同的，可以理解为将一个 NameNode 切分为了多个 NameNode，每一个 NameNode 只负责管理一部分数据。
HDFS Federation 中的多个 NameNode 共用 DataNode。

四十七、HDFS Federation 的架构图

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

四十八、HDFS Federation搭建

\1. 服务器角色规划

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

\2. 创建HDFS Federation 版本Hadoop程序目录

在bigdata01上创建目录/opt/modules/hadoopfederation /用来存放Hadoop Federation环境。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

\3. 新解压 Hadoop 2.5.0

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

\4. 配置 Hadoop JDK 路径

修改 hadoop-env.sh、mapred-env.sh、yarn-env.sh 文件中的 JDK 路径。

export JAVA_HOME=”/opt/modules/jdk1.7.0_67”

\5. 配置hdfs-site.xml

```
<configuration><property><!—配置三台NameNode -->
    <name>dfs.nameservices</name>
    <value>ns1,ns2,ns3</value>
  </property>
  <property><!—第一台NameNode的机器名和rpc端口，指定了NameNode和DataNode通讯用的端口号 -->
    <name>dfs.namenode.rpc-address.ns1</name>
    <value>bigdata-senior01.chybinmy.com:8020</value>
  </property>
   <property><!—第一台NameNode的机器名和rpc端口，备用端口号 -->
    <name>dfs.namenode.serviceerpc-address.ns1</name>
    <value>bigdata-senior01.chybinmy.com:8022</value>
  </property>
  <property><!—第一台NameNode的http页面地址和端口号 -->
    <name>dfs.namenode.http-address.ns1</name>
    <value>bigdata-senior01.chybinmy.com:50070</value>
  </property><property><!—第一台NameNode的https页面地址和端口号 -->
    <name>dfs.namenode.https-address.ns1</name>
    <value>bigdata-senior01.chybinmy.com:50470</value>
  </property>

  <property>
    <name>dfs.namenode.rpc-address.ns2</name>
    <value>bigdata-senior02.chybinmy.com:8020</value>
  </property>
   <property>
    <name>dfs.namenode.serviceerpc-address.ns2</name>
    <value>bigdata-senior02.chybinmy.com:8022</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.ns2</name>
    <value>bigdata-senior02.chybinmy.com:50070</value>
  </property>
    <property>
    <name>dfs.namenode.https-address.ns2</name>
    <value>bigdata-senior02.chybinmy.com:50470</value>
  </property>


  <property>
    <name>dfs.namenode.rpc-address.ns3</name>
    <value>bigdata-senior03.chybinmy.com:8020</value>
  </property>
   <property>
    <name>dfs.namenode.serviceerpc-address.ns3</name>
    <value>bigdata-senior03.chybinmy.com:8022</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.ns3</name>
    <value>bigdata-senior03.chybinmy.com:50070</value>
  </property>
    <property>
    <name>dfs.namenode.https-address.ns3</name>
    <value>bigdata-senior03.chybinmy.com:50470</value>
  </property></configuration>
```

\6. 配置 core-site.xml

```
<configuration><property>
    <name>hadoop.tmp.dir</name>
    <value>/opt/modules/hadoopha/hadoop-2.5.0/data/tmp</value></property></configuration>
```

hadoop.tmp.dir 设置 hadoop 临时目录地址，默认时，NameNode 和 DataNode 的数据存在这个路径下。

\7. 配置 slaves 文件

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

\8. 配置 yarn-site.xml

```
<configuration><property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
 </property>     
 <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>bigdata-senior02.chybinmy.com</value>
 </property>     
 <property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
 </property>     
 <property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>106800</value>
 </property>     </configuration>
```

\9. 分发到其他节点

分发之前先将 share/doc 目录删除，这个目录中是帮助文件，并且很大，可以删除。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

10. 格式化 NameNode

在第一台上进行 NameNode 格式化。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这里一定要指定一个集群 ID，使得多个 NameNode 的集群 ID 是一样的，因为这三个 NameNode 在同一个集群中，这里集群 ID 为 hadoop-federation-clusterId。

在第二台 NameNode 上。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在第三台 NameNode 上。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

11. 启动 NameNode

在第一台、第二台、第三台机器上启动 NameNode：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

启动后，用 jps 命令查看是否已经启动成功。

查看 HDFS Web 页面，此时三个 NameNode 都是 standby 状态。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

12. 启动 DataNode

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

启动后，用 jps 命令确认 DataNode 进程已经启动成功。

四十九、测试HDFS Federation

\1. 修改 core-site.xml

在bigdata-senior01机器上,修改core-site.xml文件，指定连接的NameNode是第一台NameNode。

[hadoop@bigdata-senior01 hadoop-2.5.0]$ vim etc/hadoop/core-site.xml

```
<configuration>
  <property>
     <name>fs.defaultFS</name>
     <value>hdfs://bigdata-senior01.chybinmy.com:8020</value>
  </property><property>
    <name>hadoop.tmp.dir</name>
    <value>/opt/modules/hadoopfederation/hadoop-2.5.0/data/tmp</value></property></configuration>
```

\2. 在 bigdate-senior01 上传一个文件到 HDFS

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

\3. 查看 HDFS 文件

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaPZfsIDmrMFibwO7lteFCeWDmnebu4VQTo3pXKYe4K0kLHNTTryckbDP4BqYQLXC8WCc63Zg6wgicHg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

可以看到，刚才的文件只上传到了 bigdate-senior01 机器上的 NameNode 上了，并没有上传到其他的 NameNode 上去。

这样，在 HDFS 的客户端，可以指定要上传到哪个 NameNode 上，从而来达到了划分 NameNode 的目的。