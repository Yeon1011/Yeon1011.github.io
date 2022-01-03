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
- CentOS에 Hadoop 설치

---
## 설치

### 설치 위치
/home/yeon/dev
