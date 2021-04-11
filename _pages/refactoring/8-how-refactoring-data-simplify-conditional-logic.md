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


---

### Reference

- `리팩터링 2판 ` 
(https://front.wemakeprice.com/product/822375110?search_keyword=%25EB%25A6%25AC%25ED%258C%25A9%25ED%2586%25A0%25EB%25A7%2581&_service=5&_no=1)