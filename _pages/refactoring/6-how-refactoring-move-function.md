---
layout: post
comments: true
title: 어떻게 리팩토링을 해야할까 - 기능이동
navdisplay: false
tags: [refactoring]
---

### How do I refactorig? - 기능이동


**8.1 함수 옮기기**
![1]({{ site.images | relative_url }}/pages/refactoring/6-how-refactoring-move-function/1.jpeg) 

```js
class Account
{
    get overdraftCharge(){}
}
```

```js
class AccountType
{
    get overdraftCharge(){}
}
```

- 연관된 요소를 함께 묶어 모듈화를 하는것은 응집도를 높이고 결합도를 줄일수있는 방법입니다.
- 함수를 옮길때는 현재 컨텍스트와 후보 컨텍스트를 둘러보고 적당한지 확인해야합니다.

**8.2 필드 옮기기**
![2]({{ site.images | relative_url }}/pages/refactoring/6-how-refactoring-move-function/2.jpeg) 

```js
class Customer
{
    get plan(){return this._plan;}
    get discountRate(){return this._discountRate;}
}
```

```js
class Customer
{
    get plan(){return this._plan;}
    get discountRate(){return this.plan.discountRate;}
}
```

- 적합한 데이터 구조에서 동작코드는 자연스럽고 단순하고 직관적으로 짤수있게됩니다.
- 현재 데이터구조가 적절하지 않음이 확인되면 바로 바꾸는것이 좋습니다..

**8.3 문장을 함수로 옮기기**
![3]({{ site.images | relative_url }}/pages/refactoring/6-how-refactoring-move-function/3.jpeg) 

```js
result.push('<p>제목: ${person.photo.title}></p>');
result.concat(photoData(person.photo));

function photoData(aPhoto)
{
    return[
        '<p>위치: ${aPhto.location}</p>',
        '<p>날짜: ${aPhto.date.toDateString()}</p>',
    ]
}
```

```js
result.concat(photoData(person.photo));

function photoData(aPhoto)
{
    return[
        '<p>제목: ${person.photo.title}></p>',
        '<p>위치: ${aPhto.location}</p>',
        '<p>날짜: ${aPhto.date.toDateString()}</p>',
    ]
}
```

- 반복되는 코드를 제거하는 것은 코드를 건강하게 관리하는 가장 효과적인 방법입니다. 자주 함께 사용되는 문장은 같이 함수화 시켜주는것이 좋습니다.

**8.4 문장을 호출한 곳으로 옮기기**
![4]({{ site.images | relative_url }}/pages/refactoring/6-how-refactoring-move-function/4.jpeg) 

```js
emitPhotoData(outStream, person.photo);

function emitPhotoData(outStream, photo)
{
    outStream.write('<p>제목: ${person.photo.title}></p>');
    outStream.write('<p>위치: ${aPhto.location}</p>');
}
```

```js
emitPhotoData(outStream, person.photo);
outStream.write('<p>제목: ${person.photo.title}></p>');

function emitPhotoData(outStream, photo)
{
    outStream.write('<p>위치: ${aPhto.location}</p>');
}
```

- 여러군데서 조금씩 다르게 중복되는경우달라진 동작을 함수에서 꺼는것이 더 좋을수도 있습니다.
- 작은변경이라면 괜찮지만, 더 적합한 경계를 찾아보는것이 더 좋을 수 있습니다.

**8.5 인라인 코드를 함수 호출로 바꾸기**
![5]({{ site.images | relative_url }}/pages/refactoring/6-how-refactoring-move-function/5.jpeg) 

```js
let appliesToMass = false;
for(const s of states) 
{
    if (s === "MA") appliesToMass = true;
}
```

```js
appliesToMass = states.includes("MA");
```

- 함수화는 여러동작을 하나로 묶어주고, 함수 이름으로써 코드를 이해하기 쉽게 만들어줄 수 있습니다.
- 특히 라이브러리가 제공하는 함수로 대체할 수 있으면 좋습니다.

**8.6 문장 슬라이드하기**
![6]({{ site.images | relative_url }}/pages/refactoring/6-how-refactoring-move-function/6.jpeg) 

```js
const pricingPlan = retrievePricingPlan();
const order = retreiveOrder();
let charge;
const chargePerUnit = pricingPlan.unit;
```

```js
const pricingPlan = retrievePricingPlan();
const chargePerUnit = pricingPlan.unit;
const order = retreiveOrder();
let charge;
```

- 관련된 코드를 가까이 위치시켜주는것은 코드를 이해하기 더 쉽게만드는 방법입니다.
- 변수는 처음 사용할때 선언하는것이 좋습니다.

**8.7 반복문 쪼개기**
![7]({{ site.images | relative_url }}/pages/refactoring/6-how-refactoring-move-function/7.jpeg) 

```js
let averageAge = 0;
let totalSalary = 0;
for (const p of people)
{
    averageAge += p.age;
    totalSalary += p.salary;
}
averageAge = averageAge / people.length;
```

```js
let totalSalary = 0;
for (const p of people)
{
    totalSalary += p.salary;
}
let averageAge = 0;

for (const p of people)
{
    averageAge += p.age;
}
averageAge = averageAge / people.length;
```

- 반복문 하나에서 여러작업을 수행한다면, 반복문이 수정될때마다 두가지 일 모두를 신경써써 작업해야합니다. 반복문의 분리로 작업 관심을 분리시켜주는것이 좋습니다.
- 반복문을 두번 실행해야되기 때문에 불편하다고 생각할수도있지만, `리팩터링`과 `성능최적화`의 목적이 다름을 기억해야합니다. 

**8.8 반복문을 파이프라인으로 바꾸기**
![8]({{ site.images | relative_url }}/pages/refactoring/6-how-refactoring-move-function/8.jpeg) 

```js
const names = [];
for (const i of input) 
{
    if(i.job === "programmer")
    {
        names.push(i.name);
    }
}
```

```js
const names = input
    .filter(i => i.job === "programmer")
    .map(i => i.name);
```

- 파이프라인은 그자체로써 논리적 흐름을 나타낼 수 있습니다.
- https://martinfowler.com/articles/refactoring-pipelines.html#FinalThoughts

> That concludes this set of refactoring examples. I hope it's given you a good sense of how collection pipelines can clarify the logic of code that manipulates a collection, and how it's often quite straightforward to refactor a loop into a collection pipeline.
> 
> As with any refactoring, there is a similar inverse refactoring to turn a collection pipeline into a loop, but I hardly ever do that.
> 
> These days most modern languages offer first class functions and a collection library that includes the necessary operations for collection pipelines. If you're unused to collection pipelines, it is a good exercise to take loops that you run into and refactor them like this. If you find the final pipeline isn't clearer than the original loop, you can always revert the refactoring when you're done. Even if you do revert the refactoring, the exercise can teach you a lot about this technique. I've been using this programming pattern for a long time and find it a valuable way of helping me read my own code. As such I think it's worth the effort of exploring, to see if your team comes to a similar conclusion.


**8.9 죽은코드 제거하기**
![9]({{ site.images | relative_url }}/pages/refactoring/6-how-refactoring-move-function/9.jpeg) 

```js
if(false)
{
    doSomethingThatUsedToMatter();
}
```

```js
```

- 사용되지 않는 코드는 그 소프트웨어의 동작을 이해하는데 커다란 걸림돌이 될 수 있습니다.
- 코드가 나중에 사용되지 않을까 걱정할 필요는 없습니다. 우리에겐 형상관리 시스템이 있습니다.
---

### Reference

- `리팩터링 2판 ` 
(https://front.wemakeprice.com/product/822375110?search_keyword=%25EB%25A6%25AC%25ED%258C%25A9%25ED%2586%25A0%25EB%25A7%2581&_service=5&_no=1)