---
layout: post
comments: true
title: Vertica + Kafka 연동하기
tags: [db, vertica, kafka]
---

### 개요  

`Message producer` - `Kafka` - `Vertica`    

위와 같은 아키텍쳐가 있다고 가정했을때, 딱 보면 데이터들이 Kafka를 통해 들어오게되고 이를 Vertica DB에 저장하려고 하는 목적임을 아실수 있으실것 같습니다.  
일반적인 구성으로는 `producer` - `kafka` - `consumer` - `DB` 다음과같은 구조를 통해 Kafka의 메세지를 Consumer를 통해 소비시키는 방식으로 서비스가 이용되겠지만 Vertica는 Kafka 데이터를 읽어올수 있는 내장 모듈이 포함되어있습니다.   
  
위의 내용은 Vertica 공식 홈페이지에서 모두 제공하고 있습니다. (https://www.vertica.com/docs/9.2.x/HTML/Content/Authoring/KafkaIntegrationGuide/VerticaAndApacheKafka.htm?tocpath=Integrating%20with%20Apache%20Kafka%7C_____3)  

---

### Kafka 메세지 수동으로 가져오기

```sql
COPY public.from_kafka 
SOURCE KafkaSource(stream='iot_data|0|-2,iot_data|1|-2',
                    brokers='kafka01.example.com:9092',
                    duration=interval '10000 milliseconds') 
PARSER  KafkaAvroParser(schema_registry_url='http://localhost:8081/subjects/iot_data-value/versions/1')
REJECTED DATA AS TABLE public.rejections 
DIRECT NO COMMIT;
```

Vertica에서 제공하는 `COPY` 명령어를 통해 간단한 Kafka source만 설정해준다면 수동으로 kafka 메세지를 읽어올 수 있습니다.  
위의 소스에서 볼수 있다시피 읽어드린 데이터를 테이블에 바로 넣을수 있으며, custom PARSER 설정을 통해 메세지 형태에 따라 파싱을 할수있습니다.  
또한 오류가있는 메세지들을 별도의 테이블에 저장 시킬수도 있음을 확인 하실수 있습니다.  


---

### Kafka 메세지 자동으로 가져오기 (마이크로배치 구축)

1\. kafka 메세지 연결 확인  

배치 설정전, kafka와의 연결 상태를 확인할 수 있는 vertica에서 기본 라이브러리로 제공하는 `kafkacat`을 통해 브로커 연결을 테스트 해 줍니다.  

```c
/opt/vertica/packages/kafka/bin/kafkacat -b <host> -t <topic-name> -o beginning -e
```

2\. 스케쥴러 리소스 풀 설정  

스케쥴러를 실행시키기 위한 메모리 리소스풀을 설정해줍니다. 이때 vertica super user 권한이 필요합니다.   
해당 설정에따라 할당된 메모리를 통해 스케쥴러를 운용 할 수있게 됩니다.  

```c
vsql -c "CREATE RESOURCE POOL kafka_pool MEMORYSIZE '10%' PLANNEDCONCURRENCY 10 QUEUETIMEOUT 0;"
```

3\. property 파일 등록  

각 설정들에 이용할 property 파일을 작성합니다.   

```c
vi /tmp/sched.props
 
dbhost=127.0.0.1
dbport=5433
username={SuperUserName}
password={SuperUserPassword}
config-schema=kafka_config
```
  
4\. 스케쥴러 생성  

스케쥴러 기본 정보를 등록해줍니다.  

```c
/opt/vertica/packages/kafka/bin/vkconfig scheduler --create --conf /tmp/sched.props --config-schema kafka_config --frame-duration 00:00:60 --eof-timeout-ms 1 --operator vertica  --resource-pool kafka_pool --username {SuperUserName} --password {SuperUserPassword}
```

5\. 클러스터 설정 (Kafka 클러스터)  

Kafka 브로커 정보를 등록해줍니다.  

```c
/opt/vertica/packages/kafka/bin/vkconfig cluster --conf /tmp/sched.props --create  --hosts <host> --cluster kafka_cluster
```

6\. Datasource 설정 (topic)  
 
실제 데이터를 가지고올 source (kafka topic)을 설정해 줍니다.  

```c
/opt/vertica/packages/kafka/bin/vkconfig source --conf /tmp/sched.props --create  --source vertica-test-topic  --cluster kafka_cluster  --partitions 1
``` 

7\. Target 설정 (저장 위치)  
 
메세지를 저장시킬 vertica 테이블을 설정해 줍니다.   

```c
/opt/vertica/packages/kafka/bin/vkconfig target  --conf /tmp/sched.props  --create --target-schema public  --target-table kafka_table
```

8\. Load spec 설정 (파서)  
 
메세지 파서를 등록해 줍니다.  

```c
/opt/vertica/packages/kafka/bin/vkconfig load-spec --conf /tmp/sched.props --create --load-spec tsv-load-spec --filters "FILTER KafkaInsertDelimiters(delimiter=E'\n') delimiter E'\t'" --parser "" --load-method DIRECT
```

9\.Micro batch 설정  
 
마지막으로 위에서 정의해준 설정들을 조합하여 마이크로 배치를 설정해줍니다.  

```c
/opt/vertica/packages/kafka/bin/vkconfig microbatch --create --conf /tmp/sched.props --microbatch kafka_batch --target-schema public --target-table kafka_table --add-source vertica-test-topic --add-source-cluster kafka_cluster  --load-spec tsv-load-spec
```
10\. 실행  
 
```c
/opt/vertica/packages/kafka/bin/vkconfig launch --conf /tmp/sched.props & 
```

---

#### Kafka Consumer-group 설정 

Kafka consumer로써, Vertica 역시 consumer group을 지정 할 수 있습니다.    
위의 micro batch 설정 단계에서 consumer group id를 지정해줌으로써 여러개의 컨슈머를 하나의 그룹으로 묶어서 설정이 가능합니다.  

```c
/opt/vertica/packages/kafka/bin/vkconfig microbatch --create --conf /tmp/sched.props --microbatch vertica-batch-1 --target-schema public --target-table ep_file_kafka --add-source vertica-new-test-topic --add-source-cluster ep_kafka_cluster  --load-spec ep-load-spec --consumer-group-id vertica-batch
 
 
/opt/vertica/packages/kafka/bin/vkconfig microbatch --create --conf /tmp/sched.props --microbatch vertica-batch-2 --target-schema public --target-table ep_file_kafka --add-source vertica-new-test-topic --add-source-cluster ep_kafka_cluster  --load-spec ep-load-spec --consumer-group-id vertica-batch
 
 
/opt/vertica/packages/kafka/bin/vkconfig microbatch --create --conf /tmp/sched.props --microbatch vertica-batch-3 --target-schema public --target-table ep_file_kafka --add-source vertica-new-test-topic --add-source-cluster ep_kafka_cluster  --load-spec ep-load-spec --consumer-group-id vertica-batch
 
 
/opt/vertica/packages/kafka/bin/vkconfig microbatch --create --conf /tmp/sched.props --microbatch vertica-batch-4 --target-schema public --target-table ep_file_kafka --add-source vertica-new-test-topic --add-source-cluster ep_kafka_cluster  --load-spec ep-load-spec --consumer-group-id vertica-batch

```