---
title: "[Hadoop-1] Hadoop 설치"
author:
  name: Yeoni
  link: https://github.com/cotes2020
date: 2021-12-27 17:00:00 +0900
categories: [BigData, Hadoop]
tags: [Hadoop]
math: true
mermaid: true
---


# Hadoop 설치

---
## 환경 구성

| Tools                        | Version          |
|:-----------------------------|-----------------:|
| VirtualBox                   | 6.1              |
| CentOS                       | 7                |
| Java Version                 | 1.8              |
| Hadoop                       | 3.3.1            |

---
## 사용포트 정리

| Program                      | Port Number      |
|:-----------------------------|-----------------:|
| NameNode                     | 9870             |
| DataNode                     | 9864             |
| ResourceManager              | 8088             |
| NodeManager                  | 8042             |
| MapReduce JobHistory Server  | 19888            |

---
## 선행 작업

- VirtualBox 설치
- VirtualBox에 CentOS 설치
- CentOS에 java설치

---
## 설치

### 설치 위치
/home/yeon/dev

설치위치는 DEV_HOME으로 지정하여 ~/.bashrc에 변수로 환경변수 등록.

```console
[yeon@yeon-host lib]$ vi ~/.bashrc
```

```shell
...
# Develop Home
export DEV_HOME=/home/yeon/dev
# Library Home
export LIB_HOME=/home/yeon/lib
```
---
### Hadoop 3.3.1 다운로드 및 압축해제
```console
cd $LIB_HOME
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.1/hadoop-3.3.1.tar.gz
tar -xvzf hadoop-3.3.1.tar.gz
```
---
### Hadoop 심볼릭 링크 적용
```console
cd $DEV_HOME
ln -s $LIB_HOME/hadoop-3.3.1 hadoop
```

![Desktop View](/assets/img/contents/BigData/Hadoop/hadoop-setting/image1.png)
_Hadoop 심볼릭 링크 적용까지 작업했을 경우_

---
### Hadoop 환경변수 설정
- /etc/profile에 자바 환경변수 설정을 해놨지만, 따로 설정을 해야함.

~/.bashrc
```shell
...
# Java Environment Variable Setting
export JAVA_HOME=/usr/lib/jvm/jre-1.8.0/

# Hadoop Setting
export HADOOP_HOME=$DEV_HOME/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
```

```console
# bash 파일 변경정보 적용
[yeon@yeon-host dev]$ source ~/.bashrc
```

---
### core-site.xml 프로퍼티 설정
```console
[yeon@yeon-host dev]$ vi $HADOOP_HOME/etc/hadoop/core-site.xml
```
core-site.xml
```shell
...
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://yeon-host:9000</value> # yeon-host 부분은 서버의 호스트명을 넣기
        </property>
        <property>
                <name>hadoop.proxyuser.hadoop.hosts</name>
                <value>*</value>
        </property>
        <property>
                <name>hadoop.proxyuser.hadoop.groups</name>
                <value>*</value>
        </property>
</configuration>
```

---
### hdfs-site.xml 프로퍼티 설정
```console
[yeon@yeon-host dev]$ vi $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```
hdfs-site.xml
```shell
...
<configuration>
 <property>
         <name>dfs.replication</name>
         <value>1</value>
 </property>
 <property>
        <name>dfs.name.dir</name>
         <value>file:///hadoopdata/hdfs/namenode</value>
 </property>
 <property>
         <name>dfs.data.dir</name>
         <value>file:///hadoopdata/hdfs/datanode</value>
 </property>
 <property>
        <name>dfs.client.datanode-restart.timeout</name>
         <value>30</value>
 </property>
</configuration>
```

---
### mapred-site.xml 프로퍼티 설정
```console
[yeon@yeon-host dev]$ vi $HADOOP_HOME/etc/hadoop/mapred-site.xml
```
mapred-site.xml
```shell
...
<configuration>
 <property>
         <name>mapreduce.framework.name</name>
         <value>yarn</value>
 </property>
 <property>
         <name>yarn.app.mapreduce.am.env</name>
         <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
 </property>
 <property>
         <name>mapreduce.map.env</name>
         <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
 </property>
 <property>
         <name>mapreduce.reduce.env</name>
         <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
 </property>
 <property>
         <name>mapreduce.job.user.classpath.first</name>
         <value>true</value>
 </property>
</configuration>
```

---
### yarn-site.xml 프로퍼티 설정
```console
[yeon@yeon-host dev]$ vi $HADOOP_HOME/etc/hadoop/yarn-site.xml
```
yarn-site.xml
```shell
...
<configuration>
 <property>
         <name>yarn.nodemanager.aux-services</name>
         <value>mapreduce_shuffle</value>
 </property>
 <property>
         <name>yarn.nodemanager.vmem-check-enabled</name>
         <value>false</value>
 </property>
</configuration>
```

---
### ssh 공개키 생성
```console
[yeon@yeon-host dev]$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/yeon/.ssh/id_rsa):
Created directory '/home/yeon/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/yeon/.ssh/id_rsa.
Your public key has been saved in /home/yeon/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:/28Cr8KQOBre+agjAZ6qTYTrfA+i/Qz0i4Oj9iNgs28 yeon@yeon-host
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|                 |
|                 |
|..               |
|+ +  . .S        |
|oO..o o  ..      |
|====.o o  .o     |
|OB*E+o  o  .o .  |
|B=@BBo.  ....+.  |
+----[SHA256]-----+

[yeon@yeon-host .ssh]$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
[yeon@yeon-host .ssh]$ chmod 640 ~/.ssh/authorized_keys
```

---
### HadoopData 생성
- root 계정으로 hadoop 데이터 저장 디렉토리를 생성하고, yeon 계정에 모든 권한 부여


```console
[root@yeon-host ~]# mkdir -p /hadoopdata/hdfs/namenode
[root@yeon-host ~]# mkdir -p /hadoopdata/hdfs/datanode
# /hadoopdata 하위에 모든 권한 부여
[root@yeon-host ~]# chown -R yeon:yeon /hadoopdata
```

---
### Hadoop Namenode 초기화
```console
[yeon@yeon-host ~]$ hdfs namenode -format
```

---
### Hadoop 실행
```console
[yeon@yeon-host ~]$ start-all.sh
WARNING: Attempting to start all Apache Hadoop daemons as yeon in 10 seconds.
WARNING: This is not a recommended production deployment configuration.
WARNING: Use CTRL-C to abort.
Starting namenodes on [yeon-host]
Starting datanodes
Starting secondary namenodes [yeon-host]
Starting resourcemanager
Starting nodemanagers

# jps 명령어로 아래와 같이 Node/Manager 프로세스가 띄워져있는지 확인
[yeon@yeon-host ~]$ jps
7672 DataNode
7561 NameNode
8206 NodeManager
8542 Jps
7871 SecondaryNameNode
8095 ResourceManager
```

---
### Hadoop Namenode 모니터링 페이지
http://192.168.25.2:9870/
![Desktop View](/assets/img/contents/BigData/Hadoop/hadoop-setting/hadoop-namenode1.png)

- 참고로 하둡 실행 후, Summary 하위 safemode가 on으로 설정된다. 정상적으로 다 켜질 때까지 기다린 다음 safemode가 off로 전환 된 후에 hive를 구동해야 한다.

---
### Hadoop Yarn Resource Manager 모니터링 페이지
http://192.168.25.2:8088/
![Desktop View](/assets/img/contents/BigData/Hadoop/hadoop-setting/yarn-clustering1.png)

---
## 참고

### 하둡 실행시 Permission denied 오류 발생
- start-all.sh 로 hdfs 구동 시 아래와 같은 에러가 발생함.


```console
[yeon@yeon-host ~]$ start-all.sh
WARNING: Attempting to start all Apache Hadoop daemons as yeon in 10 seconds.
WARNING: This is not a recommended production deployment configuration.
WARNING: Use CTRL-C to abort.
Starting namenodes on [yeon-host]
yeon-host: Warning: Permanently added 'yeon-host,192.168.25.2' (ECDSA) to the list of known hosts.
yeon-host: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
Starting datanodes
localhost: Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
localhost: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
Starting secondary namenodes [yeon-host]
yeon-host: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
Starting resourcemanager
Starting nodemanagers
localhost: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
```
- 해결방법

ssh 공개키 설정 작업을 잘 했는지 확인 필요.

---
### 하둡 실행시 User를 못찾는 오류 발생
- start-all.sh 로 hdfs 구동 시 아래와 같은 에러가 발생함.


```console
ERROR: Attempting to operate on yarn nodemanager as root
ERROR: but there is no YARN_NODEMANAGER_USER defined. Aborting operation.
```
- 해결방법

$HADOOP_HOME/etc/hadoop/hadoop-env.sh
```shell
...
export HDFS_NAMENODE_USER="yeon"
export HDFS_DATANODE_USER="yeon"
export HDFS_SECONDARYNAMENODE_USER="yeon"
export YARN_RESOURCEMANAGER_USER="yeon"
export YARN_NODEMANAGER_USER="yeon"
```
---
