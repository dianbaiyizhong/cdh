# 单机版HIVE测试环境搭建
单机配置hadoop+hive+yarn+hiveserver2













https://blog.csdn.net/winterking3/article/details/87617607
https://blog.csdn.net/winterking3/article/list/3
https://blog.csdn.net/leanaoo/article/details/83351240



### 准备一份日志文件emp.txt

```
7369,SMITH,CLERK,7902,1980-12-17,800.00,20
7499,ALLEN,SALESMAN,7698,1981-2-20,1600.00,300.00,30
7521,WARD,SALESMAN,7698,1981-2-22,1250.00,500.00,30
7566,JONES,MANAGER,7839,1981-4-2,2975.00,20
7654,MARTIN,SALESMAN,7698,1981-9-28,1250.00,1400.00,30
7698,BLAKE,MANAGER,7839,1981-5-1,2850.00,30
7782,CLARK,MANAGER,7839,1981-6-9,2450.00,10
7788,SCOTT,ANALYST,7566,1987-4-19,3000.00,20
7839,KING,PRESIDENT,,1981-11-17,5000.00,10
7844,TURNER,SALESMAN,7698,1981-9-8,1500.00,0.00,30
7876,ADAMS,CLERK,7788,1987-5-23,1100.00,20
7900,JAMES,CLERK,7698,1981-12-3,950.00,30
7902,FORD,ANALYST,7566,1981-12-3,3000.00,20
7934,MILLER,CLERK,7782,1982-1-23,1300.00,10
```



```sql
create table emp(
empno int,
ename string,
job string,
mgr int,
hiredate string,
sal double,
deptno int
)
row format delimited fields terminated by '\t'
```





# HiveStatment获取日志

hive-site需要配置

```xml
<property>
  <name>hive.async.log.enabled</name>
  <value>false</value>
</property>

```







https://blog.csdn.net/weixin_43283487/article/details/106600166


https://www.jianshu.com/p/b5c9c78ffd07

https://www.keisuke.dev/2020/06/11/setup_hive3.html


## 安装hadoop

vim /etc/profile
```sh

export JAVA_HOME=/usr/local/bigdata/jdk1.8.0_45
export PATH=$PATH:$JAVA_HOME/bin
export HADOOP_HOME=/usr/local/bigdata/hadoop-3.3.1
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HIVE_HOME=/usr/local/bigdata/apache-hive-3.1.2-bin
PATH=$PATH:$HOME/bin:$HIVE_HOME/bin
export TEZ_HOME=/usr/local/bigdata/apache-tez-0.9.2-bin
export HADOOP_CLASSPATH=${TEZ_HOME}/conf:${TEZ_HOME}/*:${TEZ_HOME}/lib/*

```









关闭防火墙

```sh
systemctl stop firewalld.service
systemctl disable firewalld.service
```
配置hosts文件

```sh
192.168.68.220 standalone
```

配置ssh
```sh
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```


hdfs-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
      <name>dfs.namenode.http.address</name>
       <value>standalone:9870</value>
    </property>
     <property>
      <name>dfs.namenode.decommission.interval</name>
      <value>30</value>
  </property>
  <property>
      <name>dfs.client.datanode-restart.timeout</name>
      <value>30</value>
  </property>
</configuration>

```




hadoop-env.sh
```sh
export JAVA_HOME=/usr/local/bigdata/jdk1.8.0_45
export HADOOP_HOME=/usr/local/bigdata/hadoop-3.3.1
```

core-site.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration> 
  <property> 
    <name>fs.defaultFS</name>  
    <value>hdfs://standalone:9000</value> 
  </property>  
  <property> 
    <name>hadoop.proxyuser.root.hosts</name>  
    <value>*</value> 
  </property>  
  <property> 
    <name>hadoop.proxyuser.root.groups</name>  
    <value>*</value> 
  </property> 
</configuration>

```





yarn-site.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<configuration> 
  <property> 
    <name>yarn.nodemanager.aux-services</name>  
    <value>mapreduce_shuffle</value>  
    <final>true</final> 
  </property>  
  <property> 
    <name>yarn.nodemanager.vmem-check-enabled</name>  
    <value>false</value> 
  </property>  
  <property> 
    <name>yarn.nodemanager.local-dirs</name>  
    <value>/root/hadoop-tmp/nm-local-dir</value> 
  </property> 
</configuration>

```



mapred-site.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<configuration> 
  <property> 
    <name>mapreduce.framework.name</name>  
    <value>yarn</value>  
    <final>true</final>  
    <description>The runtime framework for executing MapReduce jobs</description> 
  </property>  
  <property> 
    <name>yarn.app.mapreduce.am.env</name>  
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value> 
  </property>  
  <property> 
    <name>mapreduce.map.env</name>  
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value> 
  </property>  
  <property> 
    <name>mapreduce.reduce.env</name>  
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value> 
  </property> 
</configuration>

```


修改sbin/start-dfs.sh和sbin/stop-dfs.sh,在文件头加入以下内容
```sh
HDFS_DATANODE_USER=root
HADOOP_SECURE_DN_USER=root
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
```

修改sbin/start-yarn.sh和sbin/stop-yarn.sh,在文件头加入以下内容
```sh
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=root
YARN_NODEMANAGER_USER=root
```


格式化
```sh
hdfs namenode -format
```
启动和关闭
```sh
start-all.sh
stop-all.sh
```

```访问连接
9870和8088端口
```





## 安装HIVE

hive-env.sh
```sh
HADOOP_HOME=/usr/local/bigdata/hadoop-3.3.1/
```



hive-site.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<configuration> 
  <property> 
    <name>javax.jdo.option.ConnectionURL</name>  
    <value>jdbc:mysql://192.168.68.220:3306/hive?createDatabaseIfNotExist=true&amp;serverTimezone=Asia/Shanghai&amp;useSSL=false</value> 
  </property>  
  <property> 
    <name>javax.jdo.option.ConnectionDriverName</name>  
    <value>com.mysql.jdbc.Driver</value> 
  </property>  
  <property> 
    <name>javax.jdo.option.ConnectionUserName</name>  
    <value>root</value> 
  </property>  
  <property> 
    <name>javax.jdo.option.ConnectionPassword</name>  
    <value>root</value> 
  </property>  
  <property> 
    <name>hive.metastore.schema.verification</name>  
    <value>false</value> 
  </property>  
  <property> 
    <name>system:user.name</name>  
    <value>root</value>  
    <description>user name</description> 
  </property>  
  <property> 
    <name>hive.execution.engine</name>  
    <value>tez</value> 
  </property> 
</configuration>
```


上传mysql-connector驱动包到lib目录下

初始化数据库
```sh
schematool -dbType mysql -initSchema
```

## 安装Tez

```sh
hadoop fs -mkdir -p /apps/tez-0.9.2
hadoop fs -copyFromLocal tez.tar.gz /apps/tez-0.9.2
```

tez-site.xml
```xml
<configuration>
    <property>
        <name>tez.lib.uris</name>
        <value>${fs.defaultFS}/apps/tez-0.9.2/tez.tar.gz</value>
    </property>
</configuration>
```

```sh
hiveserver2 -hiveconf hive.execution.engine=mr
```


beeline

```sh
!connect jdbc:hive2://localhost:10000/;
```
账号是root，密码是空