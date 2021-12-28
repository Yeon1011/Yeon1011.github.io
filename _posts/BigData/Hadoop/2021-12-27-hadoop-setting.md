---
title: Hadoop 설치
author:
  name: Yeoni
  link: https://github.com/cotes2020
date: 2021-12-27 17:00:00 +0900
categories: [BigData, Hadoop]
tags: [typography]
math: true
mermaid: true
#image:
#  src: /assets/img/contents/BigData/Hadoop/hadoop-setting/test.png
#  width: 800
#  height: 500
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
## 선행 작업

- VirtualBox 설치
- VirtualBox에 CentOS 설치
- CentOS에 java설치

---
## 설치

### 설치 위치
/root/dev

### Hadoop 3.3.1 다운로드 및 압축해제
```console
cd /root/dev
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.1/hadoop-3.3.1.tar.gz
tar -xvzf hadoop-3.3.1.tar.gz
```
### Hadoop 심볼릭 링크 적용
```console
ln -s hadoop-3.3.1 hadoop
```

![Desktop View](/assets/img/contents/BigData/Hadoop/hadoop-setting/image1.png)
_Hadoop 심볼릭 링크 적용까지 작업했을 경우_

<br>


### Hadoop 환경변수 설정
#### ~/.bashrc
```shell
# Java Setting
export JAVA_HOME=/usr/lib/jvm/jre-1.8.0/

# Dev Home
export DEV_HOME=/root/dev

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

### User 정보 추가
#### $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```shell
export HDFS_NAMENODE_USER="root"
export HDFS_DATANODE_USER="root"
export HDFS_SECONDARYNAMENODE_USER="root"
export YARN_RESOURCEMANAGER_USER="root"
export YARN_NODEMANAGER_USER="root"
```
