# hive 数据的上传与保存

首先，将文件上传到hdfs目录中。

再进入hive

创建数据库，并在数据库中根据需要上传的文件格式创建表。

create database 数据库名;

create table 表名( name string ,age int);例如，姓名string，年龄 int。

# 导入到hive表

```
load data inpath '/home/dat0204.log' into table tb1;
将hdfs  home目录下的dat0204.log上传到 hive的表tb1中。
```

# 导出hive表

导出到本地目录（！存放目录原来文件会被覆盖，仅存一个hive存储文件）

```
hive> insert overwrite local directory '/home/wyp/wyp'
    > select * from wyp;
   
```

导出到hdfs目录

```
hive> insert overwrite directory '/home/wyp/hdfs'
    > select * from wyp;

```

将会在HDFS的/home/wyp/hdfs目录下保存导出来的数据。注意，和导出文件到本地文件系统的HQL少一个local，数据的存放路径就不一样了。