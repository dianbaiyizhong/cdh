

# CDH

cdh搭建文档 
## 目的
此文档是为了方便快速搭建一套测试使用的cdh环境，通过这个文档，傻瓜应该都能上手
## 扩展
包含kerberos的配置

### 组件版本信息

| 名称      | 版本号 |          备注          |
| --------- | -----: | :--------------------: |
| CDH       |  7.1.4 | 截至2020年11月23日最新 |
| HADOOP    |  3.1.1 |                        |
| HDFS      |  3.1.1 |                        |
| HBase     |  2.2.3 |                        |
| Hive      |  3.1.3 |                        |
| Kafka     |  2.4.1 |                        |
| ZooKeeper |  3.5.5 |                        |

## 基本软件环境准备

软件：VMware® Workstation 15 Pro
系统镜像：CentOS-7-x86_64-DVD-1810.iso



## 单机版搭建过程

### 第一阶段-前期基本配置

#### 下载好离线安装包

  请将以下需要的文件下载作为备用(推荐某雷下载,会比较快)

| rpm离线包                                                    |
| ------------------------------------------------------------ |
| https://archive.cloudera.com/cm7/7.1.4/allkeys.asc           |
| https://archive.cloudera.com/cm7/7.1.4/redhat7/yum/RPMS/x86_64/cloudera-manager-agent-7.1.4-6363010.el7.x86_64.rpm |
| https://archive.cloudera.com/cm7/7.1.4/redhat7/yum/RPMS/x86_64/cloudera-manager-daemons-7.1.4-6363010.el7.x86_64.rpm |
| https://archive.cloudera.com/cm7/7.1.4/redhat7/yum/RPMS/x86_64/cloudera-manager-server-7.1.4-6363010.el7.x86_64.rpm |
| https://archive.cloudera.com/cm7/7.1.4/redhat7/yum/RPMS/x86_64/cloudera-manager-server-db-2-7.1.4-6363010.el7.x86_64.rpm |
| https://archive.cloudera.com/cm7/7.1.4/redhat7/yum/RPMS/x86_64/enterprise-debuginfo-7.1.4-6363010.el7.x86_64.rpm |
| https://archive.cloudera.com/cm7/7.1.4/redhat7/yum/RPMS/x86_64/openjdk8-8.0+232_9-cloudera.x86_64.rpm |

  

| parcels包                                                    |
| ------------------------------------------------------------ |
| https://archive.cloudera.com/cdh7/7.1.4.0/parcels/CDH-7.1.4-1.cdh7.1.4.p0.6300266-el7.parcel |
| https://archive.cloudera.com/cdh7/7.1.4.0/parcels/CDH-7.1.4-1.cdh7.1.4.p0.6300266-el7.parcel.sha256 |
| https://archive.cloudera.com/cdh7/7.1.4.0/parcels/manifest.json |



| mysql驱动包,可以到mvn repository仓库下载,这里直接附上链接,下载后要将文件名改成mysql-connector-java.jar,而不是mysql-connector-java-5.1.49.jar |
| ------------------------------------------------------------ |
| https://repo1.maven.org/maven2/mysql/mysql-connector-java/5.1.49/mysql-connector-java-5.1.49.jar |






#### 设置静态IP
这一步其实可以省略，看你需要，我这里会将机器的静态ip设置为192.168.88.220

#### 配置主机名
这里配置为[single-cdh]
```sh
hostnamectl set-hostname single-cdh
```


####  配置host文件

```sh
vi /etc/hosts
```
记住，这里的hosts末尾有s的，不是host，是hosts

加入你的机器名和ip的映射关系

192.168.88.220 single-cdh

####  关闭防火墙

```sh
systemctl stop firewalld.service
systemctl disable firewalld.service
```

#### 配置免密登录
    该步骤单机版不知道需不需要,这里也做上吧

```sh
ssh-keygen
```
后面一路回车即可
#### 关闭selinux
```sh
setenforce 0
```



#### 调整时差

> 若centos的时区跟实际对不上,那就使用以下命令切换一下

```sh
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```





#### 恭喜完成第一阶段






------



### 第二阶段 安装MYSQL

#### 拉取MYSQL镜像

```sh
docker pull mysql:5.7
```
#### 运行MYSQL镜像

```sh
docker run --name mysql -e MYSQL_ROOT_PASSWORD=root -d --restart=always -p 3306:3306 mysql:5.7
```
#### 建库建表

```sql
CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

CREATE DATABASE hive DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

CREATE DATABASE ranger DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
```



------



### 第三阶段 配置离线安装环境
> 这里采用离线安装,否则使用网络安装,可能会因为网络问题而失败

#### 启动httpd

> 一般来说centos是自带的,如果没有,使用yum装一下就可以了

```sh
yum install httpd createrepo -y
```

启动

```shell
systemctl start httpd
systemctl enable httpd
```

#### 建立rpm本地源

创建目录

```sh
mkdir -p /var/www/html/cm7/7.1.4/
```

将备用的rpm包上传到/var/www/html/cm7/7.1.4/

```sh
mv cloudera-manager-* /var/www/html/cm7/7.1.4/
mv allkeys.asc /var/www/html/cm7/7.1.4/
mv openjdk8-8.0+232_9-cloudera.x86_64.rpm /var/www/html/cm7/7.1.4/
mv enterprise-debuginfo-7.1.4-6363010.el7.x86_64.rpm /var/www/html/cm7/7.1.4/
mv RPM-GPG-KEY-cloudera  /var/www/html/cm7/7.1.4/
```



上传结束后,运行命令,注意后面的点

```sh
createrepo .
```

制作repo源

```shell
cd /etc/yum.repos.d
vim cm7.repo
```

填写内容如下,注意地址跟你本机匹配,这里是single-cdh
```shell
[cm7]
name=cm7
baseurl=http://single-cdh/cm7/7.1.4
gpgcheck=0
enabled=1
```

#### 建立parcel目录

创建目录

```sh
mkdir -p /var/www/html/cdh7/7.1.4
```



```sh
mv CDH* /var/www/html/cdh7/7.1.4
mv manifest.json /var/www/html/cdh7/7.1.4
```

将备用的parcel包上传到/var/www/html/cdh7/7.1.4

> 这里不用使用命令createrepo .,这只是用来做文件服务器,又不是建立rpm本地源



### 安装JDK以及mysql驱动

#### 安装JDK

还记的yum源里有一个jdk安装包吗,去到他的目录,直接运行以下命令安装

```shell
rpm -ivh openjdk8-8.0+232_9-cloudera.x86_64.rpm
```

安装好后,应该会出现以下目录

```shell
/usr/java
```



#### 上传mysql驱动包

上传到以下的目录

```shell
/usr/share/java
```

```sh
mv mysql-connector-java.jar /usr/share/java
```







------





### 安装Cloudera Manager Server

#### yum安装

```shell
yum -y install cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server
```

#### 导入TLS

直接运行即可

```sh
JAVA_HOME=/usr/java/jdk1.8.0_232-cloudera /opt/cloudera/cm-agent/bin/certmanager setup --configure-services
```



#### 初始化数据库

直接执行下面语句,后面两个参数是数据库账号和密码

```shell
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql scm root root
```





若你走到了这里,建议此时此刻保存一下虚拟机的快照,免得后面的操作有问题可以快速回复

------



### 开始启动

#### 启动cloudera-scm-server
启动命令：

```shell
systemctl start cloudera-scm-server.service
```
查看日志
```shell
tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
```

停止
```shell
systemctl stop cloudera-scm-server.service
```



#### 启动agent

```sh
vi /etc/cloudera-scm-agent/config.ini
```

修改server_host=single-cdh

启动

```sh
systemctl start cloudera-scm-agent
```

查看状态

```sh
systemctl status cloudera-scm-agent
```

查看日志

```sh
tail -f /var/log/cloudera-scm-agent/cloudera-scm-agent.log
```





### 管理平台新建集群

打开浏览器 http://192.168.88.220:7180/

![](.\images\微信截图_20210517212427.png)

选择60天试用

![](.\images\微信截图_20210517212609.png)

扫描并选中主机名

![](.\images\微信截图_20210517212748.png)





选择配置parcel源

![](images\微信截图_20210520203910.png)



将url全部删掉，留下我们自己配置的仓库url，输入后点击“save & Verify Configuration”

![](images\微信截图_20210517213219.png)

这里一定要选择7.1.4的版本，如果没有这个版本选择，那估计是前面的步骤哪里没做好了，只能重做

![](images\微信截图_20210520202749.png)





这里输入机器的账号密码，这里都是root



![](images\微信截图_20210517214245.png)



如无意外，会来到这个页面，这个页面下载不会很慢，因为我们自己配置了下载仓库，耐心等待

![](images\微信截图_20210520203114.png)



![](images\微信截图_20210520205320.png)

选择自定义服务

![](images\微信截图_20210520205337.png)



这里我先选择安装hdfs、hive、zookeeper和yarn，其他的我们再看

![](D:\project\cdh\images\微信截图_20210520205358.png)





![](images\微信截图_20210520205501.png)



这里有一点要注意，hive这里的WebHcat和HiveServer2要选择好机器

![](images\微信截图_20210521132919.png)







数据库选择hive和rman，点击测试连接，没问题后点击继续

![](images\微信截图_20210520205703.png)



