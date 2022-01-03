---
title: "[Hive-1] Hive 설치"
author:
  name: Yeoni
  link: https://github.com/cotes2020
date: 2022-01-03 22:00:00 +0900
categories: [BigData, Hive]
tags: [Hadoop, Hive]
math: true
mermaid: true
---


# Hive 설치

---
## 환경 구성

| Tools                        | Version          |
|:-----------------------------|-----------------:|
| VirtualBox                   | 6.1              |
| CentOS                       | 7                |
| Java Version                 | 1.8              |
| Hadoop                       | 3.3.1            |
| Hive                         | 3.1.2            |

---

---
## 선행 작업

- VirtualBox 설치
- VirtualBox에 CentOS 설치
- CentOS에 java설치
- CentOS에 Hadoop 설치

---
## 설치

- hive를 설치

### 설치 위치
/home/yeon/dev

---
### Hive 3.1.2 다운로드 및 압축해제
```console
cd $LIB_HOME
wget https://downloads.apache.org/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz
tar -xvzf apache-hive-3.1.2-bin.tar.gz
```
---
### Hive 심볼릭 링크 적용
```console
cd $DEV_HOME
ln -s $LIB_HOME/apache-hive-3.1.2-bin hive
```
![Desktop View](/assets/img/contents/BigData/Hive/hive-setting/symbolic1.png)
_Hive 심볼릭 링크 적용까지 작업했을 경우_

### Hive 환경변수 설정

~/.bashrc
```shell
...
# Hive Setting
export HIVE_HOME=$DEV_HOME/hive
export PATH="$HOME/.local/bin:$HOME/bin:$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HIVE_HOME/bin"
```

```console
# bash 파일 변경정보 적용
[yeon@yeon-host dev]$ source ~/.bashrc
```

### warehouse 생성
```console
hdfs dfs -mkdir /tmp
hdfs dfs -mkdir -p /user/hive/warehouse
# hadoop 클러스터 하나에 여러 사용자가 공유할 수 있으므로 권한을 따로 부여한다.
hdfs dfs -chmod g+w /tmp
hdfs dfs -chmod -R g+w /user
# hdfs dfs -chmod -R 777 / # 이건 따로 안줌. (gram에는 주고 samsung에는 안함)
```
---

### warehouse 확인
```console
[yeon@yeon-host dev]$ hdfs dfs -ls -R /user/hive
drwxrwxr-x   - yeon supergroup          0 2022-01-03 23:34 /user/hive/warehouse
```
---
### guava lib 옮기기
- guava : Google의 Java Opensource library. util 관련 라이브러리이다.
- Hadoop에 있는 guava-27.jar를 Hive/lib 로 옮기기. (버전을 맞추지 않으면 꼬일 수 있으므로..)

```console
# Hadoop/lib의 guava-27.jar을 Hive/lib로 이동
[yeon@yeon-host dev]$ cp $DEV_HOME/hadoop/share/hadoop/common/lib/guava-27.0-jre.jar $DEV_HOME/hive/lib
# Hive/lib에 있던 guava-19.jar 삭제
[yeon@yeon-host dev]$ rm $DEV_HOME/hive/lib/guava-19*.jar
# guava-*.jar 확인
[yeon@yeon-host dev]$ ll $DEV_HOME/hive/lib/guava-*
-rw-r--r--. 1 yeon yeon 2747878  1월  3 23:48 /home/yeon/dev/hive/lib/guava-27.0-jre.jar
```
---
