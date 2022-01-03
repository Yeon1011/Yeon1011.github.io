---
title: CentOS7 작업 내용
author:
  name: Yeoni
  link: https://github.com/cotes2020
date: 2022-01-01 13:00:00 +0900
categories: [Dev, Infomation]
tags: [CentOS7]
math: true
mermaid: true
#image:
#  src: /assets/img/contents/BigData/Hadoop/hadoop-setting/test.png
#  width: 800
#  height: 500
---


# CentOS7 설치

## 네트워크 도구 설치
- 해당 툴을 다운받아야 `ifconfig` 명령어 사용 가능.


```shell
[yeon@yeon-host dev]$ yum install net-tools
```
---
## Java 설치
- `java-1.8.0-openjdk-devel`를 설치하면 `jps` 명령어 사용 가능


```shell
[yeon@yeon-host dev]$ yum install java-1.8.0-openjdk
[yeon@yeon-host dev]$ yum install java-1.8.0-openjdk-devel
# Java version 확인
[yeon@yeon-host dev]$ java -version
```

![Desktop View](/assets/img/contents/Dev/Infomation/centos-setting/java-version1.png)
_java version 확인_

```shell
[yeon@yeon-host dev]$ readlink -f /usr/bin/java
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64/jre/bin/java
# root 계정으로 /etc/profiles에 환경변수 관련 설정 추가
## /etc/profile : 전역 프로파일로 부팅 시점에서 기본적으로 적용되는 환경 설정 파일
[root@yeon-host ~]# vi /etc/profile
---
...
## Java Environment Variable Setting
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64
PATH=$PATH:$JAVA_HOME/bin
CLASSPATH=$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar

export JAVA_HOME PATH CLASSPATH
---
# /etc/profile 변경내용 적용
[root@yeon-host ~]# source /etc/profile
```

![Desktop View](/assets/img/contents/Dev/Infomation/centos-setting/java-environment1.png)
_java 환경변수 적용 확인_


### jps 명령어
- JVM(자바 가상 머신)위에 띄워진 프로세스를 모니터링 할 수 있음.

![Desktop View](/assets/img/contents/Dev/Infomation/centos-setting/jps1.png)
_jps 명령어 입력시 출력되는 내용_


---

## 방화벽 설정

### 방화벽 설정 (전체)
```shell
# 방화벽 켜기
[root@yeon-host ~]# systemctl start firewalld

# 방화벽 끄기
[root@yeon-host ~]# systemctl stop firewalld
```

### 방화벽 설정 (특정포트)
```shell
# 특정 포트 방화벽 열기
[root@yeon-host ~]# firewall-cmd --permanent --zone=public --add-port=8088/tcp
success

# 특정포트 방화벽 닫기
[root@yeon-host ~]# firewall-cmd --permanent --zone=public --remove-port=8088/tcp
success

# 방화벽 설정 적용
[root@yeon-host ~]# firewall-cmd --reload
success

# 방화벽 포트 열렸는지 확인
[yeon@yeon-host ~]$ netstat -ntlp
# Local Address가 127.0.0.1로 되있으면 Local에서만 접근가능하다는 의미
# Local Address가 0.0.0.0 또는 :: 로 되있으면 어디서나 접근가능하다는 의미
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      -
tcp6       0      0 127.0.0.1:35197         :::*                    LISTEN      7672/java
tcp6       0      0 :::8030                 :::*                    LISTEN      8095/java
tcp6       0      0 :::8031                 :::*                    LISTEN      8095/java
tcp6       0      0 192.168.25.2:9000       :::*                    LISTEN      7561/java
tcp6       0      0 :::41613                :::*                    LISTEN      8206/java
tcp6       0      0 :::9870                 :::*                    LISTEN      7561/java
tcp6       0      0 :::22                   :::*                    LISTEN      -
tcp6       0      0 :::8088                 :::*                    LISTEN      8095/java
tcp6       0      0 ::1:25                  :::*                    LISTEN      -
tcp6       0      0 :::13562                :::*                    LISTEN      8206/java
                    # 해당 부분 ip와 port로 열렸는지 확인가능. (예시. 8088 포트는 열림)
```
---
## 서버시각 동기화
```shell
# 현재 서버 시각 확인
[root@yeon-host ~]# date
2022. 01. 02. (일) 02:07:43 KST

# ntp 설치
[root@yeon-host ~]# yum install ntp

# rdate 설치
[root@yeon-host ~]# yum install rdate

# 원격 서버의 로컬 시각 정보 조회
[root@yeon-host ~]# rdate -p time.bora.net
rdate: [time.bora.net]  Sun Jan  2 02:13:23 2022

# 원격 서버 정보를 로컬 시각으로 적용
[root@yeon-host ~]# rdate -s time.bora.net
```

### ntp
- Network Time Protocol. 네트워크 타임 프로토콜
- 가변 레이턴시 데이터 네트워크르르 통해 컴퓨터 시스템 간 시간 동기화를 위한 네트워크 프로토콜.

### rdate
- Remote Date의 줄임말으로,  로컬 시스템의 시각을 원격 서버의 시각 정보로 맞춰주는 명령어.

---
## Bash 파일을 잘못 올렸을 때
- 잘못입력하여 적용시킬 경우 아무런 명령어도 먹지 않아, 아래처럼 그 경로에 직접 기본값을 넣어 초기화를 해줌.

```shell
[root@yeon-host ~]# PATH=/usr/local/bin:/bin:/usr/bin
```
---
## Bash 파일 변경시 source 명령어를 사용하여 적용
- ~/.bashrc파일은 로그인할 때 읽는 파일
- source 명령어를 사용하지 않으면 적용되지 않으며, 재로그인을 할 경우에는 적용이 된다.

```shell
[root@yeon-host ~]# vi ~/.bashrc
... 작업 ...
# source 명령어를 사용하여 변경사항을 적용
[root@yeon-host ~]# source ~/.bashrc
```
---
## Host 추가
- root 계정 사용

/etc/hosts
```shell
...
192.168.25.2    yeon-host
```
---
