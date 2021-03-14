---
layout: post
comments: true
title: 어떻게 리팩토링을 해야할까 - 캡슐화
tags: [refactoring, encapsulation, oop]
---

### How do you refactorig? - 캡슐화

많이들 사용 하고 계시겠지만, `캡슐화`를 한다고하면 다음과같은 작업을 수행하게 됩니다.

- 객체의 속성(field)과, 함수(method)들을 한데 묶는다
- 외부 공개가 필요한 요소들만 노출시켜 내부 데이터나 기능들을 은닉시킨다

이러한 방법을 통해 객체지향에서 `캡슐화`가 갖는 의미를 아래와 같이 정의 할 수 있을것입니다.

- 응집도 강화
- 결합도 약화

즉, 관련된 데이터와 기능들을 한군데 묶어 응집도를 높게 만들고 다른 모듈간의 작업시에는 필요한 요소들만 사용 할 수 있도록해 결합도를 낮추는것이 목적이라고 할 수 있겠습니다.  

이번 포스팅에서는 캡슐화를 통한 리팩터링 방법들을 소개해보도록 하겠습니다.  


**7.1 레코드 캡슐화하기**
![1]({{ site.images | relative_url }}/posts/2021-03-14-how-refactoring-encapsulate/1.jpeg) 

```js
organization = {name: "애크미 구스베리", country: "GB"}
```

```js
class Organization
{
    constructor(data) 
    {
        this._name = data.name;
        this._country = data.country;
    }
    get name(){return this._name;}
    set name(arg){this._name=arg;}
    get country(){return this._country;}
    set country(arg){this._country=arg;}
}
```

- 정의된 레코드를 직접 참조하는 코드가 많을때, 캡슐화를 해주는것이 좋습니다.


**7.2 컬렉션 캡슐화하기**
![2]({{ site.images | relative_url }}/posts/2021-03-14-how-refactoring-encapsulate/2.jpeg) 

```js
class Person 
{
    get courses() {return this._courses;}
    set courses(aList) {this._courses = aList;}
}
```

```js
class Person 
{
    get courses() {return this._courses.slice();}
    addCourse(aCourse) {...}
    removeCourse(aCourse) {...}
}
```

- 객체에서 컬렉션을 다룰때 외부에서 컬렉션을 직접접근하지 못하도록 캡슐화 하는것이 좋습니다.
- 내부 컬렉션을 변경할때에도 내부 모듈의 수정만으로도 쉽게 변경 할 수 있을것입니다. (Map -> List) 


**7.3 기본형을 객체로 바꾸기**
![3]({{ site.images | relative_url }}/posts/2021-03-14-how-refactoring-encapsulate/3.jpeg) 

```js
orders.filter(o => "high"=== o.priority | "rush" === o.priority)
```

```js
orders.filter(o => o.priority.higherThan(new Priority("normal")))
```

- 숫자 혹은 문자같은 간단한 데이터 정보로 사용을 하다가 단순 데이터 정보를 넘어 기능들이 필요해질때에는, 객체로 만들어 주는것이 좋습니다.
- 특별한 기능이 추가될때마다 클래스화 된 데이터는 빛을 발하게 될 것입니다.

**7.4 임시 변수를 질의함수로 바꾸기**
![4]({{ site.images | relative_url }}/posts/2021-03-14-how-refactoring-encapsulate/4.jpeg) 

```js
const basePrice = this._quantity * this._itemPrice;
if (basePrice > 1000)
    return basePrice * 0.95;
else
    return basePrice * 0.98;
```

```js
get basePrice = {this._quantity * this._tiemPrice;}
...
if (basePrice > 1000)
    return basePrice * 0.95;
else
    return basePrice * 0.98;
```

- 임시 변수의 사용으로 표현식의 정의를 명확하게 나타내 줄 수 있지만, 한걸음 더 나아가 이를 함수로 만들어버리는것이 더 나을때가 많습니다.
- 비슷한 꼐산을 수행하는 곳에서 함수를 사용해 중복코드를 줄일수 있으며 특히나 클래스내부에서 적용할 때 이 효과는 가장 크게 나타납니다.


**7.5 클래스 추출하기**
![5]({{ site.images | relative_url }}/posts/2021-03-14-how-refactoring-encapsulate/5.jpeg) 

```js
class Person 
{
    get officeAreaCode() {return this._officeAreaCode;}
    get officeNumber() {return this.officeNumber;}
}
```

```js
class Person 
{
    get officeAreaCode() {return this._telephoneNumber.areaCode;}
    get officeNumber() {return this._telephoneNumber.number;}
}
class TelephoneNumber
{
    get areaCode() {return this._areaCode;}
    get number() {return this._number;}
}
```

- 클래스에 몇가지 연산을 추가하고 데이터 보강하면서 클래스가 비대해지곤 하는데, 역할이 많아질 수록 클래스가 굉장히 복잡해집니다.
- 일부 데이터와 메서드를 따로 묶을 수 있다면, 어서 분리하라는 신호입니다.
- 

**7.6 클래스 인라인하기**
![6]({{ site.images | relative_url }}/posts/2021-03-14-how-refactoring-encapsulate/6.jpeg) 

```js
class Person 
{
    get officeAreaCode() {return this._telephoneNumber.areaCode;}
    get officeNumber() {return this._telephoneNumber.number;}
}
class TelephoneNumber
{
    get areaCode() {return this._areaCode;}
    get number() {return this._number;}
}
```
```js
class Person 
{
    get officeAreaCode() {return this._telephoneNumber.areaCode;}
    get officeNumber() {return this._telephoneNumber.number;}
}
```

- `7.5 클래스 추출`을 통해 클래스를 분리시켰지만, 분리하고 나니 남은 역할이 거의 없는경우 다시 인라인 시켜 많이 사용하는 클래스로 흡수시키는편이 좋습니다.

**7.7 위임 숨기기**
![7]({{ site.images | relative_url }}/posts/2021-03-14-how-refactoring-encapsulate/7.jpeg) 

```js
manager = aPerson.department.manager;
```
```js
manager = aPerson.manager;

class Person
{
    get manager() {return this.department.manager;}
}
```

- 캡슐화는 모듈들이 시스템의 다른 부분에 대해 알아야 할 내용을 줄여 줄 수 있습니다.
- 위임객체의 인터페이스가 변경되는 결합된 모든 코드들을 변경해야 되는 상황이 발생 할 수 있습니다. 잘 캡슐화된 모듈은 이를 방지 할 수 있습니다.

**7.8 중재자 제거하기**
![8]({{ site.images | relative_url }}/posts/2021-03-14-how-refactoring-encapsulate/8.jpeg) 

```js
manager = aPerson.manager;

class Person
{
    get manager() {return this.department.manager;}
}
```
```js
manger = aPerson.department.manager;
depName = aPerson.department.name;
depCode = aPerson.department.code;
...
```

- `7.7 위임 숨기기`를 통해 클라이언트가 모듈의 내부 데이터 정보들을 몰라도 되게 숨겼지만, 해당 위임객체들의 다른 부분들을 사용 할 필요가 있는경우에는 차라리 위임객체를 직접 호출하는게 나을수 있습니다.

**7.9 알고리즘 교체하기**
![9]({{ site.images | relative_url }}/posts/2021-03-14-how-refactoring-encapsulate/9.jpeg) 

```js
function foundPerson(people)
{
    for(let i=0; i<people.length; i++)
    {
        if(peoplle[i] === "Don")return "Don";
        if(peoplle[i] === "John")return "John";
        if(peoplle[i] === "Kent")return "Kent";
    }
    return "";
}
```

```js
function foundPerson(people)
{
    const candidates = ["Don", "John", "Kent"];
    return people.find(p => candidates.includes(p)) || "";
}
```

- 목적을 달성하는법은 다양하게 존재합니다. 그중 더 쉬운방법, 더 간명한 방법을 찾아내면 기존의 복잡한 코드를 간단하고 명확하게 리팩터링 할 수 있습니다.
- 바로 알고리즘을 고치기에는 어려운 코드들이 많으니 기존 알고리즘을 이해하기 쉽게 리팩터링 하다보면 좀더 좋은 방법이 떠오르실수 있을것입니다. 

---

### Reference

- `리팩터링 2판 ` 
(https://front.wemakeprice.com/product/822375110?search_keyword=%25EB%25A6%25AC%25ED%258C%25A9%25ED%2586%25A0%25EB%25A7%2581&_service=5&_no=1)