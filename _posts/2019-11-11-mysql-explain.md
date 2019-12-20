---
layout: post
comments: true
title: MySql Explain (실행계획)
tags: [db, mysql]
---

### MySQL Explain

![1]({{ site.images | relative_url }}/posts/2019-11-11-mysql-explain/1.png)     

Explain을 실행했을시 다음과같은 화면이 보여집니다. 각 칼럼이 의미하는 내용들을 확인해보도록 하겠습니다.  

---

#### id  

- 쿼리에 포함된 각 select 문에 대한 순차 식별자  
- select 항목을 구분하는 번호  
- 동일한  ID의 경우 위에서 아래로 진행됩니다.  
- table 항목이 "derived2"인경우 id:2 를 수행후 진행됩니다.  

---

#### select_type  

- SIMPLE : union, sub query 가 없는 단순 select  
- PRIMARY : subquery 가 있을때 가장 바깥쪽 select (복수행 실행 계획에서 1행만 존재한다)  
- DERIVED : from 절 내부 서브쿼리  
- SUBQUERY : subquery  
- DEPENEDNETSUBQUERY : 외부 쿼리와 상호 연관이 있을때  
- UNCACHEABLESUBQUERY : 사용자 변수, NOT DETERMINISTC 속성의 stored routin, UUID(), RAND() 와같이 호출할때마다 변한느 함수사용시.  
- UNION : union 이후 select   
- UNIONRESULT : UNION문으로 생성된 임시 테이블  
- DEPENEDENTUNION : 바깥 쿼리에 의존성을 가진 UNION  
- UNCACHEABLEUNION : UNION에 포함된 요소에 의하여 CACHE 불가능  
- MATERIALIZED : IN 절 내부 서브쿼리, semi-join 최적화를 위해 제공  

---

#### table

각 실행 row가 어떤  테이블을 이용하는지 나타내는 칼럼입니다.  

---

#### partitions

파티셔닝된 테이블의 경우 해당 쿼리가 사용한 파티션 목록을 표시하며 파티셔닝이 되어있지 않은 테이블의 경우 null을 표시합니다.  

---

#### type  

Optimizer가 join, select시에 어떤 방법으로 row를 조회하는지 나타내는 칼럼입니다. (상위에 위치한 type일수록 빠름)  

- system : row가 1건 미만인 테이블 조회시 (시스템테이블)  
- const : primary key, unique key를 상수로 조회하여 조회결과가 1건인 경우  
- eq_Ref : inner table의 primary key 혹은 unique key로 조회  
- ref : non unique key 를 이용한 동등비교  
- ref_or_null : ref와 동일하나, null 값이 추가되어 비교되는 경우  
- index_merge : 복수의 index 병합되어 검색되어지는 경우 (mysql에서 성능이 좋지 않음.)   
- fulltext : MATCH_AGAINST절을 이용한 full text 검색한 경우  
- unique_subquery : 서브쿼리가 고유한 값만 리턴하는 경우  
- index_subquery : 서브쿼리가 고유한 값만 리턴하지 않고(중복값 리턴), In sub query 내 중복값 제거하는 경우  
- range : index 범위 비교  
- index : index 풀스캔  
- all : table 풀스캔  

---

#### possible_keys

optimizer 가 고려한 모든 index 목록을 보여주는 칼럼입니다.  

---

#### key

optimizer가 데이터 조회에 실제 사용할 index를 보여주는 칼럼입니다.  

---

#### key_len

데이터 조회에 실제 사용할 index 길이를 나타냅니다.  

---

#### ref  

데이터 조회에 사용되는 key 항목의 index와 비교하는 컬럼입니다.  

---

#### rows

쿼리 실행시 검색해야 하는 row 수 (optimizer 추정치) : 실제데이터와 상당히 다름

---

#### extra

optimizer가 해석한 쿼리의 추가정보 표시하는 칼럼입니다.  

- const row not found : 대상 테이블에 데이터 없음  
- distinct : 중복제거 (오버헤드, 수정고려)  
- full scan on null key : In절에 서브쿼리가 있을때 나타날수있음, outer query에서 null값이 전달될때 풀스캔 일어남 (반드시 수정필요)  
- Impossible having : 문법적으로 having 절이 항상 false인 경우 (수정 필요)  
- Impossible where : 문법적으로 wherer 절이 항상 false인 경우 (수정 필요)  
- Impossible Where notices after reading const table : 한 row를 실행해보고나서 impossible 한 경우 (수정 필요)  
- No matcching min/max row : aggregate 함수가 있는 쿼리의 조건절에 일치하는 row가 한건도 없는 경우  
- No matching row in const table : join에 사용된 테이블에서 const방식으로 접근할때, 일치 row 없는 경우  
- No table used : from절에 table이 없거나 from dual 구문사용, 물리적 테이블 없음  
- Not exists : left join 형태의 anti join 쿼리를 Not Exists 형태로 최적화 하느 ㄴ경우, 조건에 맞는 결과를 찾으면 추가 Join 처리 하지 않음.  
- Range checked for each record ( index map : N ) : Join 처리시 적절한 index가 없을때, 선행 테이블에서 공급되는 값에 따라 Index 사용ㅇ을 검토할수 있는 경우.  
- Scanned N databases : inofrmation_schema 조회  
- select tables optimized away : 쿼리가 aggregate 함수만 포함하고 있을때, Optimizer가 Index lookup후 1개의 결과만 리턴  
- Unique row not found : 두 테이블이 각각 UK, Outer join 실행시 outer 테이블에 일치하는 row 가 존재하지 안흔 경우  
- Usign filesort : 데이터 정렬렬을 ㄹ위하여 ㅁ루리적인 정렬자겅ㅂ 수행 필요  
- Using index : 실제 데이터 block조회가 아닌 Index만으로 결과 생성할수 있는 경우 (covered index 상태)  
- Using index for group by : index block 만으로 groupby/distinct 처리가 가능한 경우  
- Using join buffer : join 처리에 index가 없을때 join buffer가 사용됨.  
- Using sort_union(...), Using union(...), Using intersect(...) : 두개이상의 index index_merge  
- Using temporary : 쿼리 처리를 위해 임ㅁ시 테이블 사용됨/ group by에 인덱스 안탈때 / group by, order by 칼럼이 각각 컬럼을 사용할때  
- Using where : where절 다음 join에 사용될 행이나 클라이언트에게 돌려질 행을 제한하는 경우 (10개만 제한함)  

