## Docker 安装 hadoop3.2.0

### 1. 下载包

Hadoop：[Apache Hadoop](https://hadoop.apache.org/releases.html)，[hadoop-3.2.0.tar.gz](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.2.0/hadoop-3.2.0.tar.gz)

Java8：[Java SE 8u211](https://www.oracle.com/technetwork/java/javase/downloads/index.html)

```sh
➜  pwd
/Users/yuanfanbin/workspace/bug/docker/hadoop/data
➜  ls
hadoop-3.2.0.tar.gz     jdk-8u211-linux-x64.rpm
```

### 2. 基于CentOS6.6的镜像创建一个基础linux服务环境

```sh
➜  docker run -it --name hadoop-basenv -v /Users/yuanfanbin/workspace/bug/docker/hadoop/data:/data centos:6.6 /bin/bash
[root@65f91d092d3c /]#

# 安装Vim
[root@65f91d092d3c /]# yum install vim-enhanced -y

# 安装SSH
[root@65f91d092d3c /]# yum install openssh-server openssh-clients -y

# 安装Service
[root@65f91d092d3c /]# yum install initscripts -y

# 安装JDK
[root@65f91d092d3c /]# cd /data
[root@65f91d092d3c data]# rpm -ivh jdk-8u211-linux-x64.rpm
[root@65f91d092d3c data]# find / -name "rt.jar"     # 查找JDK安装目录

# 配置免密登陆
[root@65f91d092d3c data]# ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
[root@65f91d092d3c data]# cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
[root@65f91d092d3c data]# service sshd start
Generating SSH2 RSA host key:                              [  OK  ]
Generating SSH1 RSA host key:                              [  OK  ]
Generating SSH2 DSA host key:                              [  OK  ]
Starting sshd:                                             [  OK  ]

# 配置JDK环境
[root@65f91d092d3c data]# cat /etc/profile
...
export JAVA_HOME=/usr/java/jdk1.8.0_211-amd64/
export JRE_HOME=/usr/java/jdk1.8.0_211-amd64/jre/
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

[root@65f91d092d3c data]# source /etc/profile
```

### 3. 安装Hadoop

```sh
# 解压 hadoop
[root@65f91d092d3c data]# yum install tar -y
[root@65f91d092d3c data]# mkdir /usr/local/hadoop
[root@65f91d092d3c data]# cd /usr/local/hadoop
[root@65f91d092d3c hadoop]# cp /data/hadoop-3.2.0.tar.gz .
[root@65f91d092d3c hadoop]# tar -zxvf hadoop-3.2.0.tar.gz

# 新增JAVA环境变量
[root@65f91d092d3c hadoop]# cat /etc/profile
...
export HADOOP_HOME=/usr/local/hadoop/hadoop-3.2.0
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

# 创建工作目录
[root@65f91d092d3c hadoop]# mkdir tmp
[root@65f91d092d3c hadoop]# mkdir -p hdfs/name
[root@65f91d092d3c hadoop]# mkdir hdfs/data
```

#### 3.0 修改 `hadoop-env.sh` & `yarn-env.sh`

```sh
[root@65f91d092d3c hadoop]# cat hadoop-env.sh
...
export JAVA_HOME=/usr/java/jdk1.8.0_211-amd64/
...

[root@65f91d092d3c hadoop]# cat yarn-env.sh
...
export JAVA_HOME=/usr/java/jdk1.8.0_211-amd64/
...
```

#### 3.1 配置 `core-site.xml` (etc/hadoop目录下)

```xml
<configuration>
    <property>
        <name> fs.default.name </name>
        <value>hdfs://master:9000</value>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://master:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/usr/local/hadoop/tmp</value>
    </property>
</configuration>
```

#### 3.2 配置 `hdfs-site.xml`

```xml
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/hadoop/hdfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/hadoop/hdfs/data</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
        <description>副本个数（每个本分割的文件会存储在几台datanode上，默认是3），这个数量应该小于datanode机器数</description>
    </property>
    <property>
        <name>dfs.permissions.enabled</name>
        <value>false</value>
    </property>
    <property>
        <name>dfs.namenode.http-address</name>
        <value>master:9870</value>
        <description>master机器上开放端口9870，供外部访问web页面（NameNode HTTP UI），查看集群情况</description>
    </property>
    <property>
        <name>dfs.datanode.http.address</name>
        <value>master:9864</value>
        <description>slave机器上开放端口9864，供外部访问web页面（DataNode HTTP UI）</description>
    </property>
</configuration>
```

#### 3.3 配置 `mapred-site.xml`

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>0.0.0.0:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>0.0.0.0:19888</value>
    </property>
</configuration>
```

#### 3.4 配置 `yarn-site.xml`

```xml
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>master</value>
        <description>yarn resourcemanager hostname is master</description>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
        <description>just mapreduce_shuffle can run MapReduce</description>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>0.0.0.0:8088</value>
    </property>
    <property>
        <name>yarn.application.classpath</name>
        <value>
            /usr/local/hadoop/hadoop-3.2.0/etc/hadoop,
            /usr/local/hadoop/hadoop-3.2.0/etc/hadoop,
            /usr/local/hadoop/hadoop-3.2.0/share/hadoop/common/lib/*,
            /usr/local/hadoop/hadoop-3.2.0/share/hadoop/common/*,
            /usr/local/hadoop/hadoop-3.2.0/share/hadoop/hdfs,
            /usr/local/hadoop/hadoop-3.2.0/share/hadoop/hdfs/lib/*,
            /usr/local/hadoop/hadoop-3.2.0/share/hadoop/hdfs/*,
            /usr/local/hadoop/hadoop-3.2.0/share/hadoop/mapreduce/*,
            /usr/local/hadoop/hadoop-3.2.0/share/hadoop/yarn,
            /usr/local/hadoop/hadoop-3.2.0/share/hadoop/yarn/lib/*,
            /usr/local/hadoop/hadoop-3.2.0/share/hadoop/yarn/*,
            /usr/java/jdk1.8.0_211-amd64/lib/tools.jar
        </value>
    </property>
    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>
</configuration>
```

#### 3.5 配置 `workers`

```conf
node1
node2
node3
```

#### 3.6 HDFS & YARN 环境变量

```sh
[root@65f91d092d3c hadoop]# cat /etc/profile
...
export HDFS_NAMENODE_USER="root"
export HDFS_DATANODE_USER="root"
export HDFS_SECONDARYNAMENODE_USER="root"
export YARN_RESOURCEMANAGER_USER="root"
export YARN_NODEMANAGER_USER="root"
```

#### 3.7 生成docker image

```sh
[root@65f91d092d3c hadoop]#         # 退出container ⌘ + P ⌘ + Q

# 生成hadoop基础镜像
➜  docker commit -m "hadoop: init" hadoop-basenv yuanfanbin/hadoop-distributed-base
```

#### 3.8 docker-compose.yaml

```yaml
version: "3.0"
services:
    hadoop-master:
        container_name: hadoop-master
        hostname: master
        image: yuanfanbin/hadoop-distributed-base
        tty: true
        ports:
          - 9870:9870
          - 8088:8088
        volumes:
          - /Users/yuanfanbin/workspace/bug/docker/hadoop/data:/data
    hadoop-node1:
        container_name: hadoop-node1
        hostname: node1
        image: yuanfanbin/hadoop-distributed-base
        tty: true
        volumes:
          - /Users/yuanfanbin/workspace/bug/docker/hadoop/data:/data
    hadoop-node2:
        container_name: hadoop-node2
        hostname: node2
        image: yuanfanbin/hadoop-distributed-base
        tty: true
        volumes:
          - /Users/yuanfanbin/workspace/bug/docker/hadoop/data:/data
    hadoop-node3:
        container_name: hadoop-node3
        hostname: node3
        image: yuanfanbin/hadoop-distributed-base
        tty: true
        volumes:
          - /Users/yuanfanbin/workspace/bug/docker/hadoop/data:/data
```

```sh
# 启动hadoop集群
➜  docker-compose up -d
➜  docker container ls
CONTAINER ID    IMAGE                                COMMAND        CREATED             STATUS              PORTS                                            NAMES
b44c386b8b17    yuanfanbin/hadoop-distributed-base   "/bin/bash"    7 minutes ago       Up 7 minutes                                                         hadoop-node3
9168b0358297    yuanfanbin/hadoop-distributed-base   "/bin/bash"    7 minutes ago       Up 7 minutes                                                         hadoop-node2
5a87d16c9f7c    yuanfanbin/hadoop-distributed-base   "/bin/bash"    7 minutes ago       Up 7 minutes        0.0.0.0:8088->8088/tcp, 0.0.0.0:9870->9870/tcp   hadoop-master
56f9f28c2d6f    yuanfanbin/hadoop-distributed-base   "/bin/bash"    7 minutes ago       Up 7 minutes                                                         hadoop-node1
65f91d092d3c    centos:6.6                           "/bin/bash"    About an hour ago   Up About an hour                                                     hadoop-basenv

# 登陆集群，启动sshd + 更改/etc/hosts
➜  docker exec -it hadoop-master /bin/bash
[root@master /]# cat /etc/hosts
172.23.0.4	master
172.23.0.2	node1
172.23.0.3	node2
172.23.0.5	node3
[root@master /]# service sshd start
Starting sshd:                                             [  OK  ]

# 启动集群
[root@master /]# source /etc/profile
[root@master /]# hadoop namenode -format
[root@master /]# start-all.sh
```

### 参考资料

1. [Docker搭建Hadoop集群](https://www.jianshu.com/p/7ab2b6168cc9)
2. [hadoop-3.0.0集群搭建](https://my.oschina.net/u/3163032/blog/1622221)
3. [hadoop 8088端口无法访问](https://blog.csdn.net/u013761049/article/details/81780245)
4. [HDFS_NAMENODE_USER, HDFS_DATANODE_USER & HDFS_SECONDARYNAMENODE_USER not defined](https://stackoverflow.com/questions/48129029/hdfs-namenode-user-hdfs-datanode-user-hdfs-secondarynamenode-user-not-defined)
