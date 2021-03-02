---
layout: post
comments: true
title: Debezium CDC를 활용한 DB 변경 로그 시스템 구축
tags: [dbezium, elk, log]
---

### CDC(Change Data Capture)를 사용하는 이유

- DBMS의 Transaction log를 통해 변경데이터를 분석하기때문에 데이터 변경사항이 100% 보장됩니다.
- 쿼리를 통한 데이터 추출이 아니기때문에 DBMS에 부하가 적습니다.
- 쿼리를 통한 '폴링'방식의 데이터 추출시 최종 데이터 변경 내용만 조회 되기 때문에 모든 데이터 변경 내용을 캡쳐하기 어려울수 있습니다. CDC는 기록되어있는 트랜잭션 로그를 통해 중간에 발생한 모든 변경로그를 조회 할 수 있습니다.
- 삭제된 데이터도 캡쳐가 가능합니다.
- 실시간 처리가 가능합니다.
- CDC를 사용할수있는 여러 오픈소스가 많으나, Kafka와 가장 친화적인 오픈소스로 Debezium이 최고

---

### 환경 구축하기

이번 예제의 목표는 `MySQL`의 모든 변경사항을 로그로 남기기위해 `CDC`와 `Kafka+ELK`스택을 사용하여 로그시스템을 구축하는데에 있습니다.

로컬환경에서 Docker를 통해 예제 환경을 구축해보겠습니다.

![1]({{ site.images | relative_url }}/posts/2021-03-02-dbzium-cdc/1.png)

테스트를 위한 환경은 debezium에서 tutorial로 제공하고 있습니다.
(https://debezium.io/documentation/reference/1.4/tutorial.html#starting-kafka)

**zookeeper run**

```
$ docker run --rm --name zookeeper -p 2181:2181 -p 2888:2888 -p 3888:3888 debezium/zookeeper:1.4
```

***Kafka run***

```
$ docker run --rm --name kafka -p 9092:9092 --link zookeeper:zookeeper debezium/kafka:1.4
```

**MySQL run**
```
$ docker run --rm --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=debezium -e MYSQL_USER=mysqluser -e MYSQL_PASSWORD=mysqlpw debezium/example-mysql:1.4
```

**Connector run**

```
$ docker run --rm --name connect -p 8083:8083 -e GROUP_ID=1 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets -e STATUS_STORAGE_TOPIC=my_connect_statuses --link zookeeper:zookeeper --link kafka:kafka --link mysql:mysql debezium/connect:1.4
```

**Connector 등록**

이제 Connector를 DB를 등록해두면 binlog 를 분석해 설정에 따른 CDC를 시작 할 수 있습니다.

tutorial 에서는 Debezium docker repository에서 설치한 MySQL에 기본으로 세팅되어있는 'inventory' 스키마의 CDC를 등록해보겠습니다.

```
$ curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{ "name": "inventory-connector", "config": { "connector.class": "io.debezium.connector.mysql.MySqlConnector", "tasks.max": "1", "database.hostname": "mysql", "database.port": "3306", "database.user": "debezium", "database.password": "dbz", "database.server.id": "184054", "database.server.name": "dbserver1", "database.include.list": "inventory", "database.history.kafka.bootstrap.servers": "kafka:9092", "database.history.kafka.topic": "dbhistory.inventory" } }'
```

등록 json 정보
```json
{
   "name":"inventory-connector",
   "config":{
      "connector.class":"io.debezium.connector.mysql.MySqlConnector",
      "tasks.max":"1",
      "database.hostname":"mysql",
      "database.port":"3306",
      "database.user":"debezium",
      "database.password":"dbz",
      "database.server.id":"184054",
      "database.server.name":"dbserver1",
      "database.include.list":"inventory",
      "database.history.kafka.bootstrap.servers":"kafka:9092",
      "database.history.kafka.topic":"schema-changes.inventory"
   }
}
```

아래 명령어를 통해 등록된 "inventory-connector"와 등록정보를 확인 할 수 있습니다.

```
$ curl -H "Accept:application/json" localhost:8083/connectors/
 
 
> ["inventory-connector"]
 
 
$ curl -i -X GET -H "Accept:application/json" localhost:8083/connectors/inventory-connector
 
 
> {"name":"inventory-connector","config":{"connector.class":"io.debezium.connector.mysql.MySqlConnector","database.user":"debezium","database.server.id":"184054","tasks.max":"1","database.hostname":"mysql","database.password":"dbz","database.history.kafka.bootstrap.servers":"kafka:9092","database.history.kafka.topic":"dbhistory.inventory","name":"inventory-connector","database.server.name":"dbserver1","database.port":"3306","database.include.list":"inventory"},"tasks":[{"connector":"inventory-connector","task":0}],"type":"source"}
```

위 등록까지 마치게되면 MySQL, Kafka에 여러 로그가 올라오는것이 보이게 될 것입니다.

---

## CDC 데이터 확인하기

Kafka에 등록된 토픽을 확인해보면 아래와 같습니다.


![2]({{ site.images | relative_url }}/posts/2021-03-02-dbzium-cdc/2.png)

- dbserver1 : 스키마 DDL 변경데이터
- dbserver1.inventory.products : inventory.products 테이블 변경데이터
- dbserver1.inventory.products_on_hand : inventory.products_on_hand 테이블 변경데이터
- dbserver1.inventory.customers : inventory.customers 테이블 변경데이터
- dbserver1.inventory.orders : inventory.orders 테이블 변경데이터


현재 inventory.customer에 등록된 데이터는 아래와 같습니다.


![3]({{ site.images | relative_url }}/posts/2021-03-02-dbzium-cdc/3.png)


토픽에 생성된 메세지들을 보기 위해 kafka에서 console-consumer를 실행시켜보면 다음과같은 같은 메세지를 확인 할 수 있습니다.

```
$ bin/kafka-console-consumer.sh --bootstrap-server 172.10.0.4:9092 --topic dbserver1.inventory.customers --from-beginning
```

```json
> {"schema":{"type":"struct","fields":[{"type":"struct","fields":[{"type":"int32","optional":false,"field":"id"},{"type":"string","optional":false,"field":"first_name"},{"type":"string","optional":false,"field":"last_name"},{"type":"string","optional":false,"field":"email"}],"optional":true,"name":"dbserver1.inventory.customers.Value","field":"before"},{"type":"struct","fields":[{"type":"int32","optional":false,"field":"id"},{"type":"string","optional":false,"field":"first_name"},{"type":"string","optional":false,"field":"last_name"},{"type":"string","optional":false,"field":"email"}],"optional":true,"name":"dbserver1.inventory.customers.Value","field":"after"},{"type":"struct","fields":[{"type":"string","optional":false,"field":"version"},{"type":"string","optional":false,"field":"connector"},{"type":"string","optional":false,"field":"name"},{"type":"int64","optional":false,"field":"ts_ms"},{"type":"string","optional":true,"name":"io.debezium.data.Enum","version":1,"parameters":{"allowed":"true,last,false"},"default":"false","field":"snapshot"},{"type":"string","optional":false,"field":"db"},{"type":"string","optional":true,"field":"table"},{"type":"int64","optional":false,"field":"server_id"},{"type":"string","optional":true,"field":"gtid"},{"type":"string","optional":false,"field":"file"},{"type":"int64","optional":false,"field":"pos"},{"type":"int32","optional":false,"field":"row"},{"type":"int64","optional":true,"field":"thread"},{"type":"string","optional":true,"field":"query"}],"optional":false,"name":"io.debezium.connector.mysql.Source","field":"source"},{"type":"string","optional":false,"field":"op"},{"type":"int64","optional":true,"field":"ts_ms"},{"type":"struct","fields":[{"type":"string","optional":false,"field":"id"},{"type":"int64","optional":false,"field":"total_order"},{"type":"int64","optional":false,"field":"data_collection_order"}],"optional":true,"field":"transaction"}],"optional":false,"name":"dbserver1.inventory.customers.Envelope"},"payload":{"before":null,"after":{"id":1001,"first_name":"Sally","last_name":"Thomas","email":"sally.thomas@acme.com"},"source":{"version":"1.4.1.Final","connector":"mysql","name":"dbserver1","ts_ms":0,"snapshot":"true","db":"inventory","table":"customers","server_id":0,"gtid":null,"file":"mysql-bin.000005","pos":154,"row":0,"thread":null,"query":null},"op":"c","ts_ms":1612758600095,"transaction":null}}
> ...
> ...
> {"schema":{"type":"struct","fields":[{"type":"struct","fields":[{"type":"int32","optional":false,"field":"id"},{"type":"string","optional":false,"field":"first_name"},{"type":"string","optional":false,"field":"last_name"},{"type":"string","optional":false,"field":"email"}],"optional":true,"name":"dbserver1.inventory.customers.Value","field":"before"},{"type":"struct","fields":[{"type":"int32","optional":false,"field":"id"},{"type":"string","optional":false,"field":"first_name"},{"type":"string","optional":false,"field":"last_name"},{"type":"string","optional":false,"field":"email"}],"optional":true,"name":"dbserver1.inventory.customers.Value","field":"after"},{"type":"struct","fields":[{"type":"string","optional":false,"field":"version"},{"type":"string","optional":false,"field":"connector"},{"type":"string","optional":false,"field":"name"},{"type":"int64","optional":false,"field":"ts_ms"},{"type":"string","optional":true,"name":"io.debezium.data.Enum","version":1,"parameters":{"allowed":"true,last,false"},"default":"false","field":"snapshot"},{"type":"string","optional":false,"field":"db"},{"type":"string","optional":true,"field":"table"},{"type":"int64","optional":false,"field":"server_id"},{"type":"string","optional":true,"field":"gtid"},{"type":"string","optional":false,"field":"file"},{"type":"int64","optional":false,"field":"pos"},{"type":"int32","optional":false,"field":"row"},{"type":"int64","optional":true,"field":"thread"},{"type":"string","optional":true,"field":"query"}],"optional":false,"name":"io.debezium.connector.mysql.Source","field":"source"},{"type":"string","optional":false,"field":"op"},{"type":"int64","optional":true,"field":"ts_ms"},{"type":"struct","fields":[{"type":"string","optional":false,"field":"id"},{"type":"int64","optional":false,"field":"total_order"},{"type":"int64","optional":false,"field":"data_collection_order"}],"optional":true,"field":"transaction"}],"optional":false,"name":"dbserver1.inventory.customers.Envelope"},"payload":{"before":null,"after":{"id":1004,"first_name":"Anne","last_name":"Kretchmar","email":"annek@noanswer.org"},"source":{"version":"1.4.1.Final","connector":"mysql","name":"dbserver1","ts_ms":0,"snapshot":"true","db":"inventory","table":"customers","server_id":0,"gtid":null,"file":"mysql-bin.000005","pos":154,"row":0,"thread":null,"query":null},"op":"c","ts_ms":1612758600095,"transaction":null}}
```

위 메세지 구조를 자세히 보면 row의 변경건에 대해 변경된 상세 변경 데이터가 있는것을 확인 하실수 있으실겁니다.

특히 `payload` 필드를 통해 before/after가 확실하게 기록되어 확인가능함을 볼 수 있습니다.

```json
"payload":{
     "before":null,
     "after":{
        "id":1001,
        "first_name":"Sally",
        "last_name":"Thomas",
        "email":"sally.thomas@acme.com"
     },
```

DB에 직접 Insert/Update/Delete를 통해 CDC message가 어떻게 출력되는지 확인해보도록 하겠습니다.

**INSERT**
```sql
insert into inventory.customers(id, first_name, last_name, email)
values(1005, 'Taes', 'Kim', 'taes@mail.com')
```

```json
{"schema":{"type":"struct","fields":[{"type":"struct","fields":[{"type":"int32","optional":false,"field":"id"},{"type":"string","optional":false,"field":"first_name"},{"type":"string","optional":false,"field":"last_name"},{"type":"string","optional":false,"field":"email"}],"optional":true,"name":"dbserver1.inventory.customers.Value","field":"before"},{"type":"struct","fields":[{"type":"int32","optional":false,"field":"id"},{"type":"string","optional":false,"field":"first_name"},{"type":"string","optional":false,"field":"last_name"},{"type":"string","optional":false,"field":"email"}],"optional":true,"name":"dbserver1.inventory.customers.Value","field":"after"},{"type":"struct","fields":[{"type":"string","optional":false,"field":"version"},{"type":"string","optional":false,"field":"connector"},{"type":"string","optional":false,"field":"name"},{"type":"int64","optional":false,"field":"ts_ms"},{"type":"string","optional":true,"name":"io.debezium.data.Enum","version":1,"parameters":{"allowed":"true,last,false"},"default":"false","field":"snapshot"},{"type":"string","optional":false,"field":"db"},{"type":"string","optional":true,"field":"table"},{"type":"int64","optional":false,"field":"server_id"},{"type":"string","optional":true,"field":"gtid"},{"type":"string","optional":false,"field":"file"},{"type":"int64","optional":false,"field":"pos"},{"type":"int32","optional":false,"field":"row"},{"type":"int64","optional":true,"field":"thread"},{"type":"string","optional":true,"field":"query"}],"optional":false,"name":"io.debezium.connector.mysql.Source","field":"source"},{"type":"string","optional":false,"field":"op"},{"type":"int64","optional":true,"field":"ts_ms"},{"type":"struct","fields":[{"type":"string","optional":false,"field":"id"},{"type":"int64","optional":false,"field":"total_order"},{"type":"int64","optional":false,"field":"data_collection_order"}],"optional":true,"field":"transaction"}],"optional":false,"name":"dbserver1.inventory.customers.Envelope"},"payload":{"before":null,"after":{"id":1005,"first_name":"Taes","last_name":"Kim","email":"taes@mail.com"},"source":{"version":"1.4.1.Final","connector":"mysql","name":"dbserver1","ts_ms":1612761405000,"snapshot":"false","db":"inventory","table":"customers","server_id":223344,"gtid":null,"file":"mysql-bin.000005","pos":355,"row":0,"thread":7,"query":null},"op":"c","ts_ms":1612761405212,"transaction":null}}

```

```json
"op":"c"
"payload":{"before":null,"after":{"id":1005,"first_name":"Taes","last_name":"Kim","email":"taes@mail.com"}
```
→ 기존 Insert 내용과 동일하게 Insert log가 나오는것을 확인 할 수 있습니다.


**UPDATE**
```sql
update inventory.customers
set email = 'taes-k@gmail.com'
where id=1005
```

```json
{"schema":{"type":"struct","fields":[{"type":"struct","fields":[{"type":"int32","optional":false,"field":"id"},{"type":"string","optional":false,"field":"first_name"},{"type":"string","optional":false,"field":"last_name"},{"type":"string","optional":false,"field":"email"}],"optional":true,"name":"dbserver1.inventory.customers.Value","field":"before"},{"type":"struct","fields":[{"type":"int32","optional":false,"field":"id"},{"type":"string","optional":false,"field":"first_name"},{"type":"string","optional":false,"field":"last_name"},{"type":"string","optional":false,"field":"email"}],"optional":true,"name":"dbserver1.inventory.customers.Value","field":"after"},{"type":"struct","fields":[{"type":"string","optional":false,"field":"version"},{"type":"string","optional":false,"field":"connector"},{"type":"string","optional":false,"field":"name"},{"type":"int64","optional":false,"field":"ts_ms"},{"type":"string","optional":true,"name":"io.debezium.data.Enum","version":1,"parameters":{"allowed":"true,last,false"},"default":"false","field":"snapshot"},{"type":"string","optional":false,"field":"db"},{"type":"string","optional":true,"field":"table"},{"type":"int64","optional":false,"field":"server_id"},{"type":"string","optional":true,"field":"gtid"},{"type":"string","optional":false,"field":"file"},{"type":"int64","optional":false,"field":"pos"},{"type":"int32","optional":false,"field":"row"},{"type":"int64","optional":true,"field":"thread"},{"type":"string","optional":true,"field":"query"}],"optional":false,"name":"io.debezium.connector.mysql.Source","field":"source"},{"type":"string","optional":false,"field":"op"},{"type":"int64","optional":true,"field":"ts_ms"},{"type":"struct","fields":[{"type":"string","optional":false,"field":"id"},{"type":"int64","optional":false,"field":"total_order"},{"type":"int64","optional":false,"field":"data_collection_order"}],"optional":true,"field":"transaction"}],"optional":false,"name":"dbserver1.inventory.customers.Envelope"},"payload":{"before":{"id":1005,"first_name":"Taes","last_name":"Kim","email":"taes@mail.com"},"after":{"id":1005,"first_name":"Taes","last_name":"Kim","email":"taes-k@gmail.com"},"source":{"version":"1.4.1.Final","connector":"mysql","name":"dbserver1","ts_ms":1612761555000,"snapshot":"false","db":"inventory","table":"customers","server_id":223344,"gtid":null,"file":"mysql-bin.000005","pos":650,"row":0,"thread":7,"query":null},"op":"u","ts_ms":1612761555847,"transaction":null}}

```

```json
"op":"u"
"payload":{"before":{"id":1005,"first_name":"Taes","last_name":"Kim","email":"taes@mail.com"},"after":{"id":1005,"first_name":"Taes","last_name":"Kim","email":"taes-k@gmail.com"}
```
→ 'op' 필드와 'payload'필드를 update가 수행됬음을 확인 할 수 있습니다.

**DELETE**
```sql
delete
from inventory.customers
where id=1005
```

```json
{"schema":{"type":"struct","fields":[{"type":"struct","fields":[{"type":"int32","optional":false,"field":"id"},{"type":"string","optional":false,"field":"first_name"},{"type":"string","optional":false,"field":"last_name"},{"type":"string","optional":false,"field":"email"}],"optional":true,"name":"dbserver1.inventory.customers.Value","field":"before"},{"type":"struct","fields":[{"type":"int32","optional":false,"field":"id"},{"type":"string","optional":false,"field":"first_name"},{"type":"string","optional":false,"field":"last_name"},{"type":"string","optional":false,"field":"email"}],"optional":true,"name":"dbserver1.inventory.customers.Value","field":"after"},{"type":"struct","fields":[{"type":"string","optional":false,"field":"version"},{"type":"string","optional":false,"field":"connector"},{"type":"string","optional":false,"field":"name"},{"type":"int64","optional":false,"field":"ts_ms"},{"type":"string","optional":true,"name":"io.debezium.data.Enum","version":1,"parameters":{"allowed":"true,last,false"},"default":"false","field":"snapshot"},{"type":"string","optional":false,"field":"db"},{"type":"string","optional":true,"field":"table"},{"type":"int64","optional":false,"field":"server_id"},{"type":"string","optional":true,"field":"gtid"},{"type":"string","optional":false,"field":"file"},{"type":"int64","optional":false,"field":"pos"},{"type":"int32","optional":false,"field":"row"},{"type":"int64","optional":true,"field":"thread"},{"type":"string","optional":true,"field":"query"}],"optional":false,"name":"io.debezium.connector.mysql.Source","field":"source"},{"type":"string","optional":false,"field":"op"},{"type":"int64","optional":true,"field":"ts_ms"},{"type":"struct","fields":[{"type":"string","optional":false,"field":"id"},{"type":"int64","optional":false,"field":"total_order"},{"type":"int64","optional":false,"field":"data_collection_order"}],"optional":true,"field":"transaction"}],"optional":false,"name":"dbserver1.inventory.customers.Envelope"},"payload":{"before":{"id":1005,"first_name":"Taes","last_name":"Kim","email":"taes-k@gmail.com"},"after":null,"source":{"version":"1.4.1.Final","connector":"mysql","name":"dbserver1","ts_ms":1612761928000,"snapshot":"false","db":"inventory","table":"customers","server_id":223344,"gtid":null,"file":"mysql-bin.000005","pos":977,"row":0,"thread":7,"query":null},"op":"d","ts_ms":1612761928476,"transaction":null}}

```

```json
"op":"d"
"payload":{"before":{"id":1005,"first_name":"Taes","last_name":"Kim","email":"taes-k@gmail.com"},"after":null
```
→ 'op' 필드와 'payload'필드를 delete가 수행됬음을 확인 할 수 있습니다.

---

### CDC 데이터를 ELK에 적재하기

해당 작업의 목적은, Catalog와 상품매칭 정보의 변경 로그를 기록하는것이기 때문에 해당 CDC를 통해 기타 써드파티의 sync를 맞추는 것이 아닌 CDC 로그 자체의 데이터를 기록해주기위해 ELK에 연동해보도록 하겠습니다.

다행히도, 메세지가 단순히 Json type이기 때문에 logstash 에서 json filter를 사용 할 수 있습니다. 

스키마의 변경은 없을거라 가정하고, 데이터 변경 추적이 목적이기때문에 CDC 메세지중 payload 부분만 필터링하여

최종적으로는 아래와같은 필드들로 구성하는것을 목표로 합니다.

```json
{
    "reg_dt":"2020-01-21 10:00:00"
    , "op":"c"
    , "before":null
    , "after":{"id":1005,"first_name":"Taes","last_name":"Kim","email":"taes-k@gmail.com"}
}
```

`logstash pipeline config`를 수정합니다.

```json
// logstash/pipeline/logstash.conf
 
input {
        kafka {
                bootstrap_servers =>  "10.200.8.5:9092"
                group_id => "logstash"
                topics => ["wsin-log-topic"]
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
                target => "parsedJson"
                add_field => {"op" => "%{[parsedJson][payload][op]}"}
        }
 
 
        if [op] == "c" {
                mutate {
                        add_field => {
                                "before" => null
                                "after" => "%{[parsedJson][payload][after]}"
                                "id" => "%{[parsedJson][payload][after][id]}"
                        }
                }
        }else if [op] == "u" {
                mutate {
                        add_field => {
                                "before" => "%{[parsedJson][payload][before]}"
                                "after" => "%{[parsedJson][payload][after]}"
                                "id" => "%{[parsedJson][payload][before][id]}"
                        }
                }
        }else if [op] == "d" {
                mutate {
                        add_field => {
                                "before" => "%{[parsedJson][payload][before]}"
                                "after" => null
                                "id" => "%{[parsedJson][payload][before][id]}"
                        }
                }
        }
 
 
        mutate {
                remove_field => ["message", "parsedJson"]
        }
}
 
 
output {
        elasticsearch {
                hosts => "elasticsearch:9200"
                index => "wsin-cdc-logs-%{+YYYY.MM.dd}"
                user => "elastic"
                password => "password"
        }
}
```
이후 elasticsearch에 적재된 메세지를 확인해보면 아래와 같이 확인됩니다.

```
$ curl "localhost:9200/customer-cdc-log-2021.02.15/_search?pretty"
```

```json
{
  "took" : 837,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "wsin-cdc-logs-2021.02.15",
        "_type" : "_doc",
        "_id" : "bhRapHcBACTd9K5BJ2Ke",
        "_score" : 1.0,
        "_source" : {
          "@timestamp" : "2021-02-15T06:21:47.336Z",
          "before" : "null",
          "after" : "{\"last_name\":\"Thomas\",\"id\":1001,\"first_name\":\"Sally\",\"email\":\"sally.thomas@acme.com\"}",
          "id" : "1001",
          "@version" : "1",
          "op" : "c"
        }
      },
      {
        "_index" : "wsin-cdc-logs-2021.02.15",
        "_type" : "_doc",
        "_id" : "bxRfpHcBACTd9K5BH2Km",
        "_score" : 1.0,
        "_source" : {
          "@timestamp" : "2021-02-15T06:27:13.338Z",
          "before" : "null",
          "after" : "{\"last_name\":\"Kim\",\"id\":1005,\"first_name\":\"Taes\",\"email\":\"taes@mail.com\"}",
          "id" : "1005",
          "@version" : "1",
          "op" : "c"
        }
      },
      {
        "_index" : "wsin-cdc-logs-2021.02.15",
        "_type" : "_doc",
        "_id" : "cBRfpHcBACTd9K5BNmLF",
        "_score" : 1.0,
        "_source" : {
          "@timestamp" : "2021-02-15T06:27:19.258Z",
          "before" : "{\"last_name\":\"Kim\",\"id\":1005,\"first_name\":\"Taes\",\"email\":\"taes@mail.com\"}",
          "after" : "{\"last_name\":\"Kim\",\"id\":1005,\"first_name\":\"Taes\",\"email\":\"taes-k@gmail.com\"}",
          "id" : "1005",
          "@version" : "1",
          "op" : "u"
        }
      },
      {
        "_index" : "wsin-cdc-logs-2021.02.15",
        "_type" : "_doc",
        "_id" : "cRRfpHcBACTd9K5BSmL-",
        "_score" : 1.0,
        "_source" : {
          "@timestamp" : "2021-02-15T06:27:24.435Z",
          "before" : "{\"last_name\":\"Kim\",\"id\":1005,\"first_name\":\"Taes\",\"email\":\"taes-k@gmail.com\"}",
          "after" : null,
          "id" : "1005",
          "@version" : "1",
          "op" : "d"
        }
      }
    ]
  }
}
```
처음에 원하던대로 `op`, `id`, `before`, `after` 필드를 가진 Document가 생성된것을 확인 할 수 있습니다.

이제 위 구축된 ES에서 id검색을 통해서 특정 id의 변경 상세 로그들을 확인 할 수 있게 되었습니다.


```
curl "localhost:9200/customer-cdc-log-2021.02.15/_search?q=id:1001&pretty"
```

```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 0.35667494,
    "hits" : [
      {
        "_index" : "wsin-cdc-logs-2021.02.15",
        "_type" : "_doc",
        "_id" : "bxRfpHcBACTd9K5BH2Km",
        "_score" : 0.35667494,
        "_source" : {
          "@timestamp" : "2021-02-15T06:27:13.338Z",
          "before" : "null",
          "after" : "{\"last_name\":\"Kim\",\"id\":1005,\"first_name\":\"Taes\",\"email\":\"taes@mail.com\"}",
          "id" : "1005",
          "@version" : "1",
          "op" : "c"
        }
      },
      {
        "_index" : "wsin-cdc-logs-2021.02.15",
        "_type" : "_doc",
        "_id" : "cBRfpHcBACTd9K5BNmLF",
        "_score" : 0.35667494,
        "_source" : {
          "@timestamp" : "2021-02-15T06:27:19.258Z",
          "before" : "{\"last_name\":\"Kim\",\"id\":1005,\"first_name\":\"Taes\",\"email\":\"taes@mail.com\"}",
          "after" : "{\"last_name\":\"Kim\",\"id\":1005,\"first_name\":\"Taes\",\"email\":\"taes-k@gmail.com\"}",
          "id" : "1005",
          "@version" : "1",
          "op" : "u"
        }
      },
      {
        "_index" : "wsin-cdc-logs-2021.02.15",
        "_type" : "_doc",
        "_id" : "cRRfpHcBACTd9K5BSmL-",
        "_score" : 0.35667494,
        "_source" : {
          "@timestamp" : "2021-02-15T06:27:24.435Z",
          "before" : "{\"last_name\":\"Kim\",\"id\":1005,\"first_name\":\"Taes\",\"email\":\"taes-k@gmail.com\"}",
          "after" : null,
          "id" : "1005",
          "@version" : "1",
          "op" : "d"
        }
      }
    ]
  }
}
```

