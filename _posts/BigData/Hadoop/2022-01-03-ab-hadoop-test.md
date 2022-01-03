---
title: "[Hadoop-2] Hadoop Command 테스트"
author:
  name: Yeoni
  link: https://github.com/cotes2020
date: 2022-01-03 20:00:00 +0900
categories: [BigData, Hadoop]
tags: [Hadoop, Test]
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
## Command 테스트
리눅스 명령어  `mkdir`, `ls`, `rm` 등 실제 명령어와 비슷하다.

`hdfs dfs`를 사용하여, hdfs와 상호작용할 수 있다.

`hadoop fs`와 `hdfs dfs` 명령어가 있는데, `hadoop fs`는 hdfs외에 local 파일 시스템도 상호작용 가능하다.

---
### mkdir - 디렉토리 생성

설치위치는 DEV_HOME으로 지정하여 ~/.bashrc에 변수로 환경변수 등록.

- hdfs dfs -mkdir [-p] {hdfsPath}

- -p : 하위 디렉토리까지 전체 생성.
```console
[yeon@yeon-host ~]$ hdfs dfs -mkdir -p /tmp/test/
```
---
### put - 파일 업로드

- hdfs dfs -put {localPath} {hdfsPath}

```console
# 파일 생성
[yeon@yeon-host test]$ cat input1.txt
aaaa
bbbb
# Hadoop에 파일 업로드
[yeon@yeon-host test]$ hdfs dfs -put input1.txt /tmp/test/
```
---
### ls - 디렉토리 정보 확인

- hdfs dfs -ls [-R] {hdfsPath}
- -R : 하위 디렉토리를 포함한 정보 확인

```console
[yeon@yeon-host test]$ hdfs dfs -ls -R /tmp/
drwxr-xr-x   - yeon supergroup          0 2022-01-03 20:43 /tmp/test
-rw-r--r--   1 yeon supergroup         10 2022-01-03 20:43 /tmp/test/input1.txt
```
---
### cat - 파일 내용 출력

- hdfs dfs -cat {hdfsPath}

```console
[yeon@yeon-host test]$ hdfs dfs -cat /tmp/test/input1.txt
aaaa
bbbb
```
---
### appendToFile  - 파일 내용 추가

- hdfs dfs -appendToFile {localPath} {hdfsPath}

```console
# 추가할 파일 조회
[yeon@yeon-host test]$ cat input3.txt
input3 append
# 파일 append
[yeon@yeon-host test]$ hdfs dfs -appendToFile input3.txt /tmp/test/input1.txt
# append 결과 확인
[yeon@yeon-host test]$ hdfs dfs -cat /tmp/test/input1.txt
aaaa
bbbb
input3 append
```
---
### mv - 파일 / 디렉토리 변경

- hdfs dfs -mv {hdfsPath} {hdfsPath}

```console
[yeon@yeon-host test]$ hdfs dfs -mv /tmp/test/input1.txt /tmp/test/input2.txt
# 변경 내용 확인
[yeon@yeon-host test]$ hdfs dfs -ls -R /tmp/
drwxr-xr-x   - yeon supergroup          0 2022-01-03 21:00 /tmp/test
-rw-r--r--   1 yeon supergroup         10 2022-01-03 20:43 /tmp/test/input2.txt
```
---
### get - hdfs의 파일 local로 가져오기

- hdfs dfs -get {hdfsPath} {localPath}

```console
[yeon@yeon-host test]$ hdfs dfs -get /tmp/test/input2.txt .
# local 파일 확인
[yeon@yeon-host test]$ ll
합계 8
-rw-rw-r--. 1 yeon yeon 10  1월  3 20:42 input1.txt
-rw-r--r--. 1 yeon yeon 10  1월  3 21:02 input2.txt
```
---
### chown - hdfs 파일 / 디렉토리 권한 변경

- hdfs dfs -chown [-R] {user} {hdfsPath}
- user에게 hdfs 디렉토리 소유권 변경
- [-R] 하위 모든 디렉토리 포함

```console
[yeon@yeon-host test]$ hdfs dfs -chown -R yeon2 /tmp/test/input2.txt
# 변경 내용 확인
[yeon@yeon-host test]$ hdfs dfs -ls /tmp/test/input2.txt
-rw-r--r--   1 yeon2 supergroup         10 2022-01-03 20:43 /tmp/test/input2.txt
```
---
### rm - 파일 / 디렉토리 삭제

- hdfs dfs -rm [-r] {user} {hdfsPath}
- [-r] 하위파일이 있는 디렉토리 삭제.

```console
[yeon@yeon-host test]$ hdfs dfs -rm /tmp/test/input2.txt
Deleted /tmp/test/input2.txt
# 파일이 삭제되었는지 확인
[yeon@yeon-host test]$ hdfs dfs -ls /tmp/test/
[yeon@yeon-host test]$
```
---
