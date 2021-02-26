---
layout: post
comments: true
title: avro schema registry 설치하기
tags: [avro, schema registry]
---

### confluent schema registry 단독 설치

`Confluent`사 에서 제공하는 `schema-registry`를 사용하기위해 찾아보던중, 공식 사이트에서 `schema-registry`를 단독으로 설치하는 예제를 제공해주지 않아 정리하여 글을 남깁니다.  

공식 사이트에서는 `confluent-platform`을 통해 `zookeeper`, `kafka`, `management-console`, `schema-registry`를 통합하여 설치 하게 되어 있는데 별도로 Kafka를 설치하여 사용중인 서버에 `schema-registry`를 구성하기 위함입니다.

https://docs.confluent.io/platform/current/installation/installing_cp/overview.html#installation

**1. public key 등록**

```
$ wget -qO - https://packages.confluent.io/deb/6.1/archive.key | sudo apt-key add -
```

**2. apt 등록**

```
$ sudo add-apt-repository "deb [arch=amd64] https://packages.confluent.io/deb/6.1 stable main"
```

**3. apt update**

```
$ sudo apt-get update
```

**4. confluent-schema-registry install**

```
$ apt-get install confluent-schema-registry
```

**5. schema-registry 설정**

```
$ vi /etc/schema-registry/schema-registry.properties

kafkastore.bootstrap.servers=PLAINTEXT://hostname:9092,SSL://hostname2:9092
```

**6. schema-registry 실행**

```
$ sudo systemctl start confluent-schema-registry
```

**7. 서비스 실행 확인**

```
$ sudo systemctl status confluent-schema-registry
```
