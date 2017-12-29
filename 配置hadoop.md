# 配置hadoop

```
1.vim /etc/profile在环境中追加hadoop环境配置

```

例如：

![img](https://segmentfault.com/img/remote/1460000010487542)

```
source /etc/profile 使得配置生效
2.接下来 配置 hadoop-env.sh、mapred-env.sh、yarn-env.sh 
文件的 JAVA_HOME参数

```

![img](https://segmentfault.com/img/remote/1460000010487544)

```
3.配置 core-site.xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

```
#此处不添加也可运行
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/mnt/hadoop-2.6.0/data/tmp </value>
    </property>
</configuration>
fs.defaultFS 为 NameNode 的地址。

hadoop.tmp.dir 为 hadoop 临时目录的地址，默认情况下，NameNode 和 DataNode 的数据文件都会存在这个目录下的对应子目录下。应该保证此目录是存在的，如果不存在，先创建。
```





## 配置启动HDFS

1. 配置 hdfs-site.xml

```

<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>

//dfs.replication 配置的是 HDFS存 储时的备份数量，因为这里是伪分布式环境只有一个节点，所以这里设置为1。
```

如果是完全分布式：

![img](http://mmbiz.qpic.cn/mmbiz_png/0vU1ia3htaaMNqpEVdFK3aXtApjKSZxEvKrEYcolIltNLojHHfhdVC5VVGEyR42hNSib5mvZjvzM7MNwnz7vgBiaQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

dfs.namenode.secondary.http-address 是指定 secondaryNameNode 的 http 访问地址和端口号，因为在规划中，我们将 BigData03 规划为 SecondaryNameNode 服务器。

所以这里设置为：bigdata-senior03.chybinmy.com:50090

2. 格式化 HDFS

![img](https://segmentfault.com/img/remote/1460000010487553)![img](https://segmentfault.com/img/remote/1460000010487552)

格式化是对 HDFS 这个分布式文件系统中的 DataNode 进行分块，统计所有分块后的初始元数据的存储在 NameNode 中。

```
执行代码  start-all.sh  启动所有服务
jps进行查看。

hdfs dfs -mkdir /input   HDFS 上创建目录
hdfs dfs -put 上传文件 /input

读取 HDFS 上的文件内容
hdfs dfs -cat 
```

![img](https://segmentfault.com/img/remote/1460000010487563)

```
从 HDFS上 下载文件到本地
hdfs dfs -get /目标目录

```

