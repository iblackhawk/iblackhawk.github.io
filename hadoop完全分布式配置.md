1.以一台机器为原型克隆n台虚拟机（n>2）

2.配置网络

3.配置 Hostname

例如：

BigData02 配置 hostname 为 bigdata-senior02.chybinmy.com

BigData03 配置 hostname 为 bigdata-senior03.chybinmy.com



4.配置 hosts

BigData01、BigData02、BigData03 三台机器 hosts 都配置为：

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEv30e0ZTnbNcF37XE7SEIwjmiceILJkdJ9WBibuGkr6fKmSEPzt73iaAoaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

**5.在第一台机器上安装新的 Hadoop**

我们采用先在第一台机器上解压、配置 Hadoop，然后再分发到其他两台机器上的方式来安装集群。

6. 解压 Hadoop 目录：

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvbRfgKqIu9vvxymSPXtGHQfpVYzfCyhknGOI5URTFwEY6qByZGvLWLA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

7. 配置 Hadoop JDK 路径修改 hadoop-env.sh、mapred-env.sh、yarn-env.sh 文件中的 JDK 路径：

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvpoPpkkj7Xuza38j8wuPOKISiauibr67guQaE6cLRtdM1zhEqJvsYk6qg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

8. 配置 core-site.xml

![img](https://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvNaG4YSU0leBvDvto9ia3uV8JyeXIo1c47TlTp8UMSV5RPhYnjTSxI4g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&retryload=1)

fs.defaultFS 为 NameNode 的地址。

hadoop.tmp.dir 为 hadoop 临时目录的地址，默认情况下，NameNode 和 DataNode 的数据文件都会存在这个目录下的对应子目录下。应该保证此目录是存在的，如果不存在，先创建。

9. 配置 hdfs-site.xml

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvKrEYcolIltNLojHHfhdVC5VVGEyR42hNSib5mvZjvzM7MNwnz7vgBiaQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

dfs.namenode.secondary.http-address 是指定 secondaryNameNode 的 http 访问地址和端口号，因为在规划中，我们将 BigData03 规划为 SecondaryNameNode 服务器。

所以这里设置为：bigdata-senior03.chybinmy.com:50090

10. 配置 slaves

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvw5rgCsB2qO4CnZZDIxRvicQggbZOyfR3xGhIEnDhK2QZdFbOtvnaSEA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

slaves 文件是指定 HDFS 上有哪些 DataNode 节点。

11. 配置 yarn-site.xml

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvibEvcytqLLBHg95zia8sibMddCuliaPlw17AicjymA4ADIj4B9nMxxGA0HQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

根据规划`yarn.resourcemanager.hostname`这个指定 resourcemanager 服务器指向`bigdata-senior02.chybinmy.com`。

`yarn.log-aggregation-enable`是配置是否启用日志聚集功能。

`yarn.log-aggregation.retain-seconds`是配置聚集的日志在 HDFS 上最多保存多长时间。

12. 配置 mapred-site.xml

从 mapred-site.xml.template 复制一个 mapred-site.xml 文件。

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvfpnkYtQ0FB2oJD4MHMMQtlKLpKZKTlHvrujKWzlApaGQ46yS7tJ1Jg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

mapreduce.framework.name 设置 mapreduce 任务运行在 yarn 上。

mapreduce.jobhistory.address 是设置 mapreduce 的历史服务器安装在BigData01机器上。

mapreduce.jobhistory.webapp.address 是设置历史服务器的web页面地址和端口号。

**二十八、设置 SSH 无密码登录**

Hadoop 集群中的各个机器间会相互地通过 SSH 访问，每次访问都输入密码是不现实的，所以要配置各个机器间的

SSH 是无密码登录的。

1. 在 BigData01 上生成公钥

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvFYQ4FAJ9IcCjcWmoY3p0IlvYcbhFrkw91JMGbs9qOI9HmaNI3cA2yg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

一路回车，都设置为默认值，然后再当前用户的Home目录下的`.ssh`目录中会生成公钥文件`（id_rsa.pub）`和私钥文件`（id_rsa）`。

2. 分发公钥

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvQ8hKthjRyLfN0abxzDkh08SHtYaRNCgrVQ8ib1zlrvibBXPrj9eRFicjg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

3. 设置 BigData02、BigData03 到其他机器的无密钥登录

同样的在 BigData02、BigData03 上生成公钥和私钥后，将公钥分发到三台机器上。

**二十九、分发 Hadoop 文件**

1. 首先在其他两台机器上创建存放 Hadoop 的目录

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEv8VLvCTEuMXTjYK4enDmaS7OicpKmiaPDFib3xPjCFibEbtzWM3QW9NRbwA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

2. 通过 Scp 分发

Hadoop 根目录下的 share/doc 目录是存放的 hadoop 的文档，文件相当大，建议在分发之前将这个目录删除掉，可以节省硬盘空间并能提高分发的速度。

doc目录大小有1.6G。

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvgbGQ8RWhnqH0pibUIVvib8susxXS2ukmlsYClicU9GmW96GcSUk7z9yOQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

**三十、格式 NameNode**

在 NameNode 机器上执行格式化：

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvibp6LibsTz8SMRAzDxNlEP6b2nhxs0MbzCJNOVydaxjxLgqhQwUb6K7Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

**注意：**

如果需要重新格式化 NameNode，需要先将原来 NameNode 和 DataNode 下的文件全部删除，不然会报错，NameNode 和 DataNode 所在目录是在`core-site.xml`中`hadoop.tmp.dir`、`dfs.namenode.name.dir`、`dfs.datanode.data.dir`属性配置的。

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvuUHtHOdEY3FKTUx58zOQD4ocsrPic6lCOKfsWbiafiaavVibf2uJJpeoHA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

因为每次格式化，默认是创建一个集群ID，并写入 NameNode 和 DataNode 的 VERSION 文件中（VERSION 文件所在目录为 dfs/name/current 和 dfs/data/current），重新格式化时，默认会生成一个新的集群ID,如果不删除原来的目录，会导致 namenode 中的 VERSION 文件中是新的集群 ID，而 DataNode 中是旧的集群 ID，不一致时会报错。

另一种方法是格式化时指定集群ID参数，指定为旧的集群ID。

**三十一、启动集群**

1.启动 HDFS

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvFzKRhtaWq0ibiblnaQ2bHBCeqMoNDuRQoxHaqPIa6FyiaDhJnx9JpMh6g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

2.启动 YARN

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvHlX4MN4n6Ig7iaOicU9hH5wNOEcfPu2vMeK7jStH2iczI9A3zpRFpHhpQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

在 BigData02 上启动 ResourceManager:

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEv4mtX3Aw0v76z06xVqUtzvMgqeCxojJGKcHULUHAAmNxvBKTnVibQjXQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

3.启动日志服务器

因为我们规划的是在 BigData03 服务器上运行 MapReduce 日志服务，所以要在 BigData03 上启动。

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvzY9t8uIzrQTSUBLpXP3WDEibbu3lheLLHbkLVicwEhzmhlOpTXU3iag5Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

4.查看 HDFS Web 页面

http://bigdata-senior01.chybinmy.com:50070/

5. 查看 YARN Web 页面

http://bigdata-senior02.chybinmy.com:8088/cluster

**三十二、测试 Job**

我们这里用 hadoop 自带的 wordcount 例子来在本地模式下测试跑mapreduce。

1.  准备 mapreduce 输入文件 wc.input

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvSSAPrAvtI1juSf8WNicibibURAiclK3PnRy9RF9KL6icSpwYYDnqt3wwDvQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

2. 在 HDFS 创建输入目录 input

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvZNMl9hLY8Fnd9ZkYAGR9V6e8tpJntQ5wF2JGTLFRWibibSPicoia13hNWg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

3. 将 wc.inpu t上传到 HDFS

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvS7AoPzMoLHic96FHic1vxbibdicOic3AxPB27eQWbAFUdlo95PibLQDvzUCA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

4. 运行 hadoop 自带的 mapreduce Demo

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvtjAZw6X68zruxw7eKibJh8nebCE5LYs65FrckP69KhUejibvQMGW8DnQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

5. 查看输出文件

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvx6r6CkdtWibkybsNV0FhvPEictKzqjcF9arbMESE8SVaIobOIPueqc6Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

