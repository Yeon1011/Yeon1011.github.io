---
title: "[Hive-3] Hive 기능 테스트 (derby)"
author:
  name: Yeoni
  link: https://github.com/cotes2020
date: 2022-01-04 23:00:00 +0900
categories: [BigData, Hive]
tags: [Hadoop, Hive, HiveQL]
math: true
mermaid: true
---


# hdfs에서 파일 변경 후 hive로 확인

-  /user/hive/warehouse/person/000000_0_copy_2 파일명 변경 후 정상동작 확인

- hdfs 작업

```console
# [1] 변경전 체크
[yeon@yeon-host derby]$ hdfs dfs -ls /user/hive/warehouse/person/
Found 5 items
-rw-r--r--   1 yeon supergroup         16 2022-01-04 00:07 /user/hive/warehouse/person/000000_0
-rw-r--r--   1 yeon supergroup         16 2022-01-04 00:07 /user/hive/warehouse/person/000000_0_copy_1
-rw-r--r--   1 yeon supergroup         16 2022-01-04 00:08 /user/hive/warehouse/person/000000_0_copy_2 # 변경할 파일
-rw-r--r--   1 yeon supergroup         16 2022-01-04 00:08 /user/hive/warehouse/person/000000_0_copy_3
-rw-r--r--   1 yeon supergroup         15 2022-01-04 00:09 /user/hive/warehouse/person/000000_0_copy_4
# [2] --- copy_2 파일을 --- copy_9로 변경
[yeon@yeon-host derby]$ hdfs dfs -mv /user/hive/warehouse/person/000000_0_copy_2 /user/hive/warehouse/person/000000_0_copy_9
# [3] 변경 후 확인
[yeon@yeon-host derby]$ hdfs dfs -ls /user/hive/warehouse/person/
Found 5 items
-rw-r--r--   1 yeon supergroup         16 2022-01-04 00:07 /user/hive/warehouse/person/000000_0
-rw-r--r--   1 yeon supergroup         16 2022-01-04 00:07 /user/hive/warehouse/person/000000_0_copy_1
-rw-r--r--   1 yeon supergroup         16 2022-01-04 00:08 /user/hive/warehouse/person/000000_0_copy_3
-rw-r--r--   1 yeon supergroup         15 2022-01-04 00:09 /user/hive/warehouse/person/000000_0_copy_4
-rw-r--r--   1 yeon supergroup         16 2022-01-04 00:08 /user/hive/warehouse/person/000000_0_copy_9 # 변경된 파일
```

- hive 확인

```console
# 변경 전
hive> select * from person where input_date = '20200103';
OK
20200103        KIM     35
Time taken: 3.703 seconds, Fetched: 1 row(s)
# 변경 전
hive> select * from person where input_date = '20200103';
OK
20200103        KIM     35
Time taken: 0.195 seconds, Fetched: 1 row(s)
```

- 결과 : 정상적으로 조회됨

## 파일을 삭제해보기

- hdfs 작업

```console
[yeon@yeon-host hive-test]$ hdfs dfs -rm /user/hive/warehouse/person/000000_0_copy_4
Deleted /user/hive/warehouse/person/000000_0_copy_4
[yeon@yeon-host hive-test]$ hdfs dfs -ls /user/hive/warehouse/person
Found 4 items
-rw-r--r--   1 yeon supergroup         16 2022-01-04 00:07 /user/hive/warehouse/person/000000_0
-rw-r--r--   1 yeon supergroup         16 2022-01-04 00:07 /user/hive/warehouse/person/000000_0_copy_1
-rw-r--r--   1 yeon supergroup         16 2022-01-04 00:08 /user/hive/warehouse/person/000000_0_copy_3
-rw-r--r--   1 yeon supergroup         16 2022-01-04 00:08 /user/hive/warehouse/person/000000_0_copy_9
[yeon@yeon-host hive-test]$
[yeon@yeon-host hive-test]$ hdfs dfs -cat /user/hive/warehouse/person/*
20200101,KIM,33
20200102,BAE,22
20200104,LEE,16
20200103,KIM,35
```

- hive 확인

```console
hive> select count(1) from person;
OK
5
Time taken: 3.797 seconds, Fetched: 1 row(s)
hive> select * from person;
OK
20200101        KIM     33
20200102        BAE     22
20200104        LEE     16
20200103        KIM     35
Time taken: 0.305 seconds, Fetched: 4 row(s)
```

- 결과 : 파일을 삭제한 후 조회를 하면 4개 행만 조회되지만, count(1)로 할 경우 5로 조회된다.
- 실데이터는 hdfs의 파일을 조회하지만 count정보는 다른 곳에서 들고오는 듯...?

---
## 파일을 신규로 생성해보기

- hdfs 작업

```console
# localPath에 파일 생성
[yeon@yeon-host hive-test]$ cat normal
20200121,YEON,28
# hdfs에 파일 put
[yeon@yeon-host hive-test]$ hdfs dfs-put normal /user/hive/warehouse/person
```

- hive 확인

```console
hive> select count(1) from person;
OK
5
hive> select * from person;
OK
20200101        KIM     33
20200102        BAE     22
20200104        LEE     16
20200120        YEON    28
20200103        KIM     35
```

- 결과 : 삭제 테스트와 동일하게 select * 로 조회했을 때는 조회되나 count는 반영되지 않음.

---
## 형식에 맞지않은 파일을 추가해보기

- hdfs 작업

```console
# 컬럼 3개중 하나만 값을 넣기
[yeon@yeon-host hive-test]$ cat notnormal
20200122
[yeon@yeon-host hive-test]$ hdfs dfs -put notnormal /user/hive/warehouse/person

# 컬럼 3개를 초과하여 입력하기
[yeon@yeon-host hive-test]$ cat manydata
20200123,YEON,28,1
[yeon@yeon-host hive-test]$ hdfs dfs -put manydata /user/hive/warehouse/person
```

- hive 확인

```console
hive> select * from person;
OK
20200101        KIM     33
20200102        BAE     22
20200104        LEE     16
20200120        YEON    28
20200103        KIM     35
20200123        YEON    28   # manydata
20200121        YEON    28
20200122        NULL    NULL # notnormal
```

- 결과 : 컬럼개수가 넘치는건 뒤의값을 자르고, 모자른건 null로 조회.

---
## 기존파일에 레코드를 추가해보기

- hdfs 작업

```console
# hdfs 파일 조회
[yeon@yeon-host hive-test]$ hdfs dfs -cat /user/hive/warehouse/person/000000_0_copy_3
20200104,LEE,16

# local File 생성 (append 용도)
[yeon@yeon-host hive-test]$ cat appendTest
20220104,KANG,29

# local File을 hdfs 파일에 append
[yeon@yeon-host hive-test]$ hdfs dfs -appendToFile appendTest /user/hive/warehouse/person/000000_0_copy_3

# hdfs 확인
[yeon@yeon-host hive-test]$ hdfs dfs -cat /user/hive/warehouse/person/000000_0_copy_3
20200104,LEE,16
20220104,KANG,29
```

- hive 확인

```console
hive> select * from person;
OK
20200101        KIM     33
20200102        BAE     22
20200104        LEE     16
20220104        KANG    29 # append 한 데이터
20200120        YEON    28
20200103        KIM     35
20200123        YEON    28
20200121        YEON    28
20200122        NULL    NULL
Time taken: 0.181 seconds, Fetched: 9 row(s)
```

- 결과 : 정상적으로 데이터가 조회됨.

---
## Hive에 record insert후 확인

- hive 작업 및 확인

```console
# 신규 데이터 insert
hive> insert into person values ('20220104','NAM','27');
...

# 데이터 확인
hive> select * from person;
OK
20200101        KIM     33
20200102        BAE     22
20220104        NAM     27 # 추가한 데이터
20200104        LEE     16
20220104        KANG    29
20200120        YEON    28
20200103        KIM     35
20200123        YEON    28
20200121        YEON    28
20200122        NULL    NULL
Time taken: 0.18 seconds, Fetched: 10 row(s)

# 데이터 건수 확인
hive> select count(1) from person;
OK
6
Time taken: 0.23 seconds, Fetched: 1 row(s)
```

- 결과 : total count가 1이 증가한 걸 확인..

---

## Hive 테이블에 hdfs 경로를 지정하여 load

- `load data inpath '{hadoopFilePath}' into table {table};`

```console
# 기존 데이터 확인
hive> select * from person;
OK
20200101        KIM     33
20200102        BAE     22
20220104        NAM     27
20200104        LEE     16
20220104        KANG    29
20200120        YEON    28
20200103        KIM     35
20200123        YEON    28
20200121        YEON    28
20200122        NULL    NULL
Time taken: 0.18 seconds, Fetched: 10 row(s)

# count 조회 결과 확인
hive> select count(1) from person;
OK
6
Time taken: 0.23 seconds, Fetched: 1 row(s)

-> hdfs 상으로는 10개의 row가 들어있으나 count(1) 결과는 6으로 나옴. (hdfs에서 수동으로 넣은 데이터는 조회만 되고 카운팅 되지 않음)

---

# [20200121,YEON,28] hdfs로 들어가 있는 데이터를 person 테이블에 load
hive> load data inpath '/user/hive/warehouse/person/normal' into table person; /
Loading data to table default.person
OK
Time taken: 0.288 seconds

# count 조회 결과 확인
hive> select count(1) from person;
Query ID = yeon_20220105000220_3ff6dd33-6b87-40d8-bf01-b820414797fa
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Starting Job = job_1641300868505_0002, Tracking URL = http://yeon-host:8088/proxy/application_1641300868505_0002/
Kill Command = /home/yeon/dev/hadoop/bin/mapred job  -kill job_1641300868505_0002
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
2022-01-05 00:02:31,463 Stage-1 map = 0%,  reduce = 0%
2022-01-05 00:02:38,896 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 1.45 sec
2022-01-05 00:02:48,324 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 3.06 sec
MapReduce Total cumulative CPU time: 3 seconds 60 msec
Ended Job = job_1641300868505_0002
MapReduce Jobs Launched:
Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 3.06 sec   HDFS Read: 13066 HDFS Write: 102 SUCCESS
Total MapReduce CPU Time Spent: 3 seconds 60 msec
OK
10
Time taken: 28.674 seconds, Fetched: 1 row(s)
```

---

## hdfs에 파일을 넣은 후, location 지정하여 hive 테이블 생성

- `CREATE TABLE {table} ({컬럼 정보}) location '{hadoopFilePath}';`


- hdfs 작업

```console
# /tmp/hive-test 디렉토리를 생성하여 데이터 insert
[yeon@yeon-host hive-test]$ hdfs dfs -cat /tmp/hive-test/*
20220104,KANG,29
20200121,YEON,28
```

- hive 테이블 생성 후 조회

- location 은 'hdfs://hostname - 부터 적어도 됨'

```console
# 테이블 생성
hive> CREATE TABLE employee (input_date String, name String, age int) location '/tmp/hive-test/';
OK
Time taken: 0.295 seconds

# 조회
hive> select * from employee;
OK
20220104,KANG,29        NULL    NULL
20200121,YEON,28        NULL    NULL
Time taken: 0.236 seconds, Fetched: 2 row(s)

```

- hdfs 확인

```console
# 따로 employee 디렉토리가 생성되지 않음.
[yeon@yeon-host hive-test]$ hdfs dfs -ls /user/hive/warehouse/
Found 1 items
drwxr-xr-x   - yeon supergroup          0 2022-01-05 00:02 /user/hive/warehouse/person
```

## hive에서 record insert

- hive 실행 및 확인

```console
# hive insert
hive> insert into employee values ('20220105', 'MUN', 33);
Query ID = yeon_20220105001741_bb6928ce-d514-49a6-9d42-40b38f123bed
Total jobs = 3
Launching Job 1 out of 3
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Starting Job = job_1641300868505_0004, Tracking URL = http://yeon-host:8088/proxy/application_1641300868505_0004/
Kill Command = /home/yeon/dev/hadoop/bin/mapred job  -kill job_1641300868505_0004
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
2022-01-05 00:17:51,730 Stage-1 map = 0%,  reduce = 0%
2022-01-05 00:18:00,139 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 1.82 sec
2022-01-05 00:18:07,441 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 3.32 sec
MapReduce Total cumulative CPU time: 3 seconds 320 msec
Ended Job = job_1641300868505_0004
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Moving data to directory hdfs://yeon-host:9000/tmp/hive-test/.hive-staging_hive_2022-01-05_00-17-41_521_7792296284703375118-1/-ext-10000
Loading data to table default.employee
MapReduce Jobs Launched:
Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 3.32 sec   HDFS Read: 16143 HDFS Write: 287 SUCCESS
Total MapReduce CPU Time Spent: 3 seconds 320 msec
OK
Time taken: 28.357 seconds

# 결과 확인
hive> select * from employee;
OK
20220105        MUN     33
20200120,YEON,28        NULL    NULL
20220104,KANG,29        NULL    NULL
20200121,YEON,28        NULL    NULL
Time taken: 0.196 seconds, Fetched: 4 row(s)
```

- hdfs 확인

```console
[yeon@yeon-host hive-test]$ hdfs dfs -cat /tmp/hive-test/*
20220105MUN33 # hive에서 insert한 데이터
20200120,YEON,28
20220104,KANG,29
20200121,YEON,28
```

- hive에서 insert한 경우에는 정상적으로 들어가지고, hdfs에서는 구분자없이 일렬로 들어간것으로 확인.

---



## localPath의 파일을 hive에 load

- hdfs 작업

```console
# 위와 비슷하게 구분자없이 일렬로 나열된 데이터 파일 생성.
[yeon@yeon-host hive-test]$ cat emptest
20220105SUN55
```

- hive 실행 및 확인

```console
# hive에 로컬 파일 load
hive> load data local inpath '/home/yeon/dev/test/hive-test/emptest' into table employee;
Loading data to table default.employee
OK
Time taken: 0.23 seconds

# hive 데이터 조회
hive> select * from employee;
OK
20220105        MUN     33
20200120,YEON,28        NULL    NULL
20220104,KANG,29        NULL    NULL
20220105SUN55   NULL    NULL  # localfile load 데이터
20200121,YEON,28        NULL    NULL
Time taken: 0.142 seconds, Fetched: 5 row(s)

```

- 결과 : 정상적으로 load는 되었지만 구분되지 않고 input_date에 다 들어감.
- `따로 옵션을 지정해야 할듯.`


## hive partition table load

- `ALTER TABLE employee ADD PARTITION (inputDate='20170101') LOCATION '{hadoopFilePath}';`
- 테스트는 아직 안해봄.

---
