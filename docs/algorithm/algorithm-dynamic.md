---
layout: default
comments: true
title: dynamic programming
parent: algorithm
date: 2019.03.19
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### Dynamic programming
주로 특정 범위 혹은 조건까지의 값을 구할때, 값을 효율적으로 구하기 위한 알고리즘이다.    
답을 저장시키며 재활용하여 다음 해를 구하는데 사용함으로써, 속도가 빠르다는 장점이있다. 동적프로그래밍을 사용하는데 유명한 예제로써 설명하도록하겠다.

### 피보나치 수열 계산
`1 1 2 3 5 8 ... `  
점화식으로 f(n) = f(n-1)+f(n-2)를 갖는 유명한 피보나치 수열을 DP 로 풀어보도록하자.   
f(N)을 구한다고 하면 다음과 같은 표를 가질수 있다.

|1  |2  |3  |4  |5  |
| :---: |:---: |:---: |:---: |
|1  |1  |2  |3  |5  |


```c
int[] result;
public void fibo(){
    
    result[1] = 1;
    result[2] = 1;
    
    for(int i=3; i<=N; i++){
        result[i] = result[i-1][i-2];
    }
    
}
```

다음과같이 이전의 해값들의 먼저 구해 원하는 해를 구하는데 이용할 수 있다.   
다음예제로, 조금더 복잡한문제로 DP에대하여 살펴보자. 

### 배낭문제
이또한 DP를 사용하는 유명한 문제로써 가치와 무게가다른 물건을 한정된 배낭에 넣는다고 할때, 가장 큰 가치를 지니는 방법에 대해 구하는 문제이다.  

예시로, 다음과 같은 물건들이 있을때 무게 5의 배낭에 넣으려고 할때를 가정해보자.

|무게  |금액  |
| :---: |:---: |
|1  |5  |
|2  |7  |
|3  |8  |
|4  |9  |

제한 무게인 5가 있기때문에 하나의 물건을 선택하는데 여러 제약조건이 생길것이다. 여기서 DP로써 생각을 해보면 물건이 하나없을때, 혹은 제한무게가 하나더 작을때와 비교하여 최적해를 찾아내면 된다. 이를 표로 설명해보면 다음과 같다.

|무게  |금액  |   0     |1      |2     |3       |4      |5      |    
| :---: |
|1      |5      |   0     |5      |5      |5      |5      |5      |
|2      |7      |   0     |5      |7      |12    |12    |12    |
|3      |8      |   0     |5      |7      |12    |13    |15    |
|4      |9      |   0     |5      |7      |12    |13    |15    |

다음과같은 DP 표를 이용해 원하는 해를 구해낼수 있다. 이 테이블에서 점화식을 도출해보면 다음과 같다.  
`result[i][j] = Max(result[i-1][j], result[i-1][j-merchant[i].weight]+merchant[i].price)`


```c

int[][] result;
int[][] merchant = [[1,5],[2,7],[3,8],[4,9]];
public void backpack(){


    for(int i=1; i<=merchant.length; i++){
        for(int j=1; j<=N; j++){
            if(merchant[i][0]>j){
                result[i][j] = result[i-1][j];
            }else{
                result[i][j] = Max(result[i-1][j], result[i-1][j-merchant[i].weight]+merchant[i].price);
            }
        }
    }

}
```
다음과같이 이전의 해값들의 먼저 구해 원하는 해를 구하는데 이용할 수 있다.   
