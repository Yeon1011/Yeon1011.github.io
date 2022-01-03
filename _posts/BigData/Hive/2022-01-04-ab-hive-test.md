---
title: "[Hive-2] Hive 실행 및 테스트 (derby)"
author:
  name: Yeoni
  link: https://github.com/cotes2020
date: 2022-01-04 00:00:00 +0900
categories: [BigData, Hive]
tags: [Hadoop, Hive, HiveQL]
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
## 사용포트 정리

| Program                      | Port Number      |
|:-----------------------------|-----------------:|
| NameNode                     | 9870             |
| DataNode                     | 9864             |


---
## 선행 작업

- VirtualBox 설치
- VirtualBox에 CentOS 설치
- CentOS에 java설치
- CentOS에 Hadoop 설치
- CentOS에 Hive 설치

---
## 하이브쉘

하이브쉘은 HiveQL 명령어로 하이브와 상호작용할 수 있는 하이브의 기본 도구. (mysql과 매우 유사)

## 실행 및 테스트

- hive-site.xml 메타스토어를 등록하지 않고, derby로 테스트.
- hive 쉘을 사용하여  테스트

### Hive 스키마 생성

- hive-site.xml에 따로 메타스토어를 설정하지 않으면 dbType에 derby를 사용하여 hive를 구동할 수 있다.

```console
# derby 테스트용 디렉토리 생성
[yeon@yeon-host derby]$ pwd
/home/yeon/dev/hive/derby
# 스키마 초기화
[yeon@yeon-host derby]$ schematool -dbType derby -initSchema
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/yeon/lib/apache-hive-3.1.2-bin/lib/log4j-slf4j-impl-2.10.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/yeon/lib/hadoop-3.3.1/share/hadoop/common/lib/slf4j-log4j12-1.7.30.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Metastore connection URL:        jdbc:derby:;databaseName=metastore_db;create=true
Metastore Connection Driver :    org.apache.derby.jdbc.EmbeddedDriver
Metastore connection User:       APP
Starting metastore schema initialization to 3.1.0
Initialization script hive-schema-3.1.0.derby.sql


Initialization script completed
schemaTool completed
```
---

### Hive 실행

- 실행
derby를 사용할 경우, 스키마를 초기화한 디렉토리에서 hive를 구동해야 한다.

```console
[yeon@yeon-host derby]$ hive
```

- 실행 (로그 출력 되도록 옵션 설정)

```console
[yeon@yeon-host derby]$ hive --hiveconf hive.root.logger=INFO,console
```

- 실행 (Background에서 hiveserver2로 구동)

```console
[yeon@yeon-host derby]$ hive --service hiveserver2 &
```


### 하이브 쉘 명령어 테스트

```console
# 데이터베이스 조회
hive> show databases;
OK
default
Time taken: 0.035 seconds, Fetched: 1 row(s)

# 테이블 조회
hive> show tables;
OK
Time taken: 0.044 seconds

# 테이블 생성
create table person ( input_date string, name string, age int ) row format delimited fields terminated by ',' lines terminated by '\n' stored as textfile;

# 테이블에 데이터 Insert
# 대략 1 row당 30초 정도 소요. (기본 mapreduce 사용)
insert into person values ('20200101', 'KIM', 33);
insert into person values ('20200102', 'BAE', 22);
insert into person values ('20200103', 'KIM', 35);
insert into person values ('20200104', 'LEE', 16);
insert into person values ('20200105', 'SO', 25);

...
2022-01-04T00:09:21,559  INFO [ccf67a99-62fa-434b-98ab-388e73b9216e main] ql.Driver: OK
2022-01-04T00:09:21,559  INFO [ccf67a99-62fa-434b-98ab-388e73b9216e main] ql.Driver: Concurrency mode is disabled, not creating a lock manager
Time taken: 32.305 seconds
2022-01-04T00:09:21,584  INFO [ccf67a99-62fa-434b-98ab-388e73b9216e main] CliDriver: Time taken: 32.305 seconds # 소요시간

# 조회 SQL
# 28초 정도 소요 (기본 mapreduce 사용)
select name, count(*), max(age), min(age) from person group by name;

...
2022-01-04T00:11:11,731  INFO [ccf67a99-62fa-434b-98ab-388e73b9216e main] mapred.FileInputFormat: Total input files to process : 1
BAE     1       22      22
KIM     2       35      33
LEE     1       16      16
SO      1       25      25
2022-01-04T00:11:11,754  INFO [ccf67a99-62fa-434b-98ab-388e73b9216e main] exec.ListSinkOperator: RECORDS_OUT_OPERATOR_LIST_SINK_10:4, RECORDS_OUT_INTERMEDIATE:0,
Time taken: 28.575 seconds, Fetched: 4 row(s)
2022-01-04T00:11:11,757  INFO [ccf67a99-62fa-434b-98ab-388e73b9216e main] CliDriver: Time taken: 28.575 seconds, Fetched: 4 row(s)
```
### Yarn Resource Manager 실행 결과 확인

- Yarn Resource Manager에서 실행 정보 확인 가능.
- insert / select 실행 결과 모두 조회 가능.

![Desktop View](/assets/img/contents/BigData/Hive/hive-test/yarn1.png)
_Yarn Resource Manager에서 실행 항목 조회_

### derby 사용 + hiveserver2 구동

아래와 같이 hive로 process는 띄워져있으나 hive port (default:10000)은 띄워져 있지 않음..

mysql이나 mssql로 metastore를 등록해야 하는듯.

```console
[yeon@yeon-host ~]$ ps  -ef | grep hive
yeon     12468  1529 65 00:22 pts/0    00:00:13 /usr/lib/jvm/jre-1.8.0//bin/java -Dproc_jar -Djava.library.path=/home/yeon/dev/hadoop/lib/native -Dproc_hiveserver2 -Dlog4j.configurationFile=hive-log4j2.properties -Djava.util.logging.config.file=/home/yeon/dev/hive/conf/parquet-logging.properties -Djline.terminal=jline.UnsupportedTerminal -Dyarn.log.dir=/home/yeon/dev/hadoop/logs -Dyarn.log.file=hadoop.log -Dyarn.home.dir=/home/yeon/dev/hadoop -Dyarn.root.logger=INFO,console -Xmx256m -Dhadoop.log.dir=/home/yeon/dev/hadoop/logs -Dhadoop.log.file=hadoop.log -Dhadoop.home.dir=/home/yeon/dev/hadoop -Dhadoop.id.str=yeon -Dhadoop.root.logger=INFO,console -Dhadoop.policy.file=hadoop-policy.xml -Dhadoop.security.logger=INFO,NullAppender org.apache.hadoop.util.RunJar /home/yeon/dev/hive/lib/hive-service-3.1.2.jar org.apache.hive.service.server.HiveServer2
yeon     12610  1553  0 00:22 pts/1    00:00:00 grep --color=auto hive
[yeon@yeon-host ~]$ netstat -ntlp
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
tcp6       0      0 ::1:25                  :::*                    LISTEN      -
tcp6       0      0 :::13562                :::*                    LISTEN      2407/java
tcp6       0      0 :::46106                :::*                    LISTEN      2407/java
tcp6       0      0 :::8030                 :::*                    LISTEN      2297/java
tcp6       0      0 :::8031                 :::*                    LISTEN      2297/java
tcp6       0      0 :::8032                 :::*                    LISTEN      2297/java
tcp6       0      0 :::8033                 :::*                    LISTEN      2297/java
tcp6       0      0 :::8040                 :::*                    LISTEN      2407/java
tcp6       0      0 :::9864                 :::*                    LISTEN      1861/java
tcp6       0      0 192.168.25.2:9000       :::*                    LISTEN      1748/java
tcp6       0      0 :::8042                 :::*                    LISTEN      2407/java
tcp6       0      0 :::9866                 :::*                    LISTEN      1861/java
tcp6       0      0 :::9867                 :::*                    LISTEN      1861/java
tcp6       0      0 :::9868                 :::*                    LISTEN      2069/java
tcp6       0      0 127.0.0.1:41260         :::*                    LISTEN      1861/java
tcp6       0      0 :::9870                 :::*                    LISTEN      1748/java
tcp6       0      0 :::22                   :::*                    LISTEN      -
tcp6       0      0 :::8088                 :::*                    LISTEN      2297/java

```
---
