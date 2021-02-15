---
layout: post
comments: true
title: ELK를 사용한 로그 시스템 구축하기
tags: [log, elk, log]
---

### ELK 로그 환경

요즈음 로그 시스템을 만들때 아마도 가장 많이 사용되는 기술은 `ELK`일 것입니다.  
`ELK`에 대해서는 너무 잘 알려져있기 때문에 설명은 생략하겠습니다. (https://www.elastic.co/kr/what-is/elk-stack)  

이번 포스팅에서는 `Kafka` + `ELK` 를 통해서 로그 시스템을 구축하는 예제를 정리하려 합니다.    
예제에 사용될 서비스들은 모두 `Docker`를 통해 Local 환경에 스택들을 구성하여 사용할 예정입니다.

전체적인 서비스 환경은 아래와 같습니다.

![1]({{ site.images | relative_url }}/posts/2021-02-15-elk-log-stack/1.png)

---

### Kafka 환경 구축하기

먼저, 로그 메세지를 통신하기 위한 `Kafka`를 띄워보겠습니다.  

우선, kafka가 잘 구성되어있는 Docker image를 사용하기 위해 아래 git 을 통해 파일을 clone 합니다.
```c
$ git clone https://github.com/wurstmeister/kafka-docker && cd kafka-docker
```

다음, docker-compose 파일을 수정합니다.   

해당 예제 환경에서는 kafka single-node 만 사용할 예정으로 `'docker-compose-single-broker.yml'` 파일의 설정을 변경 합니다.
```c
// docker-compose-single-broker.yml
 
// 서버 host name 설정
KAFKA_ADVERTISED_HOST_NAME: 10.x.x.x
// 사용할 topic 설정 (토픽은 Kafka 구성 이후 생성할수도 있음)
KAFKA_CREATE_TOPICS: "log-topic:1:1"
```

다음, `docker-compose`를 위에서 설정한`'docker-compose-single-broker.yml'` 파일 으로 실행 합니다.
```c
$ docker-compose -f docker-compose-single-broker.yml up -d
```

`kafka`컨테이너가 정상 수행 되었는지 확인 합니다.
```c
$ docker ps
```
![2]({{ site.images | relative_url }}/posts/2021-02-15-elk-log-stack/2.png)


---

### ELK 환경 구축하기

이번에는 `ELK stack`을 마찬가지로 docker를 이용해 띄워보겠습니다.  

`ELK`가 잘 구성되어있는 Docker image를 사용하기 위해 아래 git 을 통해 파일을 clone 합니다.
```
$ git clone https://github.com/deviantony/docker-elk.git && cd docker-elk
```

다음, `Elasticsearch` 파일을 수정합니다.

xpack 설정파일 주석처리    
```c
// elasticsearch/config/elasticsearch.yml
---
## Default Elasticsearch configuration from Elasticsearch base image.
## https://github.com/elastic/elasticsearch/blob/master/distribution/docker/src/docker/config/elasticsearch.yml
#
cluster.name: "docker-cluster"
network.host: 0.0.0.0
 
 
## X-Pack settings
## see https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-xpack.html
#
#xpack.license.self_generated.type: trial
#xpack.security.enabled: true
#xpack.monitoring.collection.enabled: true
```

nori 분석기 추가
```c
// elasticsearch/Dockerfile
 
ARG ELK_VERSION
   
# https://www.docker.elastic.co/
FROM docker.elastic.co/elasticsearch/elasticsearch:${ELK_VERSION}
 
# Add your elasticsearch plugins setup here
# Example: RUN elasticsearch-plugin install analysis-icu
RUN elasticsearch-plugin install analysis-nori
```

다음, `Logstash` 파일을 수정합니다.

xpack 설정파일 주석처리 
```c

// logstash/config/logstash.yml
---
## Default Logstash configuration from Logstash base image.
## https://github.com/elastic/logstash/blob/master/docker/data/logstash/config/logstash-full.yml
#
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.hosts: [ "http://elasticsearch:9200" ]
 
 
## X-Pack security credentials
#
#xpack.monitoring.enabled: true
#xpack.monitoring.elasticsearch.username: elastic
#xpack.monitoring.elasticsearch.password: changeme
```

pipeline 설정 내용 세팅
```c
// logstash/pipeline/logstash.config
 
input {
        kafka {
                bootstrap_servers =>  "10.x.x.x:9092" // kafka ip
                group_id => "logstash"
                topics => ["log-topic"]
                consumer_threads => 1
        }
        tcp {
                port => 5000
        }
}
 
## Add your filters / logstash plugins configuration here
output {
        elasticsearch {
                hosts => "elasticsearch:9200"
                index => "logs-%{+YYYY.MM.dd}"
                user => "elastic"
                password => "password"
        }
}
```

다음, `Kibana` 파일을 수정합니다.

xpack 설정파일 주석처리 
```c
// kibana/config/kibana.yml
 
 
---
## Default Kibana configuration from Kibana base image.
## https://github.com/elastic/kibana/blob/master/src/dev/build/tasks/os_packages/docker_generator/templates/kibana_yml.template.ts
#
server.name: kibana
server.host: 0.0.0.0
elasticsearch.hosts: [ "http://elasticsearch:9200" ]
monitoring.ui.container.elasticsearch.enabled: true
 
 
## X-Pack security credentials
#
#elasticsearch.username: elastic
#elasticsearch.password: changeme
```


이제 `ELK` 각각의 구성설정을 마쳤으니 `docker-compose` 파일을 수정해줍니다.  

elastic password 수정
```c
// docker-compose
...
      ELASTIC_PASSWORD: password
...


// docker-stack
...
      ELASTIC_PASSWORD: password
...
```


다음, `docker-compose`를 실행 합니다.
```c
$ docker-compose up -d
```

`ELK` 컨테이너가 정상 수행 되었는지 확인 합니다.
```c
$ docker ps
```
![3]({{ site.images | relative_url }}/posts/2021-02-15-elk-log-stack/3.png)

---

### Log 적재하기

이제 `Kafka` + `ELK` 환경이 갖추어졌으니 로그 시스템이 다 갖춰졌습니다.  

다음과 같은 로직으로 위 시스템을 검증해보려 합니다.

- Kafka 에서 message 생성
- ES Index 생성 확인
- ES Document 생성 확인
- Kibana 데이터 조회

#### 1\. Kafka Message 생성  

Kafka container bash 접속
```c
// docker ps 를 통해 container id를 조회 후 bash를 실행
 
$ sudo docker exec -it 0c0af88851f0 /bin/bash
```

Kafka console producer 실행
```
$ kafka-console-producer.sh --broker-list 10.200.8.5:9092 --topic log-topic
```

log message 발행
```json
> {"reg_dt":"2020-01-21 10:00:00","ldept_nm":"개발본부","mdept_nm":"개발실2","sdept_nm":"개발팀","reg_id":"test_id_1","reg_nm":"테스터1","emp_id":"202101010001","product_id":1000001,"product_name":"테스트 상품1","task_dtl":"상품 생성"}
> {"reg_dt":"2020-01-21 10:05:03","ldept_nm":"개발본부","mdept_nm":"개발실2","sdept_nm":"개발팀","reg_id":"test_id_1","reg_nm":"테스터1","emp_id":"202101010001","product_id":1000001,"product_name":"테스트 상품1","task_dtl":"상품 수정"}
> {"reg_dt":"2020-01-21 10:05:03","ldept_nm":"개발본부","mdept_nm":"개발실2","sdept_nm":"개발팀","reg_id":"test_id_1","reg_nm":"테스터1","emp_id":"202101010001","product_id":1000002,"product_name":"테스트 상품2","task_dtl":"상품 생성"}
> {"reg_dt":"2020-01-21 10:05:13","ldept_nm":"개발본부","mdept_nm":"개발실2","sdept_nm":"개발팀","reg_id":"test_id_1","reg_nm":"테스터1","emp_id":"202101010001","product_id":1000003,"product_name":"테스트 상품3","task_dtl":"상품 생성"}
> {"reg_dt":"2020-01-21 10:17:03","ldept_nm":"개발본부","mdept_nm":"개발실2","sdept_nm":"개발팀","reg_id":"test_id_1","reg_nm":"테스터1","emp_id":"202101010001","product_id":1000002,"product_name":"테스트 상품2","task_dtl":"상품 수정"}
```


#### 2\. Elasticsearch Index 생성 확인

`Elasticsearch` index 조회
```c
$ curl localhost:9200/_cat/indices?v
```

결과
```c
health status index                           uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   logs-2021.02.05            w4Oy85xWThaXT9nat6yFTg   1   1          5            0     15.8kb         15.8kb
green  open   .apm-custom-link                vvCTctk7R--fI17vdW-QQg   1   0          0            0       208b           208b
green  open   .kibana_task_manager_1          2I5nX90PQlKWg2ExJIPd_Q   1   0          5          537    112.3kb        112.3kb
green  open   .apm-agent-configuration        Hjmli6SFRtaRW8CyVWV1DA   1   0          0            0       208b           208b
green  open   .kibana_1                       bUDjQkl7Skm7EL14oSa1sg   1   0          9            0      2.1mb          2.1mb
green  open   .kibana-event-log-7.10.2-000001 LPNt7Q0zSqS53vv4vURPVQ   1   0          1            0      5.6kb          5.6kb
```
로그스태시에 설정한대로 `logs-2021.02.05` 인덱스가 생성된것을 확인 할 수 있습니다.  



#### 3\. ES Document 생성 확인

이제 index 내부에 Document가 실제로 생성되었는지 확인할 차례입니다.
```
$ curl localhost:9200/logs-2021.02.05/_search?pretty
```

결과
```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "logs-2021.02.05",
        "_type" : "_doc",
        "_id" : "vyCRcHcBRkK_WRzTT99_",
        "_score" : 1.0,
        "_source" : {
          "@version" : "1",
          "message" : "{"reg_dt":"2020-01-21 10:00:00","ldept_nm":"개발본부","mdept_nm":"개발실2","sdept_nm":"개발팀","reg_id":"test_id_1","reg_nm":"테스터1","emp_id":"202101010001","product_id":1000001,"product_name":"테스트 상품1","task_dtl":"상품 생성"}",
          "@timestamp" : "2021-02-05T05:01:46.492Z"
        }
      },
      {
        "_index" : "ogs-2021.02.05",
        "_type" : "_doc",
        "_id" : "xiCUcHcBRkK_WRzTqN8L",
        "_score" : 1.0,
        "_source" : {
          "@version" : "1",
          "message" : "{"reg_dt":"2020-01-21 10:05:03","ldept_nm":"개발본부","mdept_nm":"개발실2","sdept_nm":"개발팀","reg_id":"test_id_1","reg_nm":"테스터1","emp_id":"202101010001","product_id":1000001,"product_name":"테스트 상품1","task_dtl":"상품 수정"}",
          "@timestamp" : "2021-02-05T05:05:26.431Z"
        }
      },
      {
        "_index" : "logs-2021.02.05",
        "_type" : "_doc",
        "_id" : "xyCUcHcBRkK_WRzTt9_3",
        "_score" : 1.0,
        "_source" : {
          "@version" : "1",
          "message" : "{"reg_dt":"2020-01-21 10:05:03","ldept_nm":"개발본부","mdept_nm":"개발실2","sdept_nm":"개발팀","reg_id":"test_id_1","reg_nm":"테스터1","emp_id":"202101010001","product_id":1000002,"product_name":"테스트 상품2","task_dtl":"상품 생성"}",
          "@timestamp" : "2021-02-05T05:05:30.510Z"
        }
      },
      {
        "_index" : "logs-2021.02.05",
        "_type" : "_doc",
        "_id" : "yCCVcHcBRkK_WRzTxN8f",
        "_score" : 1.0,
        "_source" : {
          "@version" : "1",
          "message" : "{"reg_dt":"2020-01-21 10:05:13","ldept_nm":"개발본부","mdept_nm":"개발실2","sdept_nm":"개발팀","reg_id":"test_id_1","reg_nm":"테스터1","emp_id":"202101010001","product_id":1000003,"product_name":"테스트 상품3","task_dtl":"상품 생성"}",
          "@timestamp" : "2021-02-05T05:06:39.157Z"
        }
      },
      {
        "_index" : "logs-2021.02.05",
        "_type" : "_doc",
        "_id" : "1SC6cHcBRkK_WRzTk9-G",
        "_score" : 1.0,
        "_source" : {
          "@version" : "1",
          "message" : "{"reg_dt":"2020-01-21 10:17:03","ldept_nm":"개발본부","mdept_nm":"개발실2","sdept_nm":"개발팀","reg_id":"test_id_1","reg_nm":"테스터1","emp_id":"202101010001","product_id":1000002,"product_name":"테스트 상품2","task_dtl":"상품 수정"}",
          "@timestamp" : "2021-02-05T05:46:51.547Z"
        }
      }
    ]
  }
}
```

위 검증로직을 통해 ES에 정상적으로 로그 메세지가 적재되는것을 확인하였습니다.

---

### 검색 가능한 ES 적재

위 상태에서는 넘겨준 message가 ES 에서 단순히 message 에 적재되고 있습니다.
검색이 가능하도록 필드를 구성하여 적재시켜보도록 하겠습니다.

현재 log 메세지가 Json 형태로 넘어오고 있기 때문에 단순히 logstash 에서 json 을 파싱해주는 filter만 추가해주면 됩니다.

```c
// logstash/pipeline/logstash.config
 
 
input {
        kafka {
                bootstrap_servers =>  "10.200.8.5:9092"
                group_id => "logstash"
                topics => ["log-topic"]
                consumer_threads => 1
        }
 
        tcp {
                port => 5000
        }
}
 
 
## Add your filters / logstash plugins configuration here
filter {
      json {
        source => "message"
      }
}
 
output {
        elasticsearch {
                hosts => "elasticsearch:9200"
                index => "logs-%{+YYYY.MM.dd}"
                user => "elastic"
                password => "password"
        }
}
```
이후 메세지를 다시 생성해 ES 를 조회해보면 다음과같은 결과를 확인 할 수 있습니다.

```json
{
  "took" : 385,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "logs-2021.02.05",
        "_type" : "_doc",
        "_id" : "mXlUcXcBQOlug89tq04l",
        "_score" : 1.0,
        "_source" : {
          "product_name" : "테스트 상품1",
          "reg_id" : "test_id_1",
          "message" : "{\"reg_dt\":\"2020-01-21 10:00:00\",\"ldept_nm\":\"개발본부\",\"mdept_nm\":\"개발실2\",\"sdept_nm\":\"개발팀\",\"reg_id\":\"test_id_1\",\"reg_nm\":\"테스터1\",\"emp_id\":\"202101010001\",\"product_id\":1000001,\"product_name\":\"테스트 상품1\",\"task_dtl\":\"상품 생성\"}",
          "@timestamp" : "2021-02-05T08:35:09.744Z",
          "ldept_nm" : "개발본부",
          "product_id" : 1000001,
          "emp_id" : "202101010001",
          "task_dtl" : "상품 생성",
          "@version" : "1",
          "mdept_nm" : "개발실2",
          "reg_nm" : "테스터1",
          "reg_dt" : "2020-01-21 10:00:00",
          "sdept_nm" : "개발팀"
        }
      },
      {
        "_index" : "logs-2021.02.05",
        "_type" : "_doc",
        "_id" : "mnlVcXcBQOlug89tWk7l",
        "_score" : 1.0,
        "_source" : {
          "product_name" : "테스트 상품1",
          "reg_id" : "test_id_1",
          "message" : "{\"reg_dt\":\"2020-01-21 10:05:03\",\"ldept_nm\":\"개발본부\",\"mdept_nm\":\"개발실2\",\"sdept_nm\":\"개발팀\",\"reg_id\":\"test_id_1\",\"reg_nm\":\"테스터1\",\"emp_id\":\"202101010001\",\"product_id\":1000001,\"product_name\":\"테스트 상품1\",\"task_dtl\":\"상품 수정\"}",
          "@timestamp" : "2021-02-05T08:35:55.128Z",
          "ldept_nm" : "개발본부",
          "product_id" : 1000001,
          "emp_id" : "202101010001",
          "task_dtl" : "상품 수정",
          "@version" : "1",
          "mdept_nm" : "개발실2",
          "reg_nm" : "테스터1",
          "reg_dt" : "2020-01-21 10:05:03",
          "sdept_nm" : "개발팀"
        }
      },
      {
        "_index" : "-logs-2021.02.05",
        "_type" : "_doc",
        "_id" : "m3lVcXcBQOlug89tc07y",
        "_score" : 1.0,
        "_source" : {
          "product_name" : "테스트 상품2",
          "reg_id" : "test_id_1",
          "message" : "{\"reg_dt\":\"2020-01-21 10:05:03\",\"ldept_nm\":\"개발본부\",\"mdept_nm\":\"개발실2\",\"sdept_nm\":\"개발팀\",\"reg_id\":\"test_id_1\",\"reg_nm\":\"테스터1\",\"emp_id\":\"202101010001\",\"product_id\":1000002,\"product_name\":\"테스트 상품2\",\"task_dtl\":\"상품 생성\"}",
          "@timestamp" : "2021-02-05T08:36:01.545Z",
          "ldept_nm" : "개발본부",
          "product_id" : 1000002,
          "emp_id" : "202101010001",
          "task_dtl" : "상품 생성",
          "@version" : "1",
          "mdept_nm" : "개발실2",
          "reg_nm" : "테스터1",
          "reg_dt" : "2020-01-21 10:05:03",
          "sdept_nm" : "개발팀"
        }
      },
      {
        "_index" : "logs-2021.02.05",
        "_type" : "_doc",
        "_id" : "nHlVcXcBQOlug89tp04A",
        "_score" : 1.0,
        "_source" : {
          "product_name" : "테스트 상품3",
          "reg_id" : "test_id_1",
          "message" : "{\"reg_dt\":\"2020-01-21 10:05:13\",\"ldept_nm\":\"개발본부\",\"mdept_nm\":\"개발실2\",\"sdept_nm\":\"개발팀\",\"reg_id\":\"test_id_1\",\"reg_nm\":\"테스터1\",\"emp_id\":\"202101010001\",\"product_id\":1000003,\"product_name\":\"테스트 상품3\",\"task_dtl\":\"상품 생성\"}",
          "@timestamp" : "2021-02-05T08:36:14.613Z",
          "ldept_nm" : "개발본부",
          "product_id" : 1000003,
          "emp_id" : "202101010001",
          "task_dtl" : "상품 생성",
          "@version" : "1",
          "mdept_nm" : "개발실2",
          "reg_nm" : "테스터1",
          "reg_dt" : "2020-01-21 10:05:13",
          "sdept_nm" : "개발팀"
        }
      },
      {
        "_index" : "logs-2021.02.05",
        "_type" : "_doc",
        "_id" : "nXlVcXcBQOlug89twk4g",
        "_score" : 1.0,
        "_source" : {
          "product_name" : "테스트 상품2",
          "reg_id" : "test_id_1",
          "message" : "{\"reg_dt\":\"2020-01-21 10:17:03\",\"ldept_nm\":\"개발본부\",\"mdept_nm\":\"개발실2\",\"sdept_nm\":\"개발팀\",\"reg_id\":\"test_id_1\",\"reg_nm\":\"테스터1\",\"emp_id\":\"202101010001\",\"product_id\":1000002,\"product_name\":\"테스트 상품2\",\"task_dtl\":\"상품 수정\"}",
          "@timestamp" : "2021-02-05T08:36:21.559Z",
          "ldept_nm" : "개발본부",
          "product_id" : 1000002,
          "emp_id" : "202101010001",
          "task_dtl" : "상품 수정",
          "@version" : "1",
          "mdept_nm" : "개발실2",
          "reg_nm" : "테스터1",
          "reg_dt" : "2020-01-21 10:17:03",
          "sdept_nm" : "개발팀"
        }
      }
    ]
  }
}
```
원하던대로 필드가 구성되어 ES에 적재된것을 확인 할 수 있습니다.   
이제 `Elasticsearch`에 원하는 검색 조건을 추가하여 조회 할 수 있습니다.

```c
$ curl localhost:9200/logs-2021.02.05/_search?q=product_id:1000001
```

```json
{
   "took":1,
   "timed_out":false,
   "_shards":{
      "total":1,
      "successful":1,
      "skipped":0,
      "failed":0
   },
   "hits":{
      "total":{
         "value":2,
         "relation":"eq"
      },
      "max_score":1.0,
      "hits":[
         {
            "_index":"logs-2021.02.05",
            "_type":"_doc",
            "_id":"mXlUcXcBQOlug89tq04l",
            "_score":1.0,
            "_source":{
               "product_name":"테스트 상품1",
               "reg_id":"test_id_1",
               "message":"{\"reg_dt\":\"2020-01-21 10:00:00\",\"ldept_nm\":\"개발본부\",\"mdept_nm\":\"개발실2\",\"sdept_nm\":\"개발팀\",\"reg_id\":\"test_id_1\",\"reg_nm\":\"테스터1\",\"emp_id\":\"202101010001\",\"product_id\":1000001,\"product_name\":\"테스트 상품1\",\"task_dtl\":\"상품 생성\"}",
               "@timestamp":"2021-02-05T08:35:09.744Z",
               "ldept_nm":"개발본부",
               "product_id":1000001,
               "emp_id":"202101010001",
               "task_dtl":"상품 생성",
               "@version":"1",
               "mdept_nm":"개발실2",
               "reg_nm":"테스터1",
               "reg_dt":"2020-01-21 10:00:00",
               "sdept_nm":"개발팀"
            }
         },
         {
            "_index":"logs-2021.02.05",
            "_type":"_doc",
            "_id":"mnlVcXcBQOlug89tWk7l",
            "_score":1.0,
            "_source":{
               "product_name":"테스트 상품1",
               "reg_id":"test_id_1",
               "message":"{\"reg_dt\":\"2020-01-21 10:05:03\",\"ldept_nm\":\"개발본부\",\"mdept_nm\":\"개발실2\",\"sdept_nm\":\"개발팀\",\"reg_id\":\"test_id_1\",\"reg_nm\":\"테스터1\",\"emp_id\":\"202101010001\",\"product_id\":1000001,\"product_name\":\"테스트 상품1\",\"task_dtl\":\"상품 수정\"}",
               "@timestamp":"2021-02-05T08:35:55.128Z",
               "ldept_nm":"개발본부",
               "product_id":1000001,
               "emp_id":"202101010001",
               "task_dtl":"상품 수정",
               "@version":"1",
               "mdept_nm":"개발실2",
               "reg_nm":"테스터1",
               "reg_dt":"2020-01-21 10:05:03",
               "sdept_nm":"개발팀"
            }
         }
      ]
   }
}
```