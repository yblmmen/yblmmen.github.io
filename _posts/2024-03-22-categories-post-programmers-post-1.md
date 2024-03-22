---
title: "[Coding Test] 프로그래머스 Lv.2 - 카펫 "
excerpt: "bruteforce를 이용한 프로그래머스 Lv.2 카펫 문제 풀기"

categories:
  - Programmers
tags:
  - [algorithm, codingtest]

permalink: /programmers/post-programmers-post-1/

toc: true
toc_sticky: true

date: 2024-03-22
last_modified_at: 2024-03-22
---

## I. 문제 설명
Leo는 카펫을 사러 갔다가 아래 그림과 같이 중앙에는 노란색으로 칠해져 있고 테두리 1줄은 갈색으로 칠해져 있는 격자 모양 카펫을 봤습니다.
![output](/assets/images/posts_img/programmers-cate/carpet.png)  
Leo는 집으로 돌아와서 아까 본 카펫의 노란색과 갈색으로 색칠된 격자의 개수는 기억했지만, 전체 카펫의 크기는 기억하지 못했습니다.

Leo가 본 카펫에서 갈색 격자의 수 brown, 노란색 격자의 수 yellow가 매개변수로 주어질 때 카펫의 가로, 세로 크기를 순서대로 배열에 담아 return 하도록 solution 함수를 작성해주세요.  
>### 제한사항  
> * 갈색 격자의 수 brown은 8 이상 5,000 이하인 자연수입니다.
> * 노란색 격자의 수 yellow는 1 이상 2,000,000 이하인 자연수입니다.
> * 카펫의 가로 길이는 세로 길이와 같거나, 세로 길이보다 깁니다.
  
  
>### 입출력 ex)
> ![output](/assets/images/posts_img/programmers-cate/carper1.png)



## II. Solution 
>1. 중앙에 노란색으로 칠해져 있고 노란색 주위로 테두리 1줄은 갈색이다. 
* yellow가 존재하려면 가로 세로의 길이가 최소 3 이상이어야 한다.
  (가로가 세로와 같거나 길어야 된다는 조건이 있고 가로가 2 세로가 2 라면 yellow를 감쌀 수 없기 때문에 최소 3 이상이어야 한다.)
>2. 카펫의 넓이는 brown 격자 수 +yellow 격자 수 
>3. 갈색 테두리를 제외 하고 가로*세로를 하면 yellow 넓이가 나온다.
* yellow=(가로-2)*(세로-2)
>4. 만약에 카펫의 넓이가 12라고 하면 가로 세로는 (1,12),(2,6),(3,4),(4,3),(6,2),(12,1)이 나올 수 있다.
>5. 그러나 가로의 길이가 더 길다는 조건과 가로 세로가 최소 3 이상이어야 한다는 조건을 적용하면 (4,3)만 가능하다.

## III. 최종코드
```
class Solution {
    public int[] solution(int brown, int yellow) {
        int[] answer = new int[2];
        int area = brown + yellow;
        
         for (int row = 3; row < area; row++){//가로 세로 최소 3 이상
            int col = area/row; // 세로는 넓이/가로
            
            if (col >= 3 && area % row == 0){
                if(row>col){
                    break; //가로가 세로보다 길어야 함
                }
                if((row-2)*(col-2)==yellow){
                    answer[0]=col;
                    answer[1]=row;
                    return answer;
                }
            }
        }
        
        return answer;
    }
}
```

  



## IV. 참조
https://school.programmers.co.kr/learn/courses/30/lessons/42842


