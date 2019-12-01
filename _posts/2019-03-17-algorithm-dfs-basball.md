---
layout: post
comments: true
title: DFS로 숫자 야구게임 정답찾기
tags: [algorithm]
---

### 숫자야구  
다음에서 직접 해볼수 있다. <https://scratch.mit.edu/projects/131352991/>  
  
### JAVA 로 구현하기  
  
`[[123, 1, 1], [356, 1, 0], [327, 2, 0], [489, 0, 1]]`  
[제출값 , strike, ball] 이 다음과같이 주어질때, 정답으로 가능한 숫자들을 제시해주도록 하겠다.    
기본적으로 앞선 도출값을 통해 정답이 될수 있는 수들을 DFS 탐색을 통해 리스트로 찾은뒤 마찬가지로 다음 도출값들에서도 리스트 추출해내어 리스트를 필터링하면서 정답을 찾아나가는 방식으로 진행한다.  

```c
class Solution {

    int questionCount = 0;
    List<Integer> resultList = new ArrayList<>();
    List<Integer> tempList = new ArrayList<>();

    public int solution(int[][] baseball) {
        int answer = 0;
        getList(baseball);
        for(int el : resultList){
        System.out.println(el);
        }
        answer = resultList.size();
        return answer;
    }

    public void getList(int[][] arr){
        for(int[] question : arr){
            int[] num = {question[0]/100, (question[0]/10)%10, question[0]%10};
            int strike = question[1];
            int ball = question[2];

            boolean[] visited = new boolean[9];
            int[] result = new int[3];

            questionCount++;
            dfs(num, visited, result, 0, strike, ball);

            if(questionCount == 1){
                resultList = new ArrayList<Integer>(tempList);
            }else{
               resultList = resultList.stream().filter(e-> tempList.contains(e)).collect(Collectors.toList());
            }
            tempList.clear();
        }
    }

    public void dfs(int[] num, boolean[] visited, int[] result,int pivot, int strike, int ball){

        for(int i=0; i<9; i++){
            if(pivot == 3){
                int tempStrike=0 , tempBall=0;
                for(int j=0; j<3 ;j++){
                    for(int k=0; k<3; k++){
                        if(result[k]==num[j] ){
                            if(k==j)  tempStrike ++;
                            else    tempBall++;
                            break;
                        }
                    }
                }
                if(tempStrike == strike && tempBall == ball){
                    String resultStr = "";
                    for(int el : result){
                        resultStr += Integer.toString(el);
                    }   
                    tempList.add(Integer.parseInt(resultStr));

                }
                break;
            }

            if(!visited[i]){
                visited[i] = true;
                result[pivot] = i+1;
                dfs(num, visited, result, pivot+1, strike, ball);
                visited[i] = false;
            }
        }
    }

}
```

