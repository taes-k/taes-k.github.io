---
layout: post
comments: true
title: 어떻게 리팩토링을 해야할까  - 조건로직 간소화하기
navdisplay: false
tags: [refactoring]
---

### How do I refactorig? - 조건로직 간소화하기

**10.1 조건문 분해하기**

![1]({{ site.images | relative_url }}/pages/refactoring/8-how-refactoring-data-simplify-conditional-logic/1.jpeg) 

```js
if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerend))
    charge = quantity * plan.summerRate;
else
    charge = quantity * plan.regularRate + plan.regularServiceCharge;
```

```js
if (summer())
    charge = summerCharge()
else
    charge = regularCharge();
```

- 복잡한 조건부 로직은 해당 조건을 명확하게 표현하지 못하는 경우가 많습니다.
- 조건을 함수화해서 조건이 의미하는 바를 명확하게 강조해 무엇을 분기했는지 명백하게 표현해줍니다.


**10.2 조건식 통합하기**

![2]({{ site.images | relative_url }}/pages/refactoring/8-how-refactoring-data-simplify-conditional-logic/2.jpeg) 

```js
if (anEmployee.seniority < 2) return 0;
if (anEmployee.monthsDisabled > 12) return 0;
if (anEmployee.isPartTime) return 0;
```

```js
if (isNotEligibleForDisability) return 0;

function isNotEligibleForDisability() {
    
    return (anEmployee.seniority < 2) 
        || (anEmployee.monthsDisabled > 12);
        || (anEmployee.isPartTime); 
}
```

- 조건은 다르지만 결과 수행 로직은 같은경우 하나의 조건을 통해 합쳐주는것이 좋습니다.
- 여러개로 나뉘어진 조건들이 하나로 통합됨으로써 작업 로직이 더 명확해지고, `함수 추출하기`를 할 수 있는 시발점이 될 수 있습니다.


**10.3 중첩 조건문을 보호 구문(Guard clauses)으로 바꾸기**  

![3]({{ site.images | relative_url }}/pages/refactoring/8-how-refactoring-data-simplify-conditional-logic/3.jpeg) 

```js
function getPayAmount() {
    let result;
    if (isDead)
        result = deadAmount();
    else {
        if (isSeparated)
            result = separatedAmount();
        else {
            if (isRetired)
                result = retiriedAmount();
            else
                result = normalPayAmount();
        }
    }
    return result;
}
```

```js
function getPayAmount() {
    if (isDead) return deadAmount();
    if (isSeparated) return separatedAmount();
    if (isRetired) return retiriedAmount();
    return normalPayAmount();
}
```

- if-else 구문은 if문과 else문 모두 코드 로직상 중요하게 여겨지는데, 보호구문을 사용하게되면 의도를 좀더 명확하게 드러낼 수 있습니다.


**10.4 조건부 로직을 다형성으로 바꾸기**  

![4]({{ site.images | relative_url }}/pages/refactoring/8-how-refactoring-data-simplify-conditional-logic/4.jpeg) 

```js
switch (bird.type) {
    case '유럽 제비':
        return "보통이다";
    case '아프리카 제비':
        return (bird.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
    case '노르웨이 파랑 앵무':
        return (bird.voltage > 100) ? "그을렸다" : "예쁘다";
    default:
        return "알 수 없다";
}
```

```js
class EuropeanSwallow{
    get plumage() {
        return "보통이다"ㅣ
    }
    class AfricanSwallow {
        get Plumage() {
            return (bird.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
        }
    }
    class NorwegianBlueParrot {
        get Plumage() {
            return (bird.voltage > 100) ? "그을렸다" : "예쁘다";
        }
    }
}
```

- 복잡한 조건은 클래스와 다형성을 이용하면 로직을 직관적으로 이해하기 쉽게 변경 할 수 있습니다.


**10.5 특이 케이스 추가하기**  

![5]({{ site.images | relative_url }}/pages/refactoring/8-how-refactoring-data-simplify-conditional-logic/5.jpeg) 

```js
if (aCustomer === "미확인 고객") customerName = "거주자";
```

```js
class UnknownCustomer {
    get name() { return "거주자";}
}
```

- 특정한 데이터값을 확인해 동작을 수행하는코드가 여러군데 사용도디는경우 하나의 요소로 묶어서 사용할 수 있도록 해주는것이 좋습니다.
- 특이 케이스를 클래스화 하거나, 메스드화 하는 등의 방법이 있습니다.


**10.6 assertion 추가하기**  

![6]({{ site.images | relative_url }}/pages/refactoring/8-how-refactoring-data-simplify-conditional-logic/6.jpeg) 

```js
if (this.discountRate)
    base = base - (this.discountRate * base);
```

```js
assert(this.discountRate >= 0);
if (this.discountRate)
    base = base - (this.discountRate * base);
```

- 특정조건이 참일 때만 정상동작하는 코드가 있을때, 단순 코드만 보고서는 이해하기 어려울 수 있습니다.
- 위와 같은경우 코드에 `assertion`을 추가 함으로서, 개발자에게 어떤상태임을 가정한 채 실행되는지 알려주는 코드에 기록을 해주는것이 좋습니다.
- 단, `assertion`의 유무가 프로그램의 결과에 영향을 주어서는 안됩니다.

**10.7 제어 플래그를 탈출문으로 바꾸기**  

![7]({{ site.images | relative_url }}/pages/refactoring/8-how-refactoring-data-simplify-conditional-logic/7.jpeg) 

```js
for (const p of people){
    if (!found) {
        if (p === '조커') {
            sendAlert();
            found = true;
        }
    }
}
```

```js
for (const p of people){
    if (p === '조커') {
        sendAlert();
        break;
    }
}
```

- 다른 조건문을 검사하는데 사용되는 `제어플래그`를 사용하는 것 보다는, `break`, `continue`, `return` 과 같은 탈출문을 사용하는것이 좋습니다.

---

### Reference

- `리팩터링 2판 ` 
(https://front.wemakeprice.com/product/822375110?search_keyword=%25EB%25A6%25AC%25ED%258C%25A9%25ED%2586%25A0%25EB%25A7%2581&_service=5&_no=1)