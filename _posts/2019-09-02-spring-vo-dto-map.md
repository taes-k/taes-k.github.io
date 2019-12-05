---
layout: post
comments: true
title: VO, DTO, Entity vs Map
tags: [spring]
---

Spring을 사용하면서 많은분들이 헷갈려 하고 혼용되어 사용되기도 하는 용어인 VO, DTO, Entity에대해서 정리하고 이 Object들을 사용하는것과 Map을 사용하는것에대한 논쟁에대해서 알아보도록 하겠습니다.  

### DTO(Data Transfer Object)

약자 그대로 해석을 하면 데이터를 개체로 변환한 것으로,데이터 교환을 위해 선언한 자바빈즈 규약을 맞추어 작성한 클래스이며  
여기서 말하는 '계층'이란 view - controller - service - repository 각 레이어들을 말합니다.

```
DTO a = new DTO(1);
DTO b = new DTO(1);
> a != b
```

### VO(Value Object)

말 그대로 '값'을 위해 사용하는 오브젝트로써, 값표현을 위한 불변의 클래스입니다.  
ReadOnlly를 통해 불변의 특징을 가지며 따라서 VO의 경우 생성자를 통해 지정된 값은 변경될수 없도록 setter가 없으며, getter를 통해 값 조회만 가능하도록 만들어 주어야 합니다.
  
```
VO a = new VO(1);
VO b = new VO(1);
> a==b
```
  
```
Color.java
class Color {
  
    private int R,  G,B,A;
 
    public Color(int r, int g, int b, int a){…}
  
    public final static Color RED = new Color(255,0,0);
    public final static Color GREEN = new Color(0,255,0,0);
  
}
```  


### Entity

실제 DB 테이블과 매핑된 오브젝트.  
DTO 와 Entity를 분리하는 이유는, view layer와 db layer의 역할을 분리하기 위해서 사용됩니다.  
view layer와 통신하는 DTO의 경우 개발도중에 잦은 변화가 일어날수 있는 반면에 db layer는 변경되지 않아야하기때문에 구분해두는 이유도 있습니다.  

-----

### DTO를 찬성
- 데이터를 정의하는 객체
- 데이터의 캡슐화
- 타입 체크
- 유효성 체크
- 변수명 규격화

### DTO를 반대

- 실질적인 유효성 체크를 위해서는 결국 별도의 로직이 필요
- 여러 계층들 사이에 공통된 VO를 사용하면서 결합도가 높아질수 있음
- 중복코드 클래스가 계속해서 생성됨


### VO vs Map 논쟁에대한 참조 자료

- http://www.okjsp.pe.kr:8080/article/275461?note=937130
- https://zgundam.tistory.com/65
- http://blog.naver.com/tmondev/220344112936
- https://itmemo.tistory.com/105
- http://www.okjsp.pe.kr:8080/article/275461?note=937130

--- 
### Stackoverflow에서 관련논쟁에대해 가장 많은 투표를 받은 포스트

https://stackoverflow.com/questions/21554977/should-services-always-return-dtos-or-can-they-also-return-domain-models

해당내용은 엄밀히 말하자면, domain model + DTO vs domain model에 대한 논쟁입니다.

여기서 domain model은 DB와 대응하는  entity를 의미하는것 같습니다.

> When to Use  
>   
> For large projects.   
> Project lifetime is 10 years and above.  
> Strategic, mission critical application.   
> Large teams (more than 5)  
> Developers are distributed geographically.   
> The domain and presentation are different.   
> Reduce overhead data exchanges (the original purpose of DTO)  
> When not to Use  
>   
> Small to mid size project (5 members max)  
> Project lifetime is 2 years or so.   
> No separate team for GUI, backend, etc.  
> Arguments Against DTO  
>   
> Duplication of code.   
> Cost of development time, debugging. (use DTO generation tools http://entitiestodtos.codeplex.com/)  
> You must synchronize both models all the time.   
> Cost of developement: Additional mapping is necessary. (use auto mappers like https://github.com/AutoMapper/AutoMapper)  
> Why are Data Transfer Objects an anti-pattern?  
> Arguments With DTO  
>   
> Without DTO, the presentation and the domain is tightly coupled. (This is ok for small projects.)  
> Interface/API stability  
> May provide optimization for the presentation layer by returning a DTO containing only those attributes that are absolutely required. Using linq-projection, you don't   have to pull an entire entity.  
> To reduce development cost, use code-generating tools  
