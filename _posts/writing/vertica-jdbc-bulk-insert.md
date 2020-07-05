---
layout: post
comments: true
title: [Vertica] jdbc bulk insert
tags: [vertica, jdbc]
---

### vertica bulk insert

일반적으로 사용되는 OLTP(OnLine Transaction Processing)성 DB와는 다르게, Vertica는 OLAP(OnLine Analytical Processing)성 DB로, 대량데이터 처리에 초점이 맞추어져 있습니다. 따라서 서비스를 설계 할 때에도 이러한 DB의 특성에 맞추어 설계를 해야하는데, 예를 들어 MySQL에서 다량의 데이터를 insert를 한다면 다음과같은 쿼리를 사용하게 될것입니다.  
 
```sql
INSERT INTO SAMPLE_TABLE(?,?,?)
VALUES
    (val1, val2, val3)
    , (val1, val2, val3)
    , (val1, val2, val3)
    , ...
```

Vertica에서도 위와같은 문법으로 insert 할 수 있습니다.

```sql
INSERT INTO SAMPLE_TABLE(?,?,?)
SELECT val1, val2, val3
UNION SELECT val1, val2, val3
UNION SELECT (val1, val2, val3)
UNION SELECT ...
```

하지만 위와같은 쿼리를 통해 데이터를 넣을경우 query size 제한으로 인해 대량의 데이터 insert에 실패할 가능성이 있습니다. 하지만 앞서 말했듯이 Vertica에서는 OLAP성 DB로, 대량 데이터 처리에 강점을 가지기 때문에 대량 데이터 적재를 위한 별도의 방법이 존재합니다.

아래에서 jdbc 환경에서 Bulk-loading을 수행하기위한 방법들을 소개합니다.
(해당 내용은 Vertica 공식 docs를 참조합니다. [링크](https://www.vertica.com/docs/9.2.x/HTML/Content/Authoring/ConnectingToVertica/ClientJDBC/LoadingDataThroughJDBC.htm?tocpath=Connecting%20to%20Vertica%7CClient%20Libraries%7CProgramming%20JDBC%20Client%20Applications%7CLoading%20Data%20Through%20JDBC%7C_____0))

---
### COPY 

Vertica에서는 Bulk-loading을 위해서 `COPY` 구문을 지원합니다. 

```sql
COPY customers FROM '/data/customers.txt' DELIMITER '|' NULL '' DIRECT ENFORCELENGTH
```

위와같은 구문을 통해서 별다른 처리없이 File 데이터를 bulk-load 할 수 있습니다.  
아래 소개해드릴 JDBC를 활용한 방법들도 Vertica 내부적으로는 COPY구문으로 변환하여 데이터를 일괄처리하게 됩니다.  

---
### JDBC PreparedmentState

```java
    Properties myProp = new Properties();
    myProp.put("user", "ExampleUser");
    myProp.put("password", "password123");
    
    try(connection conn = DriverManager.getConnection(url,myProp);
    PreparedStatement pstmt = conn.prepareStatement(
                            "INSERT INTO customers (CustID, Last_Name, " + 
                            "First_Name, Email, Phone_Number)" +
                            " VALUES(?,?,?,?,?)");)
    {
        String[] firstNames = new String[] { "Anna", "Bill", "Cindy","Don", "Eric" };
        String[] lastNames = new String[] { "Allen", "Brown", "Chu","Dodd", "Estavez" };
        String[] emails = new String[] { "aang@example.com","b.brown@example.com", "cindy@example.com","d.d@example.com", "e.estavez@example.com" };
        String[] phoneNumbers = new String[] { "123-456-7890","555-444-3333", "555-867-5309","555-555-1212", "781-555-0000" };

        for (int i = 0; i < firstNames.length; i++) {
            pstmt.setInt(1, i + 1);
            pstmt.setString(2, lastNames[i]);
            pstmt.setString(3, firstNames[i]);
            pstmt.setString(4, emails[i]);
            pstmt.setString(5, phoneNumbers[i]);
            pstmt.addBatch();
        }
        
        try {
            pstmt.executeBatch();
        } catch (SQLException e) {
            System.out.println("Error message: " + e.getMessage());
            return;
        }
        conn.commit();
    } catch (SQLException e) {
        e.printStackTrace();
    }
```

JDBC preparedmentState `executeBatch`를 통해 insert를 하게되면 내부적으로 `COPY` 구문으로 변경해 bulk-loading을 하게 됩니다.  
Java에서는 Insert query를 실행했지만 실제로 Vertica에서 실행쿼리 로그를 확인해보면 정확히 확인 할 수 있습니다.   

```sql
SELECT *
FROM v_monitor.query_requests
```

![1]({{ site.images | relative_url }}/posts/222/1.png)  

---

### File Stream

메모리상의 데이터를 올리는것이 아닌, File stream을 읽어 처리도 가능합니다.  


---

### JdbcTemplate