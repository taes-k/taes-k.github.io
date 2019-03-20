---
layout: default
comments: true
title: 순열
parent: algorithm
date: 2019.03.17
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### 순열 (Permutation)
수학에서, 순서가 부여된 임의의 집합을 다른 순서로 뒤섞는 연산으로, 서로다른 n개의 값에서 r 개를 골라 나열하는것이다. 
학창시절 nPr으로 나타내던 식으로 갯수 공식은 다음과 같다.
```c
- nP0 = 1  
- nPr = n! / (n-r)!  
- nPn = n!
```
### JAVA 로 구현하기
임의의 arr 에서 순열들을 뽑아 result arr에 담는것 까지 JAVA - DFS 깊이우선탐색 으로 구현해보도록 하겠다.  
먼저, 깊이 우선 탐색의 기본은 구하고자 하는 모든 노드를 한번씩 방문후, 다시 밑에서 올라가는 backtracking 을 통해 탐색을 보장하는데 이를 순열을 뽑아내는데 이용하고자 한다.   

```c
Class Permutation{
    
    List<String> result = new ArrayList<>();
    int[] permResult = new int[r];
    boolean[] visit = new boolean[r];
    int[] arr = {1,2,3};
    int pivot = 0;
    int n = arr.length;
    int r = 3;
    
    
    public void permutation(int[] arr, int pivot, int n, int r){
        for(int i = 0; i<n; i++){
            if(pivot == r){ //r개의 노드를 모두 찾음
                String perm = "";
                for(int el : permResult){
                    perm += el;
                }
                result.add(perm);
                break;
            }
            if(!visit[i]){ //방문하지 않았던 노드
                visit[i] = true;
                permResult[pivot] = arr[i]; // 방문기록 남김
                permutations(arr,pivot+1,n,r);
                visit[i] = false; // 완료후 미방문 상태로 복귀
            }
        }
    }

}
```

위의 소스의 탐색 순서를 살펴보면   다음과 같다.

```
1   ->  1           (X)
1   ->  2   ->  1   (X)
1   ->  2   ->  2   (X)
1   ->  2   ->  3   ->  (완료)
1   ->  3   ->  1   (X)
1   ->  3   ->  2   ->  (완료)
1   ->  3   ->  3   (X) 
2   ->  1   ->  1   (X) 
2   ->  1   ->  2   (X) 
2   ->  1   ->  3   ->  (완료)    
2   ->  2           (X) 
2   ->  3   ->  1   ->  (완료)
2   ->  3   ->  2   (X) 
2   ->  3   ->  3   (X) 
3   ->  1   ->  1   (X) 
3   ->  1   ->  2   ->  (완료)
3   ->  1   ->  3   (X) 
3   ->  2   ->  1   ->  (완료)
3   ->  2   ->  2   (X) 
3   ->  2   ->  3   (X) 
```

다음과같은 3P3 에서는 3! = 6으로, 6개의 순열이 출력되었다.
