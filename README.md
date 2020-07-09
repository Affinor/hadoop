**hadoop集群搭建**

1、准备3台SentOS7虚拟机，并安装好jdk1.8运行环境

192.168.80.129

192.168.80.130

192.168.80.132

2、下载hadoop-2.7.2:https://archive.apache.org/dist/hadoop/common/hadoop-2.7.2/

3、解压

tar -zxvf hadoop-2.7.2.tar.gz -C /home/db/hadoop/

4、将hadoop添加到环境变量

vim /etc/profifile

```
export HADOOP_HOME=/home/db/hadoop/hadoop-2.7.2 
export PATH=$PATH:$HADOOP_HOME/bin
```

source /etc/profifile

5、测试

hadoop version 

6、配置

vim hadoop-env.sh

```
//文件末尾 
export JAVA_HOME=/opt/module/jdk1.8.0_231
```

vim core-site.xml

```xml
<!-- 指定HDFS中NameNode的地址 -->
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://192.168.80.129:9000</value> 
</property> 
<!-- 指定hadoop运行时产生文件的存储目录 --> 
<property>
    <name>hadoop.tmp.dir</name> 
    <value>/home/db/hadoop/hadoop-2.7.2/data/tmp</value> 
</property>
```

vim hdfs-site.xml

```xml
<property>
    <name>dfs.replication</name> 
    <value>3</value> 
</property> 
<!--secondarynamenode的地址--> 
<property> 
    <name>dfs.namenode.secondary.http-address</name> 
    <value>192.168.80.132:50090</value>
</property> 
<property> 
    <name>dfs.name.dir</name>
    <value>/home/db/hadoop/hadoop-2.7.2/name/</value>
</property> 
<property> 
    <name>dfs.data.dir</name> 
    <value>/home/db/hadoop/hadoop-2.7.2/data/</value> 
</property>
```

vim yarn-site.xml

```xml
<!-- Site specific YARN configuration properties --> 
<!--NodeManager上运行的附属服务。需配置成mapreduce_shuffle，才可运行MapReduce程序>-->
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value> 
</property> 
<!-- 指定YARN的ResourceManager的地址 -->
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>192.168.80.130</value>
</property> 
<property> 
    <name>yarn.nodemanager.resource.memory-mb</name> 
    <value>1024</value>
</property>
```

cp mapred-site.xml.template mapred-site.xml 

mapred-site.xml

```xml
<!-- 指定mr运行在yarn上 --> 
<property> 
    <name>mapreduce.framework.name</name> 
    <value>yarn</value>
</property>
```

7、配置集群中从节点信息

vim slaves

```
192.168.80.129
192.168.80.130
192.168.80.132
```

8、分发文件

scp -r hadoop-2.7.2 192.168.80.130:/home/db/hadoop/

scp -r hadoop-2.7.2 192.168.80.132:/home/db/hadoop/

9、如果集群是第一次启动，需要格式化**NameNode（格式化只进行一次！）**

hadoop namenode -format

10、集群启动

start-dfs.sh （在namenode节点启动） 

start-yarn.sh  （在resourcemanager节点启动） 

11、测试

http://192.168.80.130:8088/ 

http://192.168.80.132:50090/status.html



**HBase集群搭建**

1、下载安装包

http://archive.apache.org/dist/hbase/1.3.1/

hbase-1.3.1-bin.tar.gz

2、规划安装目录

mkdir -p /home/db/hbase/

3、上传安装包到服务器

4、解压安装包到指定的规划目录

tar -zxvf hbase-1.3.1-bin.tar.gz -C /home/db/hbase/

5、修改配置文件

- 需要把hadoop中的配置core-site.xml 、hdfs-site.xml拷贝到hbase安装目录下的conf文件夹中

- 修改conf目录下配置文件

修改 hbase-env.sh

cd /home/db/hbase/hadoop-2.7.2/etc/hadoop; 

cp core-site.xml hdfs-site.xml /home/db/hbase/hbase-1.3.1/conf/#添加java环境变量 

export JAVA_HOME=/opt/module/jdk1.8.0_231 

\#指定使用外部的zk集群 

export HBASE_MANAGES_ZK=FALSE

vim hbase-site.xml

```xml
<configuration> 

<!-- 指定hbase在HDFS上存储的路径 --> 

<property>

<name>hbase.rootdir</name> 

<value>hdfs://192.168.80.129:9000/hbase</value> 

</property> 

<!-- 指定hbase是分布式的 --> 

<property>

<name>hbase.cluster.distributed</name> 

<value>true</value> 

</property> 

<!-- 指定zk的地址，多个用“,”分割 --> 

<property>

<name>hbase.zookeeper.quorum</name> 

<value>192.168.80.129:2181,192.168.80.130:2181,192.168.80.132:2181</value> 

</property> 

</configuration>
```

修改regionservers文件

vim regionserver

```
192.168.80.129
192.168.80.130
192.168.80.132
```

vim backup-masters

```
192.168.80.129
```

6、配置hbase的环境变量

export HBASE_HOME=/home/db/hbase/hbase-1.3.1

export PATH=$PATH:$HBASE_HOME/bin 

7、分发hbase目录和环境变量到其他节点

scp -r hbase-1.3.1 192.168.80.130:/home/db/hbase/ 

scp -r hbase-1.3.1 192.168.80.132:/home/db/hbase/

8、让所有节点的hbase环境变量生效

在所有节点执行

source /etc/profifile

**hbase集群的启动和停止**

前提条件：先启动hadoop和zk集群

- 启动hbase

  start-hbase.sh

- 停止hbase

  stop-hbase.sh

**hbase集群的web管理界面**

启动好hbase集群之后，可以访问地址：http://192.168.80.129:16010