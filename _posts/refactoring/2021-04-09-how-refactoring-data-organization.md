---
layout: post
comments: true
title: 어떻게 리팩토링을 해야할까 - 데이터 조직화
tags: [refactoring]
---

### How do you refactorig? - 데이터 조직화

**9.1 변수 쪼개기**
![1]({{ site.images | relative_url }}/posts/2021-04-09-how-refactoring-data-organization/1.jpeg) 

```js
let temp = 2 * (height + width);
console.log(temp;
temp = height * width);
console.log(temp)'
```

```js
const perimeter = 2 * (height + width);
console.log(perimeter);
const area = height * width;
console.log(area);
```

- 대입이 두번이상 이루어지는 변수는 여러가지 역할을 수행한다는 신호 입니다.
- 여러 역할을 수행하는 변수는 큰 혼란을 줄 수 있습니다.


**9.2 필드 이름 바꾸기**
![2]({{ site.images | relative_url }}/posts/2021-04-09-how-refactoring-data-organization/2.jpeg) 

```js
class Organization{
    get name() {...}
}
```

```js
class Organization{
    get title() {...}
}
```

- 프로그램을 더 쉽게 이해 하기 위해 데이터 필드를 명확하게 지어주는것은 중요합니다.

**9.3 파생된 변수를 함수로 바꾸기**
![3]({{ site.images | relative_url }}/posts/2021-04-09-how-refactoring-data-organization/3.jpeg) 

```js
get discountedTotal() {return this._discountedTotal;}
set discount(aNumber) {
    const old = this._discount;
    this._discount = aNumber;
    this._discountedTotal += old - aNumber;
}
```

```js
get discountedTotal() { return this._baseTotal - this._discount;}
get discount(aNumber) {this._discount = aNumber;}
```

- 가변데이터의 유효범위를 최대한 줄여야합니다.
- 값을 쉽게 계산해 낼 수 있는 변수들은 그 자체를 함수화시켜 변수를 사용하지 않을 수 있습니다.

**9.4 참조를 값으로 바꾸기**
![4]({{ site.images | relative_url }}/posts/2021-04-09-how-refactoring-data-organization/4.jpeg) 

```js
class Product {
    applyDiscount(arg) {this._price.amount -= arg;}
}
```

```js
class Product{
    this._price = new Money(this._price.amount - arg, this._price.currency);
}
```

- 외부 객체를 참조로 담는경우 의도치 않은 외부적 요인에 의해 객체가 변하지 않도록 주의해야 합니다.
- 필드를 값으로 다룬다면, 내부에 영향을 줄까 고려하지 않아도 됩니다.

**9.5 값을 참조로 바꾸기**
![5]({{ site.images | relative_url }}/posts/2021-04-09-how-refactoring-data-organization/5.jpeg) 

```js
let customer = new Customer(customerData);
```

```js
let customer = customerRepository.get(customerData.id);
```

- 동일한 데이터를 참조해야하는데, 모든 객체들이 각각 복제된 데이터라면 데이터가 갱신될때 문제가 생길것입니다.
- 데이터의 일관성이 필요한 경우라면 복제된 데이터들은 모두 참조로 바꾸어 동일한 소스를 갖게하는것이 좋습니다.


**9.6 매직 리터럴 바꾸기**
![6]({{ site.images | relative_url }}/posts/2021-04-09-how-refactoring-data-organization/6.jpeg) 

```js
function potentailEnergy(mass, height) {
    retunr mass * 9.81 * height;
}
```

```js
const STANDARD_GRAVITY = 9.81;
function potentialEnergy(mass, height) {
    return mass * STANDARD_GRAVITY * height;
}
```

- 코드에서 사용된 임의의 리터럴값은 읽는 사람에 따라 그자체의 의미를 모를수 있습니다.
- 코드 자체가 뜻을 분명하게 드러내기 위해 상수로 정의해주는것이 좋습니다.

---

### Reference

- `리팩터링 2판 ` 
(https://front.wemakeprice.com/product/822375110?search_keyword=%25EB%25A6%25AC%25ED%258C%25A9%25ED%2586%25A0%25EB%25A7%2581&_service=5&_no=1)