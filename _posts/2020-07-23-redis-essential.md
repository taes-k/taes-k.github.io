---
layout: post
comments: true
title: Redis를 운영하면서 반드시 알아야할 내용들
tags: [redis]
---

### Redis 특성

Redis는 In-Memory Store로써, 기본적으로 `빠른` 데이터 입출력을 제공합니다. 그와 더불어 기본적으로 해시기반의 `Key-Value` 형식의 저장방식을 제공하면서 키 등록/ 조회에 O(1)의 성능을 보장합니다.  

- [Get](https://redis.io/commands/get) : O(1)
- [SET](https://redis.io/commands/set) : O(1)

Value로써 사용될 수 있는 자료구조로는 일반 String 뿐만 아니라 Set, List, Hash 등의 collection들도 지원하며, 이또한 O(1)의 성능을 보장합니다.

- [SADD](https://redis.io/commands/sadd) : O(1)
- [LSET](https://redis.io/commands/lset) : O(1)
- [HSET](https://redis.io/commands/hset) : O(1)
- [HGET](https://redis.io/commands/hget) : O(1)

위의 성능을 기반으로 Redis는 일반적으로 `cache`로써 많이 사용되고 Persistent 기능을 사용하여 `Database`로써 사용되기도 합니다.  

추가로, pub/sub 기능도 지원하여 `message-broker`의 역할로써도 사용 될 수 있습니다.

---

### Redis는 싱글스레드

Redis 4.0 부터는 기본적으로 4개의 쓰레드로 동작하지만 일반 명령어를 처리하는 `메인쓰레드 1개`와 별도의 시스템 명령들을 사용하는 전용 `sub trhead 3개` 로써, 실제로 사용자가 사용하는 명령어들을 `싱글쓰레드`로 동작한다고 생각하면 됩니다.

( Redis 6.0 - [2020.04] 부터 ThreadedIO가 추가되어 사용자 명령이 멀티쓰레드가 지원이됩니다. 하지만 명령어를 실행하는 코어부분은 여전히 single thread로 동작하며, I/O Socket read/write를 할때 멀티쓰레드로 동작하여 전반적인 성능을 향상되었습니다. https://redislabs.com/blog/diving-into-redis-6/
)

위에서 알아본 Redis의 특성중 `성능`면을 보면, Redis의 주요 기능들이 O(1)의 복잡도로 매우 빠르게 처리되기에 고성능으로 데이터를 atomic 하게 유지하는 목적으로 싱글스레드에서 최고 효율을 낼 수 있도로 설계한것이 이해가 되는 바 입니다.

단, 이 고성능의 Redis를 사용하면서 사용자는 Redis가 `Single-thread`로 동작한다는것을 잊으면 안됩니다.

Redis는 single thread로 동작하기 때문에 `long-time` 명령 수행시 다른 명령어들을 처리 할 수 없는 상태가 되기 때문에 매우 비효율적으로 동작하게 됩니다. 따라서 사용자들은 이 점을 명시하고 될 수 있는 한 `long-time` 명령의 수행들을 피해야 합니다.

위에서 Redis에서 주요 명령어들이 `O(1)`의 성능을 보인다면서 왜 `long-time` 명령을 수행하지 말라고 하는건지 의문을 가지실수 도 있는데, 일반적으로 오랜 시간이 걸리는 명령은 데이터가 매우 많은 상황에서, 여러개의 키를 다루는 명령어에서 `O(N)`의 수행시간을 가지면서 성능에 악영향을 끼치게됩니다.

```
keys : Redis에 존재하는 모든 key를 조회함으로써 long-time 수행을 일으킴 (Redis 매뉴얼에도 운영환경에서는 사용하지 말것을 권고함)

smem : 'Set' 자료구조의 모든 member들을 조회함으로써 long-time 수행을 일으킴

flushall : 전체 데이터를 지우는 명령어로, 키 갯수에 비례한 수행시간을 갖음
```

위의 명령어들은 아래와 같이 수정하여 효율적으로 사용 할 수 있습니다.

```
keys --> scan을 통해 순회 탐색으로 전체 key 조회
smem --> sscan을 통한 순회 탐색으로 전체 member 조회
```
--- 

### Redis Persistent

Redis는 휘발성 Memory에 데이터를 저장하기때문에 Persistent를 지원하기 위해 `RDB` 와 `AOF`를 지원합니다. 

`RDB`는 이름으로 인해 착각 할 수 도 있지만 `DB`와는 관련없는, 단순히 Redis memory를 snapshot한 데이터라고 생각 하시면 됩니다. 여기서 반드시 알아두셔야 할 점은 위 섹션에서 알아보았듯이 Redis는 `Single thread`로 동작하기 때문에 Snapshot을 뜨는 (`SAVE`) 시점에서는 모든 명령어 수행이 제한되게 됩니다.

위와같이 `SAVE`가 진행될때 모든 명령 수행이 제한된다면 사용상에 문제가 생기기 때문에 백그라운드에서 스냅샷을 뜨는 (`BGSAVE`)를 지원합니다. 

`BGSAVE`의 경우 fork 를 통해 자식 프로세스를 하나 생성하여 메인 프로세스와 분리된 별도의 프로세스에서 동작하기 때문에 사용자들의 명령실행은 문제없이 수행 할 수 있습니다. 

하지만 이때 하나의 프로세스가 더 생성되어 데이터를 snapshot을 수행하면서 기존 사용 메모리만큼의 메모리가 추가로 할당이 필요 할 수도 있습니다. 따라서 RDB기능을 사용할때에는 메모리가 현재 사용량의 2배가 될 수 있음을 인지하고 여유 메모리 공간을 유지해야합니다.

만약 `RDB` 저장이 실패시 Write 명령을 처리하지 않도록 block 되니 이점을 주의 하셔야 합니다. (read 명령은 수행 가능하여 HeartBeat check는 통과되어 모니터링상에는 문제 없을 수도 있음 주의)

`AOF`는 기존 DBMS에서 제공하는 Write Log와 비슷한 기능으로 Redis에 데이터를 저장하기 전에 수행되는 명령어들을 별도로 저장하여 해당 명령어로 persistent를 유지 할 수 있도록 해줍니다.

레디스에서는 snapshot 이후의 시점의 명령어들을 기록하게 되는데, 즉 위에서 말씀드렸던 `RDB`와 `AOF` 혼합하여 전체적인 persistent를 유지 할 수 있는것입니다.

---

### Redis cluster

어느정도 규모있는 프로젝트에서 Redis를 사용하고 계시다면, 당연히도 Redis Cluster를 구성하여 사용하고 계실것으로 생각됩니다. Redis는 분산처리하여 다음과같은 이점을 얻을 수 있습니다.

- 데이터를 여러 노드로 분산 시킬수 있습니다. (사용 메모리 분산)
- 장애 발생시 다른 노드로 대체운영이 가능합니다. (복제와 함께 사용시)

기본적으로 Redis cluster에 구성되어있는 node들은 `hash slot`이 16384개(0 ~ 16383)가 할당되어 있어 Redis 데이터 `Key`를 해싱하여 알맞는 `hash slot`을 찾아 클러스터의 노드들에 분산하여 저장시키게 됩니다.

노드들에 할당되어있는 hash slot은 서비스의 중단없이 쉽게 이동 시킬수도 있습니다.

---

### Redis Replication

Redis는 Replication을 구성하여 Fail over에 대한 HA를 제공 할 수 있습니다.  
이른바 `Master - Slave`의 Replication을 구성 할 수 있으며, 하나의 Master에 여러개의 Slave를 구성 할 수도 있습니다. 

Master -> Slave의 Consistent 를 유지하기위해 위`Redis Persistent`섹션에서 Persistent를 유지하기 위해 사용되었던 `RDB`와 `AOF`가 사용됩니다.   

그런데 여기서 주의해야 할 점이 있습니다. 만약 메모리의 사용량 증가를 막기 위해 `BGSSAVE` 옵션을 off 시키는 경우에 Replication을 사용 하고 있다면, 사용자의 설정 내용과 관계없이 `BGSAVE`가 일어나기 때문에 메모리 사용량이 증가 할 수 있음을 주의하셔야 합니다.

---

### Redis Sentinel

`Redis Replication`을 통해 Redis는 장애복구를 할 수 있는 시스템을 갖출수 있습니다. 그렇다면 Redis는 `장애`가 발생한 상황을 어떻게 알 수 있을까요?  
이때 사용되어지는것이 바로 `Redis Sentinel`입니다. `Sentinel`은 `master`와 `slave`들을 감시하다가 `master`에서 장애발생을 감지하면 `slave`를 `master`로 승격시키는 작업을 진행합니다.

`Sentinel`에 의해서 노드들은 master/slave 역할이 계속해서 변경 될 수 있습니다. 이러한 특성으로 인해 `Sentinel`은 노드의 역할이 변경되었을때 client에게 pub/sub를 통해 변경된 사항을 알려 master node로 접근 하도록 하는 역할도 진행합니다. (Slave node는 기본적으로 read-only로 사용되어 write 명령이 실행 불가함)

`Sentinel`은 `ping`을 통해 노드들의 장애를 판단하며 한번의 실패로 바로 장애로 판단하는것이 아닌, 여러대의 `Sentinel`에 의해 `정족수(의결을 위한 최소 구성원의 수)`를 넘게 장애로 판단되는 노드를 실제 장애 노드로 판단하게 됩니다. (주관적 DOWN -> 객관적 DOWN)

위의 과정을 통해 `master`노드가 장애로 판단되면 가용한 `slave`노드 들 중 `slave_priority`값이 가장 높은 슬레이브가 우선 선택되어 `master`역할로써 사용되게 됩니다. 이러한점을 이용하여 `slave_priority`를 `0`으로 설정하여 필요시 특정 노드가 `master`로 승격되지 않도록 설정 할 수도 있습니다.