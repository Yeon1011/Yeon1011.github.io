---
title: "[Grafana] 설치 및 구동 테스트"
author:
  name: Yeoni
  link: https://github.com/cotes2020
date: 2022-01-06 00:00:00 +0900
categories: [BigData, Grafana]
tags: [Grafana, Test]
math: true
mermaid: true
---


# Grafana 설치 및 테스트

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

```console
[yeon@yeon-host grafana]$ pwd
/home/yeon/lib/grafana
[yeon@yeon-host grafana] wget https://dl.grafana.com/enterprise/release/grafana-enterprise-8.3.3.linux-amd64.tar.gz
[yeon@yeon-host grafana] tar -zxvf grafana-enterprise-8.3.3.linux-amd64.tar.gz
```

## 구동
### 구동방법

```console
[yeon@yeon-host bin]$ pwd
/home/yeon/dev/grafana/bin
[yeon@yeon-host bin]$ grafana-server
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

- 방화벽 3000 포트 허용

```console
[root@yeon-host ~]# firewall-cmd --permanent --zone=public --add-port=3000/tcp
success
[root@yeon-host ~]# firewall-cmd --reload
success
```

- 다른 IP에서 접속했을 경우


![Desktop View](/assets/img/contents/BigData/Grafana/grafana-test/local1.png)
_다른IP 브라우저에서 url을 호출했을 경우_


### server로 구동

- 적용해보진 않음.

Start the server with systemd
To start the service and verify that the service has started:

```console
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl status grafana-server
# Configure the Grafana server to start at boot:

sudo systemctl enable grafana-server
```
