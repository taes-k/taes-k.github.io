---
layout: post
comments: true
title: 실전 리팩터링 예제
tags: [refactoring]
---

### 리팩터링 초기코드

이전 포스팅들을 통해 리팩터링의 `5W1H`를 알아보았습니다.  
이제 최종적으로 Java 예제코드를 통해 리팩터링을 진행해 보겠습니다.  

예제코드는 공연료 청구서를 출력하는 코드입니다.  
(해당 예제는 `Refactoring 2판`에서 제공하는 JS 예제를 Java로 구현한 코드입니다.)  

```java
// Play.java

@AllArgsConstructor
@Getter
@Setter
public class Play
{
    private String name;
    private String type;
}
```

```java
// Performance.java

@AllArgsConstructor
@Getter
@Setter
public class Performance
{
    private String playId;
    private int audience;
}
```

```java
// Invoice.java

@Getter
@Setter
public class Invoice
{
    private String customer;
    private List<Performance> performances;
}
```

```java
// Statement.java

public class Statement
{
    public void printCharge(Invoice invoice, Map<String, Play> plays)
    {
        int totalAmount = 0;
        int volumeCredits = 0;
        StringBuilder result = new StringBuilder();
        result.append(String.format("청구 내역 (고객명 : %s)\n", invoice.getCustomer()));

        for(Performance perf : invoice.getPerformances())
        {
            Play play = plays.get(perf.getPlayId());
            int thisAmount = 0;

            switch (play.getType())
            {
                case "tragedy": //비극
                    thisAmount = 40000;
                    if (perf.getAudience() > 30)
                    {
                        thisAmount += 1000 * (perf.getAudience() - 30);
                    }
                    break;
                case "comedy": //희극
                    thisAmount = 30000;
                    if (perf.getAudience() > 20)
                    {
                        thisAmount += 10000 + 500 * (perf.getAudience() - 20);
                    }
                    thisAmount += 300 * perf.getAudience();
                    break;
                default:
                    String errorMessage = String.format("알수없는 장르 : %s", play.getType());
                    throw new RuntimeException(errorMessage);
            }

            // 포인트적립
            volumeCredits += Math.max(perf.getAudience() - 30, 0);

            // 희극 관객 5명마다 추가 포인트적립
            if (play.getType().equals("comedy"))
            {
                volumeCredits += Math.floor(perf.getAudience() / 5);
            }

            // 청구 내역 출력
            result.append(String.format("%s: %d (%d석)\n", play.getName(), thisAmount, perf.getAudience()));
            totalAmount += thisAmount;
        }

        result.append(String.format("총액 : %d\n", totalAmount));
        result.append(String.format("적립 포인트 : %d점\n", volumeCredits));

        System.out.println(result);
    }
}
```

```java
// Main.java

public class Main
{
    public static void main(String[] args)
    {
        Map<String, Play> plays = new HashMap<>();
        plays.put("hamlet", new Play("Hamlet", "tragedy"));
        plays.put("as-like", new Play("As You Like It", "comedy"));
        plays.put("othello", new Play("Othello", "tragedy"));

        Invoice invoice = new Invoice();
        invoice.setCustomer("BigCo");
        invoice.setPerformances(
            Arrays.asList(
                new Performance("hamlet", 55)
                , new Performance("as-like", 35)
                , new Performance("othello", 40)));

        Statement statement = new Statement();
        statement.printCharge(invoice, plays);
    }
}

```

[> 1. 초기코드 repository](https://github.com/taes-k/refactoring-sample/tree/feature/initial-code)

![1]({{ site.images | relative_url }}/posts/2021-04-28-refactoring-example/1.png)   

위 초기코드로도 결과가 문제 없이 좋은 성능으로 잘 나오고 있는것을 보실 수 있습니다. 그렇지만 리팩터링의 목적은 성능개선이 아닙니다.

> `리팩터링의 목적은 프로그램의 성능을 높이는것이 아닌 코드를 이해하고 수정하게 쉽게 만드는 것 이라는 것을 명심해야합니다. `  
> 
> [\[리팩터링이란 무엇인가?\]](https://taes-k.github.io/1-what-why-refactoring) 

---

### 리팩터링 시작하기

자, 이제 리팩터링을 시작해보도록 하겠습니다.  
코드 자체가 짧아서 어렵지않지만, 좀 더 이해하기 쉽게 만들 수 있을것 같습니다.

> `기능을 추가하기 쉽게만드는 것이 리팩터링의 핵심`
> 
> [\[언제, 어디서 리팩터링을 해야하는가?\]](https://taes-k.github.io/2-where-when-refactoring)

우리는 문자열을 `HTML` 형식으로 출력하는 기능을 추가하기 위해, 기능을 추가하기 쉽도록 리팩터링을 한다는 전제를 가져보도록 하겠습니다.

우선은, `Statement`에 포함되어있는 다양한 기능들을 분리 시키면서 구조를 개선해보면 좋을것 같습니다.  

- 공연료 계산기능
- 포인트 계산 기능

리팩터링 할 포인트도 정했으니, 바로 코드 수정을 시작하면 될까요?

> `우리가 리팩토링 해야하는 코드가 테스트가 갖춰져 있지 않은 코드일 경우에 가장 먼저 해야할 일은 테스트 코드를 작성 하는 일 입니다.`  
> 
> [\[어떻게 리팩터링을 해야할까? - 테스트\]](https://taes-k.github.io/3-how-refactoring-test) 

리팩터링의 첫단계는 항상 `테스트 코드` 작성입니다.  

```java
    @Test
    void printCharge_success()
    {
        // given
        Map<String, Play> plays = new HashMap<>();
        plays.put("hamlet", new Play("Hamlet", "tragedy"));
        plays.put("as-like", new Play("As You Like It", "comedy"));
        plays.put("othello", new Play("Othello", "tragedy"));

        Invoice invoice = new Invoice();
        invoice.setCustomer("BigCo");
        invoice.setPerformances(
            Arrays.asList(
                new Performance("hamlet", 55)
                , new Performance("as-like", 35)
                , new Performance("othello", 40)));
        Statement statement = new Statement();

        // when
        statement.printCharge(invoice, plays);

        // then
        Assertions.assertTrue(true);

    }
```

위 테스트 코드는 정상수행여부를 정확하게 체크를 하지 못하고 있기 때문에 좋은 테스트 코드는 아니지만, 현재 코드가 테스트 검증을 하기 어렵게 짜여진 코드이기때문에 어쩔수 없습니다. 

리팩터링을 하게되면 자연스럽게 `'테스트 하기 쉬워진다'`는 점 또한 리팩토링을 하는 이유중 한가지가 될 수 있습니다. 

이제 진짜 코드 리팩터링 작업을 시작해 보겠습니다.  
처음 할 일은 본문 코드에서 `기능을 추가하기 쉽도록` 분리하는 작업입니다.       

- `amountFor` : 공연비 계산 함수 추출하기 [\[6.1 함수 추출하기\]](https://taes-k.github.io/4-how-refactoring-standard)  

```java
// Statement.java

...

    private int amountFor(Performance performance, Play play)
    {
        int result;
        switch (play.getType())
        {
            case "tragedy": //비극
                result = 40000;
                if (performance.getAudience() > 30)
                {
                    result += 1000 * (perf.getAudience() - 30);
                }
                break;

                ...

        }

        return result;
    }
```

- `play` : 변하지 않는 공연정보(지역변수)들을 질의 함수로 바꾸기 [\[7.4 임시변수를 질의함수로 바꾸기\]](https://taes-k.github.io/4-how-refactoring-standard)

```java
// Statement.java


public class Statement
{
    private final Map<String, Play> plays;

    public Statement(Map<String, Play> plays)
    {
        this.plays = plays;
    }


    public void printCharge(Invoice invoice)
    {
        ...
    }


    private int amountFor(Performance performance)
    {
        ...
    }

    private Play playFor(Performance performance)
    {
        return plays.get(performance.getPlayId());
    }
}

```

> 각 단계를 진행할때마다 테스트를 수행하는것을 잊으시면 안됩니다!  
> 여기까지 작업하셨다면 당연하게도 테스트에서 에러가 발생해 테스트가 실패하고 있을 겁니다.  
> 자연스럽게 테스트코드도 함께 수정해가며 리팩터링하시면 됩니다.  각 단계를 진행할때마다

지역변수를 제거함으로서, 유효범위를 신경써야할 대상이 줄어들기 때문에 추출작업이 훨씬 쉬워질 수 있습니다. 

또하나의 남은 지역변수도 제거해보도록 합시다.

- `thisAmount` : 변하지 않는 가격정보(지역변수)를 인라인으로 바꾸기 [\[6.4 변수 인라인하기\]](https://taes-k.github.io/4-how-refactoring-standard)  


```java
// Statement.java

public void printCharge(Invoice invoice)
    {
     
        ...

        for(Performance perf : invoice.getPerformances())
        {
            ...
            
            // 청구 내역 출력
            result.append(String.format("%s: %d (%d석)\n", playFor(perf).getName(), amountFor(perf), perf.getAudience()));
            totalAmount += amountFor(perf);
        }

        ...
    }
```

아직 추출 할수있는 기능이 더 남았습니다.

- `volumeCreditsFor` : 포인트 적립 함수 추출하기 [\[6.1 함수 추출하기\]](https://taes-k.github.io/4-how-refactoring-standard)  

```java
// Statement.java

    private int volumeCreditsFor(Performance perf)
    {
        int volumeCredits = 0;
        volumeCredits += Math.max(perf.getAudience() - 30, 0);

        // 희극 관객 5명마다 추가 포인트적립
        if (playFor(perf).getType().equals("comedy"))
        {
            volumeCredits += Math.floor(perf.getAudience() / 5);
        }
        
        return volumeCredits;
    }
```

- `toUSD` : 금액 변환 함수 추출 및 이름바꾸기 [\[6.1 함수 추출하기\]\[6.5 함수선언 바꾸기\]](https://taes-k.github.io/4-how-refactoring-standard)  

```java
// Statement.java

    private String toUSD(int number)
    {
        NumberFormat nf = NumberFormat.getCurrencyInstance( Locale.US );
        nf.setMaximumFractionDigits(2);
        return nf.format(number/100);
    }
```

이제 `printCharge()`함수를 살펴보면 반복문에서 두가지 일을 하고있는것이 보일겁니다. 이경우 리팩터링하기 더 까다로워지기 때문에 기능별로 반복문을 분리해주도록 하겠습니다.

- `totalAmount, volumeCredit` : 반복문 쪼개기와 사용변수 선언 위치를 변경해줍니다. [\[8.7 반복문 쪼개기\]\[8.6 문장 슬라이드하기\]](https://taes-k.github.io/6-how-refactoring-move-function)  

```java
// Statement.java

    public void printCharge(Invoice invoice)
    {
   
        ...

        int totalAmount = 0;
        for(Performance perf : invoice.getPerformances())
        {
            // 청구 내역 출력
            result.append(String.format("%s: %d (%d석)\n", playFor(perf).getName(), amountFor(perf), perf.getAudience()));
            totalAmount += amountFor(perf);
        }

        int volumeCredits = 0;
        for(Performance perf : invoice.getPerformances())
        {
            volumeCredits += volumeCreditsFor(perf);
        }

        ...
        
    }
```

위처럼 반복문을 쪼개고나니 함수로 추출할부분이 눈에 바로 보여지게 됩니다.


- `totlaVolumeCredit` : 전체 포인트 적립 함수 추출하기 [\[6.1 함수 추출하기\]](https://taes-k.github.io/4-how-refactoring-standard)  

```java
    private int getTotalVolumeCredits(Invoice invoice)
    {
        int volumeCredits = 0;
        for(Performance perf : invoice.getPerformances())
        {
            volumeCredits += volumeCreditsFor(perf);
        }
        return volumeCredits;
    }
```

위에 남은 `totalAmount`도 함수로 추출하고싶으나 청구서 정보를 담는 `result`가 포함되어 기능분리가 안되어있습니다. 위에서 사용한 `반복문 쪼개기`를 다시한번 수행 할 때 입니다.

반복문을 쪼개서 성능이 느려지지 않을까 걱정할수도 있지만 성능에 미치는 영향은 미미한 경우가 많습니다. 물론 항상 그런것은 아니기때문에 고려하는것은 좋으나, 다시한번 말하지만 리팩터링의 목적은 `성능개선`이 아닌 `수정하기 쉬운 코드를 만드는것`입니다.

- `totalAmount, 청구 내역 출력` : 반복문 쪼개기 [\[8.7 반복문 쪼개기\]](https://taes-k.github.io/6-how-refactoring-move-function)  

```java
// Statement.java

    public void printCharge(Invoice invoice)
    {
   
        ...

        for(Performance perf : invoice.getPerformances())
        {
            // 청구 내역 출력
            result.append(String.format("%s: %d (%d석)\n", playFor(perf).getName(), amountFor(perf), perf.getAudience()));
        }

        int totalAmount = 0;
        for(Performance perf : invoice.getPerformances())
        {
            totalAmount += amountFor(perf);
        }

        ...
        
    }

```

이제 함수로 추출할 수 있게 되었습니다.


- `totlaVolumeCredit` : 전체 포인트 적립 함수 추출하기 [\[6.1 함수 추출하기\]](https://taes-k.github.io/4-how-refactoring-standard)  

```java
// Statement.java

    private int getTotalAmount(Invoice invoice)
    {
        int totalAmount = 0;
        for(Performance perf : invoice.getPerformances())
        {
            totalAmount += amountFor(perf);
        }
        return totalAmount;
    }
```

- `totalAmount, volumeCredits` : 단일로 사용되는 변수들을 인라인 해줍니다 [\[6.4 변수 인라인하기\]](https://taes-k.github.io/4-how-refactoring-standard)  

```java
// Statement.java

    public void printCharge(Invoice invoice)
    {
        StringBuilder result = new StringBuilder();
        result.append(String.format("청구 내역 (고객명 : %s)\n", invoice.getCustomer()));

        for(Performance perf : invoice.getPerformances())
        {
            // 청구 내역 출력
            result.append(String.format("%s: %d (%d석)\n", playFor(perf).getName(), amountFor(perf), perf.getAudience()));
        }
        
        result.append(String.format("총액 : %s\n", toUSD(getTotalAmount(invoice))));
        result.append(String.format("적립 포인트 : %d점\n", getTotalVolumeCredits(invoice)));

        System.out.println(result);
    }
```


---

### 기능 추가하기 

위 과정을 통해 리팩터링 된 코드를 중간점검 해보도록 하겠습니다.

```java
// Statement.java

public class Statement
{
    private final Map<String, Play> plays;

    public Statement(Map<String, Play> plays)
    {
        this.plays = plays;
    }

    public void printCharge(Invoice invoice)
    {
        StringBuilder result = new StringBuilder();
        result.append(String.format("청구 내역 (고객명 : %s)\n", invoice.getCustomer()));

        for(Performance perf : invoice.getPerformances())
        {
            // 청구 내역 출력
            result.append(String.format("%s: %d (%d석)\n", playFor(perf).getName(), amountFor(perf), perf.getAudience()));
        }

        result.append(String.format("총액 : %s\n", toUSD(getTotalAmount(invoice))));
        result.append(String.format("적립 포인트 : %d점\n", getTotalVolumeCredits(invoice)));

        System.out.println(result);
    }

    private int getTotalAmount(Invoice invoice)
    {
        int totalAmount = 0;
        for(Performance perf : invoice.getPerformances())
        {
            totalAmount += amountFor(perf);
        }
        return totalAmount;
    }

    private int getTotalVolumeCredits(Invoice invoice)
    {
        int volumeCredits = 0;
        for(Performance perf : invoice.getPerformances())
        {
            volumeCredits += volumeCreditsFor(perf);
        }
        return volumeCredits;
    }

    private String toUSD(int number)
    {
        NumberFormat nf = NumberFormat.getCurrencyInstance( Locale.US );
        nf.setMaximumFractionDigits(2);
        return nf.format(number/100);
    }

    private int volumeCreditsFor(Performance perf)
    {
        int result = 0;
        result += Math.max(perf.getAudience() - 30, 0);

        // 희극 관객 5명마다 추가 포인트적립
        if (playFor(perf).getType().equals("comedy"))
        {
            result += Math.floor(perf.getAudience() / 5);
        }

        return result;
    }

    private int amountFor(Performance performance)
    {
        int result;
        switch (playFor(performance).getType())
        {
            case "tragedy": //비극
                result = 40000;
                if (performance.getAudience() > 30)
                {
                    result += 1000 * (performance.getAudience() - 30);
                }
                break;
            case "comedy": //희극
                result = 30000;
                if (performance.getAudience() > 20)
                {
                    result += 10000 + 500 * (performance.getAudience() - 20);
                }
                result += 300 * performance.getAudience();
                break;
            default:
                String errorMessage = String.format("알수없는 장르 : %s", playFor(performance).getType());
                throw new RuntimeException(errorMessage);
        }
        return result;
    }

    private Play playFor(Performance performance)
    {
        return plays.get(performance.getPlayId());
    }
}

```

[> 2. 기능분리 repository](https://github.com/taes-k/refactoring-sample/tree/feature/make-function)

코드 양 자체는 많아졌지만, 최상위 함수인 `printCharge()`는 짧아졌고 기능별로 함수화가 잘 되어 호출부만 보더라도 어떤 일을 하는 함수인지 잘 알 수 있게 되었습니다.  

앞서서 `HTML` 형식으로 출력하는것을 전제로 했기 때문에 개선된 코드에서 기능을 추가해보도록 하겠습니다. 

현재 `printCharge()` 함수에서 `PlainText`를 출력하고 있기때문에, `HTML`과 `PlainText`를 모두 사용 할 수 있는 구조로 만들어 주는것이 우선입니다.

`출력부`의 기능이 추가되는것이기 때문에, 기존에 계산과 출력을 함께하고있던 `printCharge()`를 분리시켜주는것이 좋을것 같습니다. 이때 계산에 사용될 데이터셋을 별도로 만들어주는것 까지 함께해 보겠습니다.

- `createData, renderPlainText` : [\[6.11 단계 쪼개기\]\[6.8 매개변수 객체만들기\]](https://taes-k.github.io/4-how-refactoring-standard)  

```java
// Statement.java


public class Statement
{
    ...

    @Getter
    @Setter
    private class Data
    {
        private String customer;
        private List<Performance> performances;
        private int totalAmount;
        private int totalVolumeCredits;
    }
    
    ...

    public void printCharge(Invoice invoice)
    {
        Data data = createData(invoice);
        System.out.println(renderPlainText(data));
    }


    private Data createData(Invoice invoice)
    {
        Data result = new Data();

        result.setCustomer(invoice.getCustomer());
        result.setPerformances(invoice.getPerformances());
        result.setTotalAmount(getTotalAmount(invoice));
        result.setTotalVolumeCredits(getTotalVolumeCredits(invoice));

        return result;
    }

    private String renderPlainText(Data data)
    {
        StringBuilder result = new StringBuilder();
        result.append(String.format("청구 내역 (고객명 : %s)\n", data.getCustomer()));

        for(Performance perf :data.getPerformances())
        {
            // 청구 내역 출력
            result.append(String.format("%s: %d (%d석)\n", playFor(perf).getName(), amountFor(perf), perf.getAudience()));
        }

        result.append(String.format("총액 : %s\n", toUSD(data.getTotalAmount())));
        result.append(String.format("적립 포인트 : %d점\n", data.getTotalVolumeCredits()));

        return result.toString();
    }
}

```

위와같이 단계를 분리함으로써, 공통 데이터를 생성하고 원하는 형식으로 출력 할 수 있는 코드 구성이 갖추어졌습니다.  

이제, 여기에 `printHtmlText` 기능을 추가해보도록 하겠습니다.

```java
// Statement.java

    public void printPlainText(Invoice invoice)
    {
        Data data = createData(invoice);
        System.out.println(renderPlainText(data));
    }

    public void printHtmlText(Invoice invoice)
    {
        Data data = createData(invoice);
        System.out.println(renderHtmlText(data));
    }

    ...

    private String renderHtmlText(Data data)
    {
        StringBuilder result = new StringBuilder();
        result.append(String.format("<h1>청구 내역 (고객명 : %s)</h1>\n", data.getCustomer()));
        result.append("<table>");
        result.append("<tr><th>연극</th><th>좌석 수</th><th>금액</th></tr>");

        for(Performance perf :data.getPerformances())
        {
            // 청구 내역 출력
            result.append(String.format("<tr><td>%s</td><td>%d</td><td>%d석</td></tr>\n"
                , playFor(perf).getName(), perf.getAudience(), amountFor(perf)));
        }
        result.append("</table>");

        result.append(String.format("<p>총액 : <em>%s</em></p>\n", toUSD(data.getTotalAmount())));
        result.append(String.format("<p>적립 포인트 : <em>%d</em>점</p>\n", data.getTotalVolumeCredits()));

        return result.toString();
    }
```

public 메서드가 추가됬으니 테스트 코드 또한 추가해줍니다.

```java
 @Test
    void printPlainText_success()
    {
       ...
    }

    @Test
    void printHtmlText_success()
    {
        // given
        Map<String, Play> plays = new HashMap<>();
        plays.put("hamlet", new Play("Hamlet", "tragedy"));
        plays.put("as-like", new Play("As You Like It", "comedy"));
        plays.put("othello", new Play("Othello", "tragedy"));

        Invoice invoice = new Invoice();
        invoice.setCustomer("BigCo");
        invoice.setPerformances(
            Arrays.asList(
                new Performance("hamlet", 55)
                , new Performance("as-like", 35)
                , new Performance("othello", 40)));
        Statement statement = new Statement(plays);

        // when
        statement.printHtmlText(invoice);

        // then
        Assertions.assertTrue(true);
    }

```

[> 3. 신규 기능 추가 repository](https://github.com/taes-k/refactoring-sample/tree/feature/add-new-function)


![2]({{ site.images | relative_url }}/posts/2021-04-28-refactoring-example/2.png)   

전체적으로 코드가 부쩍 늘어났지만, 함수호출을 하면서 늘어난것이기 때문에 안좋은 징조는 아닙니다. 이전 단게에서 기능을 추가하기 쉽도록 리팩터링을 잘 해 두었기에 손쉽게 추가기능을 구현 할 수 있게 되었습니다.


---

### 다형성을 사용한 코드 리팩터링

원하는 기능도 쉽게 추가했지만 아직 좀더 리팩터링 할 수 있는 부분이 남았습니다.  
공연료를 계산하는 부분이 `switch` 로서 조건들을 통해 처리하고 있는데 이부분을 리팩터링 해보도록 하겠습니다.  

우선, 계산함수들을 클래스로 추출해주도록 하겠습니다.

- `PerformanceCalculator` : [\[7.5 클래스 추출하기\]](https://taes-k.github.io/5-how-refactoring-encapsulate) 

```java
// PerformanceCalculator.java

public class PerformanceCaculator
{
    private Performance performance;
    private Play play;

    public PerformanceCaculator(Performance performance, Play play)
    {
        this.performance = performance;
        this.play = play;
    }

    public int getAmount()
    {
        int result;
        switch (play.getType())
        {
            case "tragedy": //비극
                result = 40000;
                if (performance.getAudience() > 30)
                {
                    result += 1000 * (performance.getAudience() - 30);
                }
                break;
            case "comedy": //희극
                result = 30000;
                if (performance.getAudience() > 20)
                {
                    result += 10000 + 500 * (performance.getAudience() - 20);
                }
                result += 300 * performance.getAudience();
                break;
            default:
                String errorMessage = String.format("알수없는 장르 : %s", play.getType());
                throw new RuntimeException(errorMessage);
        }
        return result;
    }

    public int getVolumeCredit()
    {
        int result = 0;
        result += Math.max(performance.getAudience() - 30, 0);

        // 희극 관객 5명마다 추가 포인트적립
        if (play.getType().equals("comedy"))
        {
            result += Math.floor(performance.getAudience() / 5);
        }

        return result;
    }
}

```

```java
// Statement.java

    private int amountFor(Performance performance)
    {
        return new PerformanceCaculator(performance, playFor(performance)).getAmount();
    }

    private int volumeCreditsFor(Performance performance)
    {
        return new PerformanceCaculator(performance, playFor(performance)).getVolumeCredit();
    }

```

금액과 포인트를 계산하는 부분을 별도의 클래스로 추출하고 나니 조금 더 각자의 기능에 집중 할 수 있는 Object 로써 구성이 된 느낌입니다.  

추가로, `PerformanceCaculator`가 분리 되면서, 유닛 테스트를 수행할 메서드들이 자연스럽게 나타났습니다.

```java
// PerformanceCaculatorTest.java


public class PerformanceCaculatorTest
{
    @Test
    void getPlayAmount_success()
    {
        // given
        Play play = new Play("Hamlet", "tragedy");
        Performance performance = new Performance("hamlet", 55);

        PerformanceCaculator performanceCaculator = new PerformanceCaculator(performance, play);

        // when
        int amount = performanceCaculator.getAmount();

        // then
        Assertions.assertEquals(65000, amount);
    }

    @Test
    void getPlayVolumeCredit_success()
    {
        // given
        Play play = new Play("Hamlet", "tragedy");
        Performance performance = new Performance("hamlet", 55);

        PerformanceCaculator performanceCaculator = new PerformanceCaculator(performance, play);

        // when
        int amount = performanceCaculator.getVolumeCredit();

        // then
        Assertions.assertEquals(25, amount);
    }
}
```

유닛 테스트 코드도 잘 작성 됬지만 아직 조금 아쉬운 부분이 있습니다. `type` 값을 기준으로 분기해 계산처리하는 부분을 `다형성`을 이용한다면 좀더 객체지향 적인 코드로 리팩터링 할 수 있을것 같습니다.  

- `PerformanceCaculator` : type 마다 처리 하는 별도의 다형성 클래스를 만든다  [\[10.4 조건부 로직을 다형성으로 바꾸기\]](https://taes-k.github.io/8-how-refactoring-data-simplify-conditional-logic)  
- static 팩터리 생성자를 통해 타입에 맞는 객체를 선언 받을수 있도록 합니다. [\[11.8 생성자를 팩터리 함수로 바꾸기\]](https://taes-k.github.io/9-how-refactoring-data-api)  



```java
// PerformanceCaculator.java

public abstract class PerformanceCaculator
{
    protected Performance performance;
    protected Play play;

    protected PerformanceCaculator(Performance performance, Play play)
    {
        this.performance = performance;
        this.play = play;
    }

    public static PerformanceCaculator createPerformanceCalculator(Performance performance, Play play)
    {
        switch (play.getType())
        {
            case "tragedy" : return new TragedyPerformanceCaculator(performance, play);
            case "comedy" : return new ComedyPerformanceCaculator(performance, play);
            default :
                String errorMessage = String.format("알수없는 장르 : %s", play.getType());
                throw new RuntimeException(errorMessage);
        }
    }

    public abstract int getAmount();

    public abstract int getVolumeCredit();
}

```


```java
// TragedyPerformanceCaculator.java

public class TragedyPerformanceCaculator extends  PerformanceCaculator
{
    public TragedyPerformanceCaculator(Performance performance, Play play)
    {
        super(performance, play);
    }

    public int getAmount()
    {
        int result;
        result = 40000;
        if (performance.getAudience() > 30)
        {
            result += 1000 * (performance.getAudience() - 30);
        }
        return result;
    }

    public int getVolumeCredit()
    {
        int result = 0;
        result += Math.max(performance.getAudience() - 30, 0);

        return result;
    }
}

```

```java
// ComedyPerformanceCaculator.java
public class ComedyPerformanceCaculator extends  PerformanceCaculator
{
    public ComedyPerformanceCaculator(Performance performance, Play play)
    {
        super(performance, play);
    }

    public int getAmount()
    {
        int result;
        result = 30000;
        if (performance.getAudience() > 20)
        {
            result += 10000 + 500 * (performance.getAudience() - 20);
        }
        result += 300 * performance.getAudience();
        return result;
    }

    public int getVolumeCredit()
    {
        int result = 0;
        result += Math.max(performance.getAudience() - 30, 0);
        // 희극 관객 5명마다 추가 포인트적립
        result += Math.floor(performance.getAudience() / 5);

        return result;
    }
}
```

상속으로 구현하면서 각각의 자식 클래스에서 메서드를 구현하도록 `abstract method`처리를 해주었지만, 어느정도 일치하는 부분이 있다면 구현을 해두어도 좋습니다.

```java
// PerformanceCaculator.java

public abstract class PerformanceCaculator
{
    ...

    public int getVolumeCredit()
    {
        return Math.max(this.performance.getAudience() - 30 , 0);
    }
}
```
```java
// TragedyPerformanceCaculator.java

public class TragedyPerformanceCaculator extends  PerformanceCaculator
{
    ...

    // 제거해도 무방
    public int getVolumeCredit()
    {
        return super.getVolumeCredit();
    }
}
```
```java
// ComedyPerformanceCaculator.java

public class ComedyPerformanceCaculator extends  PerformanceCaculator
{
    ...
    
    public int getVolumeCredit()
    {
        // 희극 관객 5명마다 추가 포인트적립
        int additionalCredit = (int) Math.floor(performance.getAudience() / 5);
        return super.getVolumeCredit() + additionalCredit;
    }
}
```

이번 리팩터링으로 클래스가 많이 생겨났지만, 기능이 클래스로 분리 되면서 앞으로 `새로운 장르가 추가 되기 쉬운 구조`가 되었습니다. 앞으로 수정이 많이 일어날 수 있는 기능들은 별도의 클래스로 구성 해 두는것이 유리합니다.  

--- 

### 최종 코드  
[> 4. 클래스분리 repository](https://github.com/taes-k/refactoring-sample/tree/feature/class-polymophism)

사실 더 리팩터링 할 수 있는 사항들은 있지만 리팩터링 예제는 여기서 멈추도록 하겠습니다. 코드양이 많아지면서 코드 자체는 복잡해졌다고 볼 수도 있지만, 코드가 하는 일들은 명확해졌습니다.  

> 좋은 코드를 가늠하는 확실한 방법은 `얼마나 수정하기 쉬운가` 이다.

`'개선점을 찾고 > 리팩터링 하고 > 이해하기 쉬워지고 > 개선점이 드러난다'` 의 선순환이 계속되면서 점점 좋은 코드를 향한 리팩터링의 모험은 계속 될 수 있을것입니다.