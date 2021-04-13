---
layout: post
comments: true
title: 어떻게 리팩토링을 해야할까 - 상속
navdisplay: false
tags: [refactoring]
---


### How do I refactorig? - 상속

**12.1 메서드 올리기**

![1]({{ site.images | relative_url }}/pages/refactoring/10-how-refactoring-inheritance/1.jpeg) 

```js
class Employee {...}

class Salesperson extends Employee {
    get name() {...}
}

class Engineer extends Employee {
    get name() {...}
}
```

```js
class Employee {
    get name(){...}
}

class Salesperson extends Employee {...}

class Engineer extends Employee {...}
```

- `중복코드`가 존재한다는 것은 한쪽의 변경이 다른 쪽에는 반영 되지 않을 수 있다는 위험을 항상 수반합니다.  
- 완전 중복코드라면 간단하게 메서드를 상속계층 위로 올릴 수 있겠지만, 리팩터링을 통해 공통부분을 찾아내서 위로 올리면 됩니다.

**12.2 필드 올리기**

![2]({{ site.images | relative_url }}/pages/refactoring/10-how-refactoring-inheritance/2.jpeg) 

```js
class Employee {...}

class Salesperson extends Employee {
    private String name;
}

class Engineer extends Employee {
    private String name;
}
```

```js
class Employee {
    protected String name;
}

class Salesperson extends Employee {...}

class Engineer extends Employee {...}
```

- 필드의 이름이 완전 같지 않더라도 비슷한 방식으로 쓰인다고 판단되면 슈퍼클래스로 올려서 사용하는 것이 좋습니다.
- 데이터 중복을 선언을 줄이고 해당 필드를 사용하는 동작 또한 슈퍼클래스에서 선언 할 수 있습니다.

**12.3 생성자 올리기**

![3]({{ site.images | relative_url }}/pages/refactoring/10-how-refactoring-inheritance/3.jpeg) 

```js
class Party {...}

class Employee extends Party {
    constructor(name, id, monthlyCost){
        super();
        this._id = id;
        this._name = name;
        this._monothlyCost = monthlyCost;
    }
}
```

```js
class Party {
    constructor(name) {
        this._name = name;
    }
}

class Employee extends Party {
    constructor(name, id, monthlyCost){
        super(name);
        this._id = id;
        this._monothlyCost = monthlyCost;
    }
}
```

- 필드 올리기와 함께 생성자도 올려줍니다.

**12.4 메서드 내리기**

![4]({{ site.images | relative_url }}/pages/refactoring/10-how-refactoring-inheritance/4.jpeg) 

```js
class Employee {
    get quota(){...}
}

class Salesperson extends Employee {...}

class Engineer extends Employee {...}
```

```js
class Employee {...}

class Salesperson extends Employee {
    get quota() {...}
}

class Engineer extends Employee {
    ...
}
```

- 특정 서브클래스에만 관련된 메서드는 슈퍼클래스에 제거하는것이 좋습니다.


**12.5 필드 내리기**

![5]({{ site.images | relative_url }}/pages/refactoring/10-how-refactoring-inheritance/5.jpeg) 

```js
class Employee {
    private String quota;
}

class Salesperson extends Employee {...}

class Engineer extends Employee {...}
```

```js
class Employee {...}

class Salesperson extends Employee {
    protected String quota;
}

class Engineer extends Employee {
    ...
}
```

- 특정 서브클래스에만 관련된 필드는 슈퍼클래스에 제거하는것이 좋습니다.


**12.6 타입 코드를 서브클래스로 바꾸기**

![6]({{ site.images | relative_url }}/pages/refactoring/10-how-refactoring-inheritance/6.jpeg) 

```js
function createEmployee(name, type){
    return new employee(name, type);
}
```

```js
function createEmployee(name, type){
    switch(type) {
        case "engineer" : return new engineer(name);
        case "salesperson": return new Salesperson(name);
        case "manager": return new Manager (name);
    }
}
```

- 서브클래스를 사용한다면 `다형성`과 타입별로 다른 동작을 정의할수 있는 장점이 있습니다.
- 서브클래스를 통해 관계를 좀더 명확히 드러내 줄 수 있습니다.

**12.7 서브클래스 제거하기**

![7]({{ site.images | relative_url }}/pages/refactoring/10-how-refactoring-inheritance/7.jpeg) 

```js
class Person {
    get genderCode() {return "X";}
}
class Male extends Person {
    get genderCode() {return "M";}
}
class Female extends Person {
    get genderCode() {return "F";}
}
```

```js
class Person {
    get genderCode() {
        return this._genderCode;
    }
}
```

- 불필요하게 사용되는 서브클래스들을 제거하고 간단하게 필드로 대체 할 수 있습니다.



**12.8 서브클래스 추출하기**

![8]({{ site.images | relative_url }}/pages/refactoring/10-how-refactoring-inheritance/8.jpeg) 

```js
class Department {
    get totalAnnualCost() {...}
    get name() {...}
    get headCount() {...}
}

class Employee {
    get annualCost() {...}
    get name() {...}
    get id() {...}
}
```

```js
class Party {
    get name() {...}
    get annualCost() {...}
}

class Department extends Party{
    get annualCost() {...}
    get headCount() {...}
}

class Employee extends Party{
    get annualCost() {...}
    get id() {...}
}
```

- 비슷한 일을 수행하는 클래스들이 보이면 상속을 이용해 비슷한 부분을 공통 슈퍼클래스로 옮겨 담는것을 고려합니다.

**12.9 계층 합치기**

![9]({{ site.images | relative_url }}/pages/refactoring/10-how-refactoring-inheritance/9.jpeg) 

```js
class Employee {...}
class Salesperson extends Employee {...}
```

```js
class Employee {...}
```

- 계층구조를 올리고 내리다보면 서브클래스와 부모클래스가 비슷해져서 계층을 합쳐버리는것이 더 좋을 경우가 발생하기도 합니다.

**12.10 서브클래스를 위임으로 바꾸기**

![10]({{ site.images | relative_url }}/pages/refactoring/10-how-refactoring-inheritance/10.jpeg) 

```js
class Order {
    get daysToShip() {
        return this._warehouse.daysToShip;
    }
}
class PriorityOrder extends Order {
    get daysToShip() {
        return this._priorityPlan.daysToShip;
    }
}
```

```js
class Order {
    get daysToShip() {
        return (this._priorityDelegate)
            ? this._prioirtyDelegate.daysToShip
            : this._warehouse.daysToShip;
    }
}
class PriorityOrder {
    get daysToShip() {
        return this._priorityPlan.daysToShip;
    }
}
```

- 상속은 기준이 정해지면 하나에대해서만 상속이 가능하고 클래스들의 관계를 아주 긴밀하게 결합시킨다는 단점이 있습니다.
- 위임을 사용한다면 결합도가 약해지고 상호작요에 필요한 인터페이스를 명확히 정의 할 수 있습니다.


**12.11 슈퍼클래스를 위임으로 바꾸기**

![11]({{ site.images | relative_url }}/pages/refactoring/10-how-refactoring-inheritance/11.jpeg) 

```js
class List {...}
class Stack extends List {...}
```

```js
class Stack {
    constructor() {
        this._storage = new List();
    }
}
class List {...}
```

- 객체지향에서 상속은 기존기능을 재활용 할 수 있는 강력하고 손쉬운 수단이지만, 상속이 혼란과 복잡도를 키우는 방식이 되기도 합니다
- 실제로 필요한 기능만 정의 해 `LCP(리스코프 치환원칙)`를 위반하지 않을 수 있습니다.

> 상속을 사용해도 좋은 경우  
> - 부모클래스의 모든 메서드가 서브 클래스에 적용 될때  
> - 서브 클래스의 모든 인스턴스가 부모클래스의 인스턴스로 치환도 가능할 때  
> - 의미상 적합한 조건일 때











---

### Reference

- `리팩터링 2판 ` 
(https://front.wemakeprice.com/product/822375110?search_keyword=%25EB%25A6%25AC%25ED%258C%25A9%25ED%2586%25A0%25EB%25A7%2581&_service=5&_no=1)