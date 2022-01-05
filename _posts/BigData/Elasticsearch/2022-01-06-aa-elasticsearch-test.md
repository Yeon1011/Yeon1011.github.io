---
title: "[Elasticsearch] 구동 테스트 및 오류"
author:
  name: Yeoni
  link: https://github.com/cotes2020
date: 2022-01-06 00:00:00 +0900
categories: [BigData, Elasticsearch]
tags: [Elasticsearch, Test]
math: true
mermaid: true
---


# Elasticsearch 설치 및 테스트

---
## 환경 구성

| Tools                        | Version          |
|:-----------------------------|-----------------:|
| VirtualBox                   | 6.1              |
| CentOS                       | 7                |
| Elasticsearch                | 7.16.2           |
| Grafana                      | 8.3.3            |

---
## 사용포트 정리

| Program                      | Port Number      |
|:-----------------------------|-----------------:|
| Elasticsearch                | 9200             |
| Grafana                      | 3000             |

---
## 설치

### 구동방법

```console
[yeon@yeon-host elasticsearch]$ pwd
/home/yeon/dev/elasticsearch
[yeon@yeon-host elasticsearch]$ bin/elasticsearch
```

### 구동후 접속 테스트

```console
[yeon@yeon-host elasticsearch]$ curl -put 'http://192.168.25.2:9200/'
Enter host password for user 't':
{
  "name" : "yeon-host",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "LAxAWhHnQe6SjY6YxUbzUw",
  "version" : {
    "number" : "7.16.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "2b937c44140b6559905130a8650c64dbd0879cfb",
    "build_date" : "2021-12-18T19:42:46.604893745Z",
    "build_snapshot" : false,
    "lucene_version" : "8.10.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

- 방화벽 9200 포트 허용

```console
[root@yeon-host ~]# firewall-cmd --permanent --zone=public --add-port=9200/tcp
success
[root@yeon-host ~]# firewall-cmd --reload
success
```

- 다른 IP에서 접속했을 경우
![Desktop View](/assets/img/contents/BigData/Elasticsearch/elasticsearch-test/local1.png)
_다른IP 브라우저에서 url을 호출했을 경우_



### 구동 오류(1)

```console
[2022-01-06T00:15:11,369][INFO ][o.e.b.BootstrapChecks    ] [yeon-host] bound or publishing to a non-loopback address, enforcing bootstrap checks

ERROR: [2] bootstrap checks failed. You must address the points described in the following [2] lines before starting Elasticsearch.
bootstrap check failure [1] of [2]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
bootstrap check failure [2] of [2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
ERROR: Elasticsearch did not exit normally - check the logs at /home/yeon/dev/elasticsearch/logs/elasticsearch.log
[2022-01-06T00:15:11,474][INFO ][o.e.n.Node               ] [yeon-host] stopping ...
[2022-01-06T00:15:11,513][INFO ][o.e.n.Node               ] [yeon-host] stopped
[2022-01-06T00:15:11,514][INFO ][o.e.n.Node               ] [yeon-host] closing ...
[2022-01-06T00:15:11,587][INFO ][o.e.n.Node               ] [yeon-host] closed
```

- max file 개수가 부족하여 구동을 못한다는 오류..

### 해결(1)

- `/etc/security/limits.conf` 파일 수정. - root로 작업

- 참고 : ulimit 는 유저(쉘,프로세스)에 대허서 할당할 자원의 한계를 정해서 다중 프로그램/사용자를 기본으로하는 리눅스 시스템에서 과부하를 막아주는 설정


```shell
...
yeon soft memlock unlimited
yeon hard memlock unlimited
yeon - nofile 65535
...
```

- `ulimit -a` : 유저의 자원 할당량을 조회하는 명령어.


```console
[yeon@yeon-host ~]$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 15064
max locked memory       (kbytes, -l) unlimited
max memory size         (kbytes, -m) unlimited
open files                      (-n) 65535 # 추가됨
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 4096
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

---

### 구동 오류(2)

```console
ERROR: [1] bootstrap checks failed. You must address the points described in the following [1] lines before starting Elasticsearch.
bootstrap check failure [1] of [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
ERROR: Elasticsearch did not exit normally - check the logs at /home/yeon/dev/elasticsearch/logs/elasticsearch.log
[2022-01-06T00:29:48,072][INFO ][o.e.n.Node               ] [yeon-host] stopping ...
[2022-01-06T00:29:48,122][INFO ][o.e.n.Node               ] [yeon-host] stopped
[2022-01-06T00:29:48,122][INFO ][o.e.n.Node               ] [yeon-host] closing ...
[2022-01-06T00:29:48,144][INFO ][o.e.n.Node               ] [yeon-host] closed
```

### 해결(2)

- `/etc/sysctl.conf`의 vm.max_map_count 값 수정. - root로 작업

```shell
...
vm.max_map_count=262144
```
