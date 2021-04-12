---
layout: post
comments: true
title: 어떻게 리팩토링을 해야할까 - API 리팩터링
navdisplay: false
tags: [refactoring]
---

### How do I refactorig? - API 리팩터링

**11.1 질의 함수와 변경 함수 분리하기**

![1]({{ site.images | relative_url }}/pages/refactoring/9-how-refactoring-data-api/1.jpeg) 

```js
function getTotalOutstandingAndSendBill() {
    const result = customer.invoices.reduce((total, each) => each.amount +total, 0);
    sendBill();
    return result;
}
```

```js
function totalOutstanding(){
    return customer.invoices.reduce((total, each) => each.amount + total, 0);
}
function sendBill() {
    emailGateway.send(formatBill(customer));
}
```

- API 만들때, 부수효과가 없도록 만들어야 언제 어디서나 원하는만큼 호출해도 문제가 생기지 않습니다.
- 만약 부수효과가 일어나는 API의 경우 명확히 구분해주는것이 좋습니다.
- '질의 함수(읽기 함수)는 반드시 부수효과가 없어야한다'를 따르는것도 좋습니다.


**11.2 함수 매개변수화하기**

![2]({{ site.images | relative_url }}/pages/refactoring/9-how-refactoring-data-api/2.jpeg) 

```js
function tenPercentRaise(aPerson){
    aPerson.salary = aPerson.salary.multiply(1.1);
}
function fivePercentRaise(aPerson) {
    aPerson.salary = aPerson.salary.multiply(1.05);
}

```

```js
function raise(aPerson, factor) {
    aPerson.salary = aPerson.salary.multiply(1 + factor);
}
```

- 함수의 로직이 비슷하고 단순히 리터럴값만 다른경우 매개변수를 받아 처리하면 함수의 유용성이 커질 수 있습니다.

**11.3 플래그 인수 제거하기**

![3]({{ site.images | relative_url }}/pages/refactoring/9-how-refactoring-data-api/3.jpeg) 

```js
function setDimension(name, value) {
    if(name === "height") {
        this._height = value;
        return;
    }
    if(name === "widht") {
        this._width = value;
        return;
    }
}

```

```js
function setHeight(value) {this._height = value;}
function setWidth(value) {this._width = value;}
```

- 플래그를 통한 함수호출을 하게되면 함수의 의미 전달이 어려워 질 수 있습니다.
- 플래그 인수가 둘 이상이면 함수 하나가 너무 많은 일을 처리하고 있다고 볼 수 있습니다.

**11.4 객체 통째로 넘기기**

![4]({{ site.images | relative_url }}/pages/refactoring/9-how-refactoring-data-api/4.jpeg) 

```js
const low = aRoom.daysTempRange.low;
const high = aRoom.daysTempRange.high;
if (aPlan.withinRange(low, high))

```

```js
if (aPlan.withinRange(aRoom.daysTempRange))
```

- 레코드를 통째로 넘기면 변화에 대응하기 쉬워질 수 있습니다.

**11.5 매개변수를 질의 함수로 바꾸기**

![5]({{ site.images | relative_url }}/pages/refactoring/9-how-refactoring-data-api/5.jpeg) 

```js
availableVacation(anEmployee, anEmployee.grade);

function availableVacation(anEmployee, grade) {
    // 연휴 계산...
}
```

```js
availableVacation(anEmployee);

function availableVacation(anEmployee) {
    const grade = anEmployee.grade;
    // 연휴 계산...
}
```

- 매개변수는 짧을수록 이해하기 쉽습니다.
- 매개변수를 제거함으로서 값을 결정하는 주체가 변경되는데, 호출하는쪽을 간소하게 만들수록 함수사용이 쉬워질수 있습니다.

**11.6 질의 함수를 매개변수로 바꾸기**

![6]({{ site.images | relative_url }}/pages/refactoring/9-how-refactoring-data-api/6.jpeg) 

```js
targetTemperature(aPlan)

function targetTemperature(aPlan) {
    currentTemperature = thermostat.currentTemperature;
    // ...
}
```

```js
targetTemperature(aPlan, thermostat.currentTemperature)

function targetTemperature(aPlan, currentTemperature) {
    // ...
}
```

- 의존성 제거가 필요한 함수의경우 매개변수로 변경하는것이 좋습니다.
- 같은 값을 건낼때 항상 동일한 결과를 얻을 수 있는 `참조 투명성`을 유지해주는것이 좋습니다.


**11.7 세터 제거하기**

![7]({{ site.images | relative_url }}/pages/refactoring/9-how-refactoring-data-api/7.jpeg) 

```js
class Person {
    get name() {...}
    set name(aString) {...}
}
```

```js
class Person {
    get name() {...}
}
```

- 세터가 존재함은 필드가 수정될 수 있다는 뜻입니다.
- 세터를 없애 수정하지 않겠다는 의도가 명백히 드러낼 수 있습니다.

**11.8 생성자를 팩터리 함수로 바꾸기**

![8]({{ site.images | relative_url }}/pages/refactoring/9-how-refactoring-data-api/8.jpeg) 

```js
leadEngineer = new Employee(document.leadEngineer, 'E');
```

```js
leadEngineer = createEngineer(document.leadEngineer);
```

- 생성자의 제약 (항상 정의된 클래스를 반환해함)을 피하기 위해 팩터리 함수를 사용 할 수 있습니다.
- 좀더 의미가 명확한 이름을 가진 함수로 호출 할 수 있습니다.

**11.9 함수를 명령으로 바꾸기**

![9]({{ site.images | relative_url }}/pages/refactoring/9-how-refactoring-data-api/9.jpeg) 

```js
function score(candidate, medicalExam, scoringGuide) {
    let reulst = 0;
    let healthLevel = 0;
}
```

```js
class Scorer {
    constructor(candidate, medicalExam, scoringGuide){
        this._candidate = candidate;
        this._medicalExam = medicalExam;
        this._scoringGuid = scoringGuid;
    }

    execute() {
        this._result = 0;
        this._healthLevel = 0;
    }
}
```

- 함수만을 위한 객체 -> 명령 객체를 만들어 캡슐화 하면 더 유용해지는 상황이 있습니다.
- 평범한 함수 메커니즘보다 훨씬 유연하게 핳ㅁ수를 제어하고 표현 할 수 있습니다.


**11.10 명령을 함수로 바꾸기**

![10]({{ site.images | relative_url }}/pages/refactoring/9-how-refactoring-data-api/10.jpeg) 


```js
class ChargeCalculator {
    constructor(customer, usage){
        this._customer = customer;
        this._usage = usage;
    }

    execute() {
        return this._custoemr.rate * this._usage;
    }
}
```

```js
function charge(customer, usage) {
    return customer.rate * usage;
}
```

- 로직이 크게 어렵거나 복잡하지않다면 명령객체를 사용하는것보다 함수로 바꿔주는것이 좋습니다.


**11.11 수정된 값 반환하기**

![11]({{ site.images | relative_url }}/pages/refactoring/9-how-refactoring-data-api/11.jpeg) 


```js
let totalAscent = 0;
calculateAscent();

function calculateAscent() {
    for (let i = 1; i<points.length; i++){
        const verticalChange = points[i].elevation - points[i-1].elevation;
        totalAscent += (verticalChange > 0)?verticalChange : 0;
    }
}
```

```js
let totalAscent = calculateAscent();

function calculateAscent() {
    let result = 0;
    for (let i = 1; i<points.length; i++){
        const verticalChange = points[i].elevation - points[i-1].elevation;
        result += (verticalChange > 0)?verticalChange : 0;
    }
    return result;
}
```

- 데이터가 어떻게 수정되는지 추적하는일은 코드에서 이해하기 가장 어려운 부분중 하나입니다.
- 데이터를 수정하는 코드가 여러곳에 퍼져있다면, 데이터의 흐름을 파악하기가 매우 어려워 질 수 있습니다.
- 객체를 넘겨 함수 내부에서 데이터를 수정하는 대신, 변경이 필요한 데이터를 함수 return 값으로 넘겨 받아 수정하게되면 수정값을 명확히 파악 할 수 있습니다.


**11.12 오류코드를 예외로 바꾸기**

![12]({{ site.images | relative_url }}/pages/refactoring/9-how-refactoring-data-api/12.jpeg) 


```js
if (data)
    return new ShippingRules(data);
else
    return -23;
```

```js
if (data)
    return new ShippingRules(data);
else
    throw new OrderProcessingError(-23);
```

- 예외를 발생시키면 오류코드를 일일히 찾아나서지 않아도 됩니다.
- 예외는 정확히 예상 밖의 동작일때 사용되어야 합니다.


**11.13 예외를 사전확인으로 바꾸기**

![13]({{ site.images | relative_url }}/pages/refactoring/9-how-refactoring-data-api/13.jpeg) 


```js
double getValueForPeriod (int periodNumber) {
    try {
        return values[periodNuimber];
    } catch (ArrayIndexOutOfBoundsException e){
        return 0;
    }
}
```

```js
double getValueForPeriod (int periodNumber) {
    return (periodNumber >= values.length) ? 0 : values[periodNumber];
}
```

- 예외는 '뜻밖의 오류'로 사용될때만 쓰여야 합니다.
- 가능하다면, 예외가 발생하기전에 검사 할 수 있다면 예외를 던지는 대신 조건을 검사해주는것이 좋습니다.

---

### Reference

- `리팩터링 2판 ` 
(https://front.wemakeprice.com/product/822375110?search_keyword=%25EB%25A6%25AC%25ED%258C%25A9%25ED%2586%25A0%25EB%25A7%2581&_service=5&_no=1)