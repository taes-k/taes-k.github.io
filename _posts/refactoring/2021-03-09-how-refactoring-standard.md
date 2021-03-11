---
layout: post
comments: true
title: 어떻게 리팩터링을 해야할까 - 기본방법
tags: [refactoring]
---

### How do you refactorig? - 기본방법

이번 포스팅부터는 실제 코드에서 어떻게 리팩터링을 진행하는지 여러가지 방법을 소개하려합니다.  
책에서는 디테일한 다양한 상황들을 예제로 리팩터링을 하는 기법들을 추가로 이야기 하고 있으니 기회가 되신다면 책을 직접 읽어보시는 것을 추천드립니다.

**6.1 함수 추출하기** 

![1]({{ site.images | relative_url }}/posts/2021-03-09-how-refactoring-standard/1.jpeg) 

예제)
```js
function printOwing(invoice)
{
    printBanner();
    let outstanding = caculateOutstanding();

    //세부 사항 출력
    console.log('고객명: ${invoice.customer}');
    console.log('채무액: ${outstannding}');
}
```

```js
function printOwing(invoice)
{
    printBanner();
    let outstanding = caculateOutstanding();
    printDetails(outstanding);

    function printDetails(outstanding)
    {
        console.log('고객명: ${invoice.customer}');
        console.log('채무액: ${outstannding}');
    }
}
```

- `목적과 구현을 분리` 시키기 위함 입니다.
- 함수의 코드를 보고 바로 목적을 바로 알 수 있도록 해야합니다.
- 함수생성시 `목적`을 잘 드러내는 이름을 붙여야 합니다. (`어떻게`가 아닌 `무엇을`)

> 값을 반환해야할 변수가 여러개라면?  
> - 각각을 반환하는 함수 여러개를 만듭니다.
> - 반환값들을 레코드로 묶어서 반환해도 되지만 다른방식으로 처리하는것이 나을때가 많습니다.
> - 임시변수를 질의함수로 바꾸거나, 변수를 쪼개는 등의 방법을 고려하는것이 좋습니다.

**6.2 함수 인라인하기**

![2]({{ site.images | relative_url }}/posts/2021-03-09-how-refactoring-standard/2.jpeg) 

```js
function getRating(driver) 
{
    return moreThanFiveLateDeliveries(driver) ? 2 : 1;
}

function moreThanFiveLateDeliveries(driver)
{
    return driver.numberOfLateDeliveries > 5;

}
```

```js
function getRating(driver) 
{
    return (driver.numberOfLateDeliveries > 5) ? 2 : 1;
}
```

- 함수 코드 자체가 이름만큼 명확한 경우라면 함수를 제거하는것이 나을수도 있습니다.
- 간접 호출이 과하게 쓰이는 코드의경우 위임관계가 복잡해질수 있기때문에 인라인하는것이 더 나을수 있습니다.

**6.3 변수 추출하기**

![3]({{ site.images | relative_url }}/posts/2021-03-09-how-refactoring-standard/3.jpeg) 

```js
return order.quantity * order.itemPrice 
- Math.max(0, order.quantity - 500) * order.itemPrice * 0.0.5 
+ Math.min(order.quantity * order.itemPrice * 0.1, 100)
```
```js
const basePrice = order.quantity * order.itemPrice;
const quantityDiscount = Math.max(0, order.quantity - 500) * order.itemPrice * 0.05;
const shipping = Math.min(basePrice * 0.1, 100);
return basePrice - quantityDiscount + shipping;
```

- 이해하기 어려운 표현식을 쪼개 변수로써 지정해주면 코드의 목적을 명확하게 드러낼 수 있습니다.
- 하나의 함수내에서만 사용하는 경우에는 변수로 추출하지만, 함수를 넘어서는 문맥에서 통용되는경우에는 함수로 추출하는것이 좋습니다.

```js
get basePrice {return order.quantity * order.itemPrice;}
get quantityDiscount{return Math.max(0, order.quantity - 500) * order.itemPrice * 0.05;}
get shipping {return Math.min(basePrice * 0.1, 100);}
```

**6.4 변수 인라인하기**

![4]({{ site.images | relative_url }}/posts/2021-03-09-how-refactoring-standard/4.jpeg) 

```js
let basePrice = anOrder.basePrice;
return (basePrice > 1000);
```
```js
return anOrder.basePrice > 1000;
```

- 변수명이 원래의 표현식과 다를바 없을때에는 그냥 인라인을 하는것이 좋습니다.

**6.5 함수 선언 바꾸기**

![5]({{ site.images | relative_url }}/posts/2021-03-09-how-refactoring-standard/5.jpeg) 

```js
function circum(radius)
{
    ...
}
```
```js
function circumference(radius)
{
    ...
}
```

- 함수의 이름이 명확하면 함수코드를 살펴볼 필요없이 해당 호출문만 보고도 무슨일을 하는지 파악 할 수 있습니다
- 좋은이름이 떠오르지 않을때는 함수의 목적에대한 설명을 주석으로 적어보면 주석이 멋진 이름으로 돌아올 수 있습니다

**6.6 변수 캡슐화하기**

![6]({{ site.images | relative_url }}/posts/2021-03-09-how-refactoring-standard/6.jpeg) 

```js
let defaultOwner = {firstName: "마틴", lastName: "파울러"};
```

```js
let defaultOwner = {firstName: "마틴", lastName: "파울러"};
export function defaultOwner() {return defaultOwnerData;}
export function setDefaultOwner(arg) {defaultOwnerData = arg;}
```

- 데이터의 캡슐화는 데이터를 변경하고 사용하는 코드를 감시할 수 있느 확실한 통로가 되어줍니다.
- 데이터의 유효범위가 넓을수록 캡슐화 해야 합니다.


**6.7 변수 이름 바꾸기**

![7]({{ site.images | relative_url }}/posts/2021-03-09-how-refactoring-standard/7.jpeg) 

```js
let a = hegith * width;
```
```js
let area = height * width;
```

- 이름이 잘지어진 변수는 작업 로직에 대해 많은것을 설명해줍니다.
- 람다식에서 사용되어지는 변수는 맥락으로부터 대체로 쉽게 파악이 가능해 한글자 이름을 짓기도 합니다.
- 사용 범위가 넓은 변수일수록 이름을 잘 지어주는것이 좋습니다.

**6.8 매개변수 객체 만들기**

![8]({{ site.images | relative_url }}/posts/2021-03-09-how-refactoring-standard/8.jpeg) 

```js
function amountInvoiced(startDate, endDate) {...}
function amouontReceived(startDate, endDate) {...}
function amountOverdue(startDate, endDate) {...}
```

```js
function amountInvoiced(aDateRange) {...}
function amouontReceived(aDateRange) {...}
function amountOverdue(aDateRange) {...}
```

- 데이터 항목 여러개가 자주 함께 사용되는 경우 데이터 구조 하나로 모아주는것이 좋습니다.
- 데이터를 뭉쳐주면 데이터 사이의 관계가 명확해집니다.
- 매개변수가 줄어둘고 일관성이 높아집니다.

> 데이터 객체  
> - 단순히 매개변수를 묶어주는 기능 외에 `객체`로써의 역할을 할 수 있게 됩니다.
> - 관련 동작들을 해당 함수에 추가 시켜 줄수 있는 메서드들이 추가 될 수 있습니다.
> ``` js
> // aDateRange
> function isValidDate(targetDate)
> {
>    return (targetDate >= startDate && targetDate <= endDate);  
> }
> ```

**6.9 여러 함수를 클래스로 묶기**

![9]({{ site.images | relative_url }}/posts/2021-03-09-how-refactoring-standard/9.jpeg) 

```js
function base(aReading){...}
function taxableCharge(aReading){...}
function calculateBaseCharge(aReading){...}
```
```js
class Reading 
{
    base() {...}
    taxableCharge() {...}
    calculateBaseCharge() {...}
}
```

- 함수들이 공유하는 공통데이터가 있는경우 하나의 클래스로 묶어주면 공통 데이터 중심으로 긴밀하게 작동 할 수 있게됩니다.(변수 객체 클래스)

**6.10 함수 추출하기**

![10]({{ site.images | relative_url }}/posts/2021-03-09-how-refactoring-standard/10.jpeg) 

```js
function base(aReading){...}
function taxableCharge(aReading){...}
```

```js
function enrichReading(argReading)
{
    const aReading = _.cloneDeep(argReading);
    aReading.baseCharge = base(aReading);
    aReading.taxableCharge = taxableCharge(aReading);
    return aReading;
}
```

- 데이터를 입력받아 여러 함수들으로 로직이 생성되는 경우 하나의 클래스로 묶어주는것이 좋습니다.
- 다만, 원본데이터가 코드내에서 갱신될 때에는 클래스로 묶는것이 좋습니다.


**6.11 단계 쪼개기**

![11]({{ site.images | relative_url }}/posts/2021-03-09-how-refactoring-standard/11.jpeg) 

```js
const orderData = orderString.split(/\s+/);
const productPrice = priceList[orderData[0].split("-")[1]];
const orderPrice = parseInt(orderData[1])*productPrice;
```

```js
const roderRecord = parseOrder(order);
const orderPrice = price(orderRecord, priceList);

function parseOrder(aString)
{
    const values = aString.split(/\s+/);
    return ({
        productID: values[0].split("-")[1],
        quantity: parseInt(valules[1]);        
    });
}
function price(order, priceList)
{
    return order.quantity * priceList[order.productID];
}
```

- 서로 다른 두 값을 한번에 다루는 코드를 발견시 별개의 모듈로 나누는것이 좋습니다.
- 하나의 연속되는 동작을 여러 단계로 쪼개어 모듈화 시키는 방법입니다.
- 단계를 쪼개어 입력값을 다루기에 편한 형태로 가공해줄수 있습니다.
- 모듈화시켜 해당 단계에서 작업하는사항을 명확하게 드러낼수있습니다.

---

### Reference

- `리팩터링 2판 ` 
(https://front.wemakeprice.com/product/822375110?search_keyword=%25EB%25A6%25AC%25ED%258C%25A9%25ED%2586%25A0%25EB%25A7%2581&_service=5&_no=1)