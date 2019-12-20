---
layout: post
comments: true
title: mysql count(*) vs count(column)
tags: [db, mysql]
---

### 개요

SQL 에서 쿼리의 실행 결과 로우의 수를 조회하고자 할때 count()함수를 사용하게 됩니다.  
이때, count(*)을 쓰는 경우와 count(column-name)을 사용하는 경우가 어떻게 다른지 확인해보도록 하겠습니다.  
  
### count(*) vs count(name)  

예시 테이블)  

|id|name|
|:-:|:-:|
|1|김하나|
|2|이둘|
|3|null|

위와같은 result data가 있다고할때,  
  
> count(*) = 3  
> 
> count(key) = 3  
> 
> count(name) = 2  
  
다음과 같은 결과가 나오게 되는데, 이 결과로 보면 count(column-name)은 null값을 제외한 데이터들의 수를 확인 한다는 차이가 있습니다.   
  
그렇다면 성능상 이슈는 어떨까요?  
직접, explain을 통해 count(*) 과 count(column-name)을 수행해보면 수행 로직은 같습니다.  
그러나 위에서 알아본것처럼 count(*)은 단순히 결과row의 count를 반환하는 반면에  count(column-name)의 경우 칼럼 데이터에 접근해 null인지 확인후 count를 진행하게됩니다.  
따라서 null값 존재 유무와는 관계없이 단순 결과데이터의 수를 조회하는 경우라면 count(*)을 사용하는것이 성능적으로 좋습니다.

