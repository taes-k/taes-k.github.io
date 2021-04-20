---
layout: post
comments: true
title: Elasticsearch 검색 성능 향상 튜닝
tags: [elasticsearch]
---

(해당 포스팅은 Elasticsearch doc 내용을 발췌한 내용입니다. [원문보기](https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-search-speed.html))


---

### Filesystem 캐시를 위한 메모리 늘리기

- hot index 영역을 메모리에 유지 할 수 있도록 합니다.
- 적어도 50% 이상의 메모리를 Filesystem 캐시에 할당 되도록 합니다.

---

### 빠른 장비 사용하기

- I/O bound가 필요한 검색의 경우 memory 추가, SSD 사용을 통해 검색 성능을 향상 시킬 수 있습니다.
- CPU bound가 필요한 검색의 경우 고성능의 CPU 사용을 통해 검색 성능을 향상 시킬 수 있습니다.

---

### Document 모델링 하기

- Join을 피할 수 있도록 Document를 모델링 해야합니다.
- 부모-자식 관계의 쿼리는 검색겨로가를 수백배 느려지게 만들 수 있습니다.
- document를 비정규화하여 join 없이 수행 될 수 있도록 합니다.


---

### 가능한 적은수의 field를 조회하기

- query string, multi match 쿼리 대상 field가 많을수록 성능은 떨어집니다.
- 인덱스시 단일필드로 인덱싱 하고 검색때 필드를 사용하는 방법으로 개선할수도 있습니다.

```json
PUT movies
{
  "mappings": {
    "properties": {
      "name_and_plot": {
        "type": "text"
      },
      "name": {
        "type": "text",
        "copy_to": "name_and_plot"
      },
      "plot": {
        "type": "text",
        "copy_to": "name_and_plot"
      }
    }
  }
}
```

---

### 사전 indexing

- 쿼리에서 필요한 데이터를 사전에 미리 indexing 해서 검색시간을 줄일 수 있습니다.

```json
PUT index/_doc/1
{
  "designation": "spoon",
  "price": 13
}
```
```json
GET index/_search
{
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 10 },
          { "from": 10, "to": 100 },
          { "from": 100 }
        ]
      }
    }
  }
}
```
```json
PUT index
{
  "mappings": {
    "properties": {
      "price_range": {
        "type": "keyword"
      }
    }
  }
}

PUT index/_doc/1
{
  "designation": "spoon",
  "price": 13,
  "price_range": "10-100"
}
```
```json
GET index/_search
{
  "aggs": {
    "price_ranges": {
      "terms": {
        "field": "price_range"
      }
    }
  }
}
```


---

### keyword mapping 사용하기

- 범위 쿼리를 사용하지 않고, 빠른검색이 중요한 경우 '숫자' 보다 키워드르 매핑하는것이 유리 할 수 있습니다.



---

### script 사용 피하기

- 가능하다면 sciprt 기반의 정렬, 집계, score 쿼리를 피하는것이 좋습니다.


---

### 반올림된 날짜 사용하기

- date field에 `now` 를 사용시 일반적으로 캐시를 사용할수 없습니다.
- 반올림 날짜 사용시 캐시를 더 잘 탈수 있습니다.

```json
GET index/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "range": {
          "my_date": {
            "gte": "now-1h/m",
            "lte": "now/m"
          }
        }
      }
    }
  }
}
```

---

### 읽기전용 인덱스

- 읽기전용 인덱스는 단일 segment로 병합한다면 좀더 효율적인 데이터 구조로써 검색을 수행할수 있습니다.
- 로그성 데이터와 같이 일자별로 인덱스가 생성되는경우 사용하기 좋습니다.

---

### 전역 서수(global ordinal) 설정하기

- 집계 성능 최적화를 위해 사용되는 데이터 구조로서, 집계에 많이 사용되는 필드를 캐시하도록 설정 할 수 있습니다.
- 힙 사용량이 증가하고 새로고침에 더 오래걸릴 수 있으니 설정에 주의해야 합니다.

```json
PUT index
{
  "mappings": {
    "properties": {
      "foo": {
        "type": "keyword",
        "eager_global_ordinals": true
      }
    }
  }
}
```

---

### filesystem 캐시 warm up 하기

- ES 머신 재 시작시 캐시가 비어있기때문에 hot 영역 인덱스를 메모리에 로드할때까지 시간이 걸립니다.
- `index.store.preload` 설정을 통해 명시적으로 지정 해 줄 수 있습니다.


---

### 색인 정렬 사용하기

- 색인 정렬은 접속사 `(a AND b AND ...)`를 효율적으로 구성하는 유용 할 수 있습니다.
- 낮은 cadinality field 에 대해서만 유용합니다.
- cadninality가 낮고, 필터링에 자주 사용되는 필드에 대해서 사용하는것을 고려하는것이 좋습니다.


---

### 캐시 활용도 최적화하기

- filesystem 캐시, request 캐시, 쿼리 캐시 등이 존재합니다.
- 위 캐시들은 노드수준에서 유지되는데, 요청이 다른 노드에 접근하면 캐시가 무용지물이 될 수 있습니다.
- 동일 사용자가 유사항 검색 요청을 하는경우가 많기때문에 request 노드 분배시 사용자 혹은 세션을 구분해 로드밸런싱을 해주도록 하는것이 좋습니다.


---

### Replication은 처리량에 도움을 주지만 검색성능을 향상시키지는 않습니다.

- 일반적으로 노드당 샤드수가 적을때 검색 성능이 좋습니다.
- filesystem 캐시를 각 샤드에 더 많이 제공하기 때문에 성능이 좋습니다.
- 복제본의 수는 성능과 상충관계가 있기때문에 잘 고려해야 합니다.
- 권장하는 replica number = `max (max_failures, ceil (num_nodes / num_primaries)-1)`





