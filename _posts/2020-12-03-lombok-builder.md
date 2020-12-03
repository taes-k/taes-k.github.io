---
layout: post
comments: true
title: Lombok Builder 사용시 유의할점
tags: [java, lombok, builder]
---

### Lombok Builder 사용시 유의할점

개발을 매우 편리하게 도와주는 Lombok의 `@Builder`를 자주 사용하실텐데요, 사용상 주의하셔야 할 점이 있습니다.

```java
Obj obj = new Obj();
obj.setId(1);
obj.setName("하나");
```

```java
Obj obj = Obj.builder
    .id(1)
    .name("하나")
    .build();
```

특정상황에서 위 선언한 두개의 객체가 다른 값들을 가질수 있다는점을 유의하셔야합니다.

멤버변수에 초기값을 지정했을때 차이가 발생하는데 아래 예시를 통해 알아보겠습니다.

```java
@Getter
@Setter
@NoArgsConstructor
public class Obj
{
    private int id;
    private String name;
    private String attr1="x";
    private String attr2="x";
    private String attr3="x";
 
 
    @Builder
    public obj (int id, String name, String attr1, String attr2, String attr3)
    {
        this.id = id;
        this.name = name;
        this.attr1 = attr1;
        this.attr2 = attr2;
        this.attr3 = attr3;
    }
}
```

```java
public class Test
{
    @Test
    public void newObject()
    {  
        Obj obj = new Obj();
        obj.setId(1);
        obj.setName("하나");
         
        Assertion.assertEqual("x", obj.getAttr1); // => 성공
    }
     
    @Test
    public void builderObject()
    {
        Obj obj = Obj.builder
            .id(1)
            .name("하나")
            .build();
        Assertion.assertNotEqual("x", obj.getAttr1); // => 성공
        Assertion.assertNull(obj.getAttr1); // => 성공
    }
}
```

위의 유닛테스트와 같이 Builder로 객체 생성시 지정하지 않은 필드에 대해서는 필드 지정 초기 값이 반영되지 않고 null로 설정됩니다.

즉.
```java
Obj obj = Obj.builder
            .id(1)
            .name("하나")
            .build();
```
위 코드는 아래와 같이 사용되는 것과 같음에 유의 하셔야합니다.
```java
Obj obj = Obj.builder
            .id(1)
            .name("하나")
            .attr1(null)
            .attr2(null)
            .attr3(null)           
            .build();
```

---

### 추가내용
Lombok 에서 생성해주는 `@Builder`를 디컴파일 해보면 아래와 같은 코드를 확인 할 수 있습니다.

```java
public class Obj {
    private int id;
    private String name;
    private String attr1 = "x";
    private String attr2 = "x";
    private String attr3 = "x";
 
    public Obj(int id, String name, String attr1, String attr2, String attr3) {
        this.id = id;
        this.name = name;
        this.attr1 = attr1;
        this.attr2 = attr2;
        this.attr3 = attr3;
    }
 
    public static Obj.ObjBuilder builder() {
        return new Obj.ObjBuilder();
    }
 
    public int getId() {
        return this.id;
    }
 
    public String getName() {
        return this.name;
    }
 
    public String getAttr1() {
        return this.attr1;
    }
 
    public String getAttr2() {
        return this.attr2;
    }
 
    public String getAttr3() {
        return this.attr3;
    }
 
    public void setId(final int id) {
        this.id = id;
    }
 
    public void setName(final String name) {
        this.name = name;
    }
 
    public void setAttr1(final String attr1) {
        this.attr1 = attr1;
    }
 
    public void setAttr2(final String attr2) {
        this.attr2 = attr2;
    }
 
    public void setAttr3(final String attr3) {
        this.attr3 = attr3;
    }
 
    public Obj() {
    }
 
    public static class ObjBuilder {
        private int id;
        private String name;
        private String attr1; // 초기값이 미설정되어있음
        private String attr2; // 초기값이 미설정되어있음
        private String attr3; // 초기값이 미설정되어있음
 
        ObjBuilder() {
        }
 
        public Obj.ObjBuilder id(final int id) {
            this.id = id;
            return this;
        }
 
        public Obj.ObjBuilder name(final String name) {
            this.name = name;
            return this;
        }
 
        public Obj.ObjBuilder attr1(final String attr1) {
            this.attr1 = attr1;
            return this;
        }
 
        public Obj.ObjBuilder attr2(final String attr2) {
            this.attr2 = attr2;
            return this;
        }
 
        public Obj.ObjBuilder attr3(final String attr3) {
            this.attr3 = attr3;
            return this;
        }
 
        public Obj build() {
            return new TestObj2(this.id, this.name, this.attr1, this.attr2, this.attr3);
        }
    }
}
```