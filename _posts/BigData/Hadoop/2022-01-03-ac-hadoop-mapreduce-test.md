---
title: "[Hadoop-3] Hadoop MapReduce 테스트"
author:
  name: Yeoni
  link: https://github.com/cotes2020
date: 2022-01-03 21:00:00 +0900
categories: [BigData, Hadoop]
tags: [Hadoop, MapReduce]
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
- CentOS에 hadoop 설치

---
## MapReduce
MapReduce는 MR이라고도 표현한다.

테스트로 WordCount 예제를 실행하여 결과를 확인할 것이다.

테스트 데이터 : 공백을 포함한 문자들.
결과 : 공백을 구분자로 단어별 개수를 출력.

테스트 절차
- /tmp/mr/in에 input1.txt, input.csv 두 파일을 업로드하고, mapreduce 실행 후 결과를 /tmp/mr/out 디렉토리에 넣는다.
- 결과파일을 출력해보고, localFileSystem에 저장한다.

### MapReduce 샘플 jar
하둡 폴더에 MapReduce 샘플 jar가 있으므로, 그걸 사용하여 테스트를 진행할 수 있다.

```console
[yeon@yeon-host mapreduce]$ pwd
/home/yeon/dev/hadoop/share/hadoop/mapreduce
[yeon@yeon-host mapreduce]$ ll | grep hadoop-mapreduce-examples-3.3.1.jar
-rw-r--r--. 1 yeon yeon  280989  6월 15  2021 hadoop-mapreduce-examples-3.3.1.jar

# 테스트 폴더로 이동
[yeon@yeon-host mapreduce]$ cp hadoop-mapreduce-examples-3.3.1.jar $DEV_HOME/test/test-mr
```
---
### 테스트 파일 생성
```console
[yeon@yeon-host test-mr]$ cat input1.txt
aaa bbb
ccc ddd
eee
[yeon@yeon-host test-mr]$ cat input2.csv
eee
fff
ggg
aaa
```
---
### hdfs에 입력데이터를 넣을 디렉토리 생성
- /tmp/mr/out 디렉토리(출력용)는 미리 생성해놓으면 mapreduce 실행 시 Exception 이 발생하므로 만들지 않는다.

```console
[yeon@yeon-host test-mr]$ hdfs dfs -mkdir -R /tmp/mr/in
```
---
### hdfs에 파일 업로드
```console
[yeon@yeon-host test-mr]$ hdfs dfs -put input1.txt /tmp/mr/in
[yeon@yeon-host test-mr]$ hdfs dfs -put input2.csv /tmp/mr/in
```
---
### mapreduce 실행

- hadoop jar {mapreduce.jar} {작업 클래스명} {hdfs 입력 데이터 디렉토리} {hdfs 출력 데이터 디렉토리}

```console
[yeon@yeon-host test-mr]$ hadoop jar hadoop-mapreduce-examples-3.3.1.jar wordcount /tmp/mr/in /tmp/mr/out
2022-01-03 21:33:48,664 INFO client.DefaultNoHARMFailoverProxyProvider: Connecting to ResourceManager at /0.0.0.0:8032
2022-01-03 21:33:49,210 INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/yeon/.staging/job_1641209571726_0001
2022-01-03 21:33:49,563 INFO input.FileInputFormat: Total input files to process : 2 # 넣은 파일 개수
2022-01-03 21:33:50,533 INFO mapreduce.JobSubmitter: number of splits:2
2022-01-03 21:33:51,243 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1641209571726_0001
2022-01-03 21:33:51,243 INFO mapreduce.JobSubmitter: Executing with tokens: []
2022-01-03 21:33:51,580 INFO conf.Configuration: resource-types.xml not found
2022-01-03 21:33:51,580 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2022-01-03 21:33:52,221 INFO impl.YarnClientImpl: Submitted application application_1641209571726_0001
2022-01-03 21:33:52,293 INFO mapreduce.Job: The url to track the job: http://yeon-host:8088/proxy/application_1641209571726_0001/
2022-01-03 21:33:52,293 INFO mapreduce.Job: Running job: job_1641209571726_0001
2022-01-03 21:34:02,539 INFO mapreduce.Job: Job job_1641209571726_0001 running in uber mode : false
2022-01-03 21:34:02,543 INFO mapreduce.Job:  map 0% reduce 0%
2022-01-03 21:34:11,771 INFO mapreduce.Job:  map 50% reduce 0%
2022-01-03 21:34:12,791 INFO mapreduce.Job:  map 100% reduce 0%
2022-01-03 21:34:18,879 INFO mapreduce.Job:  map 100% reduce 100% # mapreduce 진행현황
2022-01-03 21:34:20,937 INFO mapreduce.Job: Job job_1641209571726_0001 completed successfully # mapreduce 실행 결과
2022-01-03 21:34:21,054 INFO mapreduce.Job: Counters: 55
        File System Counters
                FILE: Number of bytes read=96
                FILE: Number of bytes written=820016
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=250
                HDFS: Number of bytes written=42
                HDFS: Number of read operations=11
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=2
                HDFS: Number of bytes read erasure-coded=0
        Job Counters
                Killed map tasks=1
                Launched map tasks=2
                Launched reduce tasks=1
                Data-local map tasks=2
                Total time spent by all maps in occupied slots (ms)=14727
                Total time spent by all reduces in occupied slots (ms)=4089
                Total time spent by all map tasks (ms)=14727
                Total time spent by all reduce tasks (ms)=4089
                Total vcore-milliseconds taken by all map tasks=14727
                Total vcore-milliseconds taken by all reduce tasks=4089
                Total megabyte-milliseconds taken by all map tasks=15080448
                Total megabyte-milliseconds taken by all reduce tasks=4187136
        Map-Reduce Framework
                Map input records=7
                Map output records=9
                Map output bytes=72
                Map output materialized bytes=102
                Input split bytes=214
                Combine input records=9
                Combine output records=9
                Reduce input groups=7
                Reduce shuffle bytes=102
                Reduce input records=9
                Reduce output records=7
                Spilled Records=18
                Shuffled Maps =2
                Failed Shuffles=0
                Merged Map outputs=2
                GC time elapsed (ms)=306
                CPU time spent (ms)=1300
                Physical memory (bytes) snapshot=544653312
                Virtual memory (bytes) snapshot=8230359040
                Total committed heap usage (bytes)=301146112
                Peak Map Physical memory (bytes)=216301568
                Peak Map Virtual memory (bytes)=2740568064
                Peak Reduce Physical memory (bytes)=113123328
                Peak Reduce Virtual memory (bytes)=2749423616
        Shuffle Errors
                BAD_ID=0
                CONNECTION=0
                IO_ERROR=0
                WRONG_LENGTH=0
                WRONG_MAP=0
                WRONG_REDUCE=0
        File Input Format Counters
                Bytes Read=36
        File Output Format Counters
                Bytes Written=42
```

---
### 실행 결과 확인

- Yarn Resource Manager에서 실행 정보 확인 가능.

![Desktop View](/assets/img/contents/BigData/Hadoop/hadoop-mapreduce-test/yarn1.png)
_Yarn Resource Manager에서 실행 항목 조회_

---
### 실행 결과 확인

- Mapreduce가 성공적으로 끝났을 경우, [출력디렉토리]에 _SUCCESS 파일이 만들어진다. (내용은 없음)
- part-r-xxxxx 파일에 출력결과가 넣어져있다.

```console
[yeon@yeon-host test-mr]$ hdfs dfs -ls /tmp/mr/out
Found 2 items
-rw-r--r--   1 yeon supergroup          0 2022-01-03 21:34 /tmp/mr/out/_SUCCESS
-rw-r--r--   1 yeon supergroup         42 2022-01-03 21:34 /tmp/mr/out/part-r-00000
[yeon@yeon-host test-mr]$ hdfs dfs -cat /tmp/mr/out/part-r-00000
aaa     2
bbb     1
ccc     1
ddd     1
eee     2
fff     1
ggg     1
```

---
### 실행 결과를 localFileSystem에 저장

```console
# localFileSystem에 출력결과를 저장할 디렉토리 생성(./out)
[yeon@yeon-host test-mr]$ mkdir out
# hdfs -> localFileSystem 파일 다운로드
[yeon@yeon-host test-mr]$ hdfs dfs -get /tmp/mr/out/* ./out
# localFileSystem 확인
[yeon@yeon-host test-mr]$ ll out
합계 4
-rw-r--r--. 1 yeon yeon  0  1월  3 21:58 _SUCCESS
-rw-r--r--. 1 yeon yeon 42  1월  3 21:58 part-r-00000
# localFileSystem 데이터 조회
[yeon@yeon-host test-mr]$ cat out/part-r-00000
aaa     2
bbb     1
ccc     1
ddd     1
eee     2
fff     1
ggg     1
```
---
