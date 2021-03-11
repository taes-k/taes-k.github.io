---
layout: post
comments: true
title: 어떻게 리팩터링을 해야할까 - 테스트
tags: [refactoring]
---

### How do you refactorig? - 테스트

이전 포스팅에서 리팩터링을 '`소프트웨어의 겉보기 동작은 그대로 유지한 채` 코드를 이해하고 수정하기 쉽도록 내부 구조를 변경하는 기법' 라고 정의 했습니다.  

`소프트웨어의 겉보기 동작은 그대로 유지한 채` 라는 뜻은 리팩토링 이전과 이후의 결과는 동일해야한다는 뜻입니다. 우리가 리팩토링 해야하는 코드가 테스트가 갖춰져 있지 않은 코드일 경우에 가장 먼저 해야할 일은 테스트 코드를 작성 하는 일 입니다.

테스트 코드를 작성하는 시점을 물어보신다면, 테스트를 작성하기 가장 좋은 시점은 프로그래밍을 시작하기 전 입니다. 즉 이제는 너무나도 주요한 개발 기법이 된 `테스트 주도개발 (TDD)`을 이야기하는 것 입니다.

리팩터링은 수시로 진행해야한다고 말씀드렸던 것 처럼, `TDD`기법의 작업방법은 `'테스트 -> 개발 -> 리팩토링'` 의 반복 입니다. 테스트를 만듬으로써 언제든 리팩토링 할 수 있는 코드를 만들어 주는것이 핵심입니다.

### How do I Test?

아래 어플리케이션을 예제로 테스트 코드를 작성 해 보도록 하겠습니다.

![1]({{ site.images | relative_url }}/posts/2021-03-06-how-refactoring-test/1.jpeg) 

테스트는 모든 메서드에 대해서 진행하는것 보다는 위험 요인을 중심으로 작성하는것이 좋습니다.  
테스트이 목적은 `테스트 작성`으로 끝나는것이 아니고 어디까지나 현재 혹은 향후에 발생하는 버그를 찾기 위함이기 때문에 위험요인을 중심으로 작성해야 합니다. 

너무 단순하여 버그가 숨어들 가능성이 거의 없는 코드들 (단순히 필드 읽기/쓰기를 하는 등의 단순한 접근자 등의 코드 등)에 대해서는 테스트 할 필요 없습니다.

예제 어플리케이션을 통해 테스트시 주의해야할 사항을 몇가지 알아보도록 하겠습니다.  

**테스트 조건은 각 테스트끼리 상호작용되서는 안된다**

위 어플리케이션의 기능중 총 수익 계산 로직 테스트코드를 짠다면 아래와 같을것입니다.

```js
describe('province', function()
{
    // 부족분 테스트
    it('shortfall', function()
    {
        const asia = new Province(sampleProvinceData());
        expect(asia.shortfall).equal(5);
    });
    // 총수익 테스트
    it('profit', function()
    {
        const asia = new Province(sampleProvinceData());
        expect(asia.profit).equal(230);
    });
}
```

테스트 코드를 작성할때에는 아래와 같이 테스트끼리 상호작용되게끔 해서는 안됩니다.

```js
describe('province', function()
{
    const asia = new Province(sampleProvinceData());
    // 부족분 테스트
    it('shortfall', function()
    {
        expect(asia.shortfall).equal(5);
    });
    // 총수익 테스트
    it('profit', function()
    {
        expect(asia.profit).equal(230);
    });
}
```

위코드의 픽스쳐는 각각의 테스트 시작시 초기화 되어져야 합니다.
```js
describe('province', function()
{
    let asia;
    beforeEach(function()
    {
        asia = new Province(sampleProvinceData());
    });
    // 부족분 테스트
    it('shortfall', function()
    {
        expect(asia.shortfall).equal(5);
    });
    // 총수익 테스트
    it('profit', function()
    {
        expect(asia.profit).equal(230);
    });
}
```
**하나의 테스트 여러개를 검증하지 말자**

```js
    it('change production', function()
    {
        asia.producers[0].production = 20;
        expect(asia.shortfall).equal(-6);
        expect(asia.profit).equal(230);
    });
```
만약 첫번째 검증에서 실패하면 다음 검증은 실해도 못해보고 테스트가 실패하게 됩니다.  
가능한, 하나의 테스트 에서는 하나의 검증만을 수행하는것이 좋습니다.

**경계조건을 검사하자**

일반적으로 테스트는 `성공하는 테스트`를 만들기 때문에 의도대로 일반적인 조건으로 테스트를 수행합니다. 하지만 이 일반적인 조건의 범위를 벗어나는 경계지점의 조건들을 테스트에 추가해주면 예외사항에 대한 처리를 빠르게 파악 할 수 있습니다.

```js
    it('zero demand', function()
    {
        //수요가 0일때
        asia.demand = 0;
        expect(asia.shortfall).equal(-25);
        expect(asia.profit).equal(0);
    });

    it('negative demand', function()
    {
        //수요가 -일때
        asia.demand = -1;
        expect(asia.shortfall).equal(-26);
        expect(asia.profit).equal(-10);
    });

    it('empty demand', function()
    {
        //수요가 비어있을때
        asia.demand = "";
        expect(asia.shortfall).NaN;
        expect(asia.profit).NaN;
    });
```

만약 결과가 에러가 나올것을 예상하는 경우에는 에러 상황을 expect하여 테스트가 성공하도록 만들어 주어야 합니다.

위와같은 테스트를 모두 하려고하면 사실 끝도 없을것입니다. 그래서 어차피 모든 버그를 잡을 수 없을테니 테스트를 작성하지 않는다면 대다수의 버그를 잡을 수 있는 기회를 놓치는 일이 될 것입니다.

처음부터 완벽한 테스트는 없습니다. 리팩터를 하면서 테스트를 작성하고, 버그를 발견하거나 리포팅을 받으면, 가장먼저 할 일은 해당 버그를 만드는 테스트를 작성 하는것입니다.

---

### Reference

- `리팩터링 2판 ` 
(https://front.wemakeprice.com/product/822375110?search_keyword=%25EB%25A6%25AC%25ED%258C%25A9%25ED%2586%25A0%25EB%25A7%2581&_service=5&_no=1)