---
layout: post
comments: true
title: Mysql hint
tags: [db, mysql]
---

(이 내용은 mysql 5.7 version 기준으로 작성되었습니다.)  

---

### Optimizer Hint  

쿼리실행 최적화를 위해 선작업으로 징행되는 옵티마이저에게 specific한 쿼리 플랜을 정해주기 위하여 설정을 해 줄수 있습니다. 이때 사용되는것이 Optimizer hint이며, 쿼리 구문별로 정의하는 hint를 통해서 Optimizer를 control 할수 있습니다.   

--- 

#### Hint 종류   

|Hint Name|Description|Applicable Scope|
|--|---|--|
|BKA, NO_BKA|Affects Batched Key Access join processing|Query block, table|
|BNL, NO_BNL|Affects Block Nested-Loop join processing|Query block, table|
|MAX_EXECUTION_TIME|Limits statement execution time|Global|
|MRR, NO_MRR|Affects Multi-Range Read optimization|Table, index|
|NO_ICP|Affects Index Condition Pushdown optimization|Table, index|
|NO_RANGE_OPTIMIZATION|Affects range optimization|Table, index|
|QB_NAME|Assigns name to query block|Query block|
|SEMIJOIN, NO_SEMIJOIN|Affects semijoin strategies|Query block|
|SUBQUERY|Affects materialization, IN-to-EXISTS subquery stratgies|Query block|  


**BKA (Batched Key Access join)**   
index + join buffer 를 이용하는 조인 알고리즘   
(https://dev.mysql.com/doc/refman/5.7/en/bnl-bka-optimization.html)  

**BNL (Block Nested-Loop join)**  

;Nested loop join

**MRR (Multi Range Read)**  

Nested loop join 다량의 random IO가 발생하게 되는데, 이런한 Random IO를 내부적으로 Buffer를 두어 Sequence한 IO로 바꾸어주는 기능입니다.   
(https://m.blog.naver.com/PostView.nhn?blogId=seuis398&logNo=70159183472&proxyReferer=https%3A%2F%2Fwww.google.com%2F)

![1]({{ site.images | relative_url }}/posts/2019-11-22-mysql-hint/1.png)      

**NO_ICP (Index Condition Pushdown)**  

인덱스를 사용해 행검색시 최적화를 해주는 기능으로, ICP가 설정되어 있다면 인덱스만으로 where 조건절의 일부분을 평가 할 수 있습니다.    

**NO_RANGE_OPTIMIZATION**  

index range access를 사용하지 않도록 하는 기능입니다.  

**QB_NAME (Query Block Naming)**  

힌트가 적용되는 쿼리 block에 이름을 부여하여 힌트의 적용범위를 명확하게 정의할수 있습니다.  

**SEMIJOIN**  

서브쿼리내에 존재한 데이터를 통해 메인쿼리에서 데이터 조건과 매칭하는 기능으로, where .. in (..) where .. exists (..) 형태에서 사용됩니다.  

**SUBQUERY**  

IN 절 내의 서브쿼리를 임시테이블로 만들어 조인하는 Materialization에 적용됩니다. Materialization이 적용되지 않을경우 , 매 레코드마다 서브쿼리가 실행되어 매우 비효율적입니다.  

---

#### Syntax  

```c
/*+ ... */
```
위와같은 신택스 구문내에 hint를 기술하여 사용합니다.

---

#### Example  

```sql
SELECT /*+ NO_RANGE_OPTIMIZATION(t3 PRIMARY, f2_idx) */ f1
    FROM t3 WHERE f1 > 30 AND f1 < 33;
SELECT /*+ BKA(t1) NO_BKA(t2) */ * FROM t1 INNER JOIN t2 WHERE ...;
SELECT /*+ NO_ICP(t1, t2) */ * FROM t1 INNER JOIN t2 WHERE ...;
SELECT /*+ SEMIJOIN(FIRSTMATCH, LOOSESCAN) */ * FROM t1 ...;
EXPLAIN SELECT /*+ NO_ICP(t1) */ * FROM t1 WHERE ...;
```

---

### Index Hint   

옵티마이저가 Index를 선택할때, 어떤 Index를 선택하여 쿼리를 실행할지에대한 선언을 해 줄 수 있습니다.

---

#### Hint 종류  

|Hint Name|Description|
|--|---|
|USE INDEX ()|사용 인덱스 정의|
|USE INDEX (... , ... , ...)|인덱스 리스트를 정의하여 옵티마이저가 선택|
|IGNORE INDEX()|사용하지 않을 인덱스 정의|
|... INDEX FOR JOIN ()|Join시 사용할 인덱스 정의|
|... INDEX FOR ORDER BY ()|Order By시 사용할 인덱스 정의|
|... INDEX FOR GROUP BY ()|Group By시 사용할 인덱스 정의|

---

#### syntax   

Index hint 는 select 구문에서만 사용 되며(UPDATE, DELETE ... X) from table 이후에 index를 설정해줄수 있습니다.   

```sql
tbl_name [[AS] alias] [index_hint_list]
```

---

#### Sample  
 
```sql
SELECT *
FROM table1 USE INDEX (col1_index,col2_index)
WHERE col1=1 AND col2=2 AND col3=3;
 
SELECT *
FROM table1 IGNORE INDEX (col3_index)
WHERE col1=1 AND col2=2 AND col3=3;
```

---


### 참고 URL  

https://dev.mysql.com/doc/refman/5.7/en/optimizer-hints.html  

https://dev.mysql.com/doc/refman/5.7/en/index-hints.html