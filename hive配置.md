**修改配置文件**

​     解压安装文件到指定的的文件夹 /opt/hive

​      tar -zxf apache-hive-2.1.0-bin.tar.gz -C  opt/hive

**2.3.1  设置环境变量**

​          vi /etc/profile

​          ![img](http://images2015.cnblogs.com/blog/159642/201707/159642-20170725120036412-1423372815.png)

**2.3.2  修改hive-site.xml文件**

​          ![img](http://images2017.cnblogs.com/blog/159642/201707/159642-20170726164402656-653868153.png)

​          ![img](http://images2017.cnblogs.com/blog/159642/201707/159642-20170726164501781-1852853436.png)

 

​          

**2.3.3 修改hive-env.sh**

​         cp hive-env.sh.template  hive-env.sh

​         vi  hive-env.sh

​         ![img](http://images2015.cnblogs.com/blog/159642/201707/159642-20170725121909927-1145171090.png)

​         

**2.4 运行hive**

​      运行hive之前首先要确保meta store服务已经启动，

​      nohup hive --service metastore > metastore.log 2>&1 &

​      ![img](http://images2017.cnblogs.com/blog/159642/201707/159642-20170726155402546-1166220885.png)

​     如果需要用到远程客户端(比如 Tableau)连接到hive数据库，还需要启动hive service

​     nohup hive --service hiveserver2 > hiveserver2.log 2>&1 &

​     ![img](http://images2017.cnblogs.com/blog/159642/201707/159642-20170726155659578-1527911828.png)

​    然后由于配置过环境变量，可以直接在命令行中输入hive

​    ![img](http://images2017.cnblogs.com/blog/159642/201707/159642-20170726155836156-1878050728.png)

 

**2.5 测试hive是否可以正确使用**

**2.5.1 创建测试表dep**

​     ![img](http://images2017.cnblogs.com/blog/159642/201707/159642-20170726165311421-1909402269.png)

**2.5.2 通过Mysql查看创建的表**

​      ![img](http://images2017.cnblogs.com/blog/159642/201707/159642-20170726164157796-2140675966.png)

**2.5.3 通过UI页面查看创建的数据位**

**        访问 xxx.xxx.xxx.xxx:50070**

**        ![img](http://images2017.cnblogs.com/blog/159642/201707/159642-20170726165802093-1673183418.png)**





## 可能出现的错误：

1.Cannot find hadoop installation: $HADOOP_HOME or $HADOOP_PREFIX must be set or hadoop must be in the path

请仔细查看hive-env.sh  ，  hadoop-env.sh ，  echo $HADOOP_HOME
看下三者的hadoop_home是不是完全一样，注意小版本号。