---
# author: cjlee
categories: problem-solving
# cover: /assets/covers/coding.png
date: "2020-10-30T08:01:00Z"
tags: ['programmers']
title: '# 프로그래머스[Lv.2] 더 맵게 ( Java )'
---

[문제 링크](https://programmers.co.kr/learn/courses/30/lessons/42626)

# Problem.

**문제 설명**

매운 것을 좋아하는 Leo는 모든 음식의 스코빌 지수를 K 이상으로 만들고 싶습니다. 모든 음식의 스코빌 지수를 K 이상으로 만들기 위해 Leo는 스코빌 지수가 가장 낮은 두 개의 음식을 아래와 같이 특별한 방법으로 섞어 새로운 음식을 만듭니다.

섞은 음식의 스코빌 지수 = 가장 맵지 않은 음식의 스코빌 지수 + (두 번째로 맵지 않은 음식의 스코빌 지수 * 2)

Leo는 모든 음식의 스코빌 지수가 K 이상이 될 때까지 반복하여 섞습니다.  
Leo가 가진 음식의 스코빌 지수를 담은 배열 scoville과 원하는 스코빌 지수 K가 주어질 때, 모든 음식의 스코빌 지수를 K 이상으로 만들기 위해 섞어야 하는 최소 횟수를 return 하도록 solution 함수를 작성해주세요.

**제한 사항**  
- scoville의 길이는 2 이상 1,000,000 이하입니다.
- K는 0 이상 1,000,000,000 이하입니다.
- scoville의 원소는 각각 0 이상 1,000,000 이하입니다.
- 모든 음식의 스코빌 지수를 K 이상으로 만들 수 없는 경우에는 -1을 return 합니다.

**입출력 예**

scoville|K|return
|:--:|:--:|:--:|
[1, 2, 3, 9, 10, 12]|7|2

**입출력 예 설명**
1. 스코빌 지수가 1인 음식과 2인 음식을 섞으면 음식의 스코빌 지수가 아래와 같이 됩니다.    
   새로운 음식의 스코빌 지수 = 1 + (2 * 2) = 5  
   가진 음식의 스코빌 지수 = [5, 3, 9, 10, 12]

2. 스코빌 지수가 3인 음식과 5인 음식을 섞으면 음식의 스코빌 지수가 아래와 같이 됩니다.    
   새로운 음식의 스코빌 지수 = 3 + (5 * 2) = 13  
   가진 음식의 스코빌 지수 = [13, 9, 10, 12]  

모든 음식의 스코빌 지수가 7 이상이 되었고 이때 섞은 횟수는 2회입니다.

# Solve
: 지난 번에 풀었던 [야근 지수 문제](https://cjlee38.github.io/problem-solving/problem_solving_13)와 살짝 흡사하다.

## 접근방법.
문제의 핵심은, **가장 작은 수(음식)을 두 개 꺼내서, 이를 계산 후 다시 집어넣는 것**이다.  
음식이 들어있는 array를 내림차순 정렬해서,   
가장 뒤에서 두개를 꺼냈다가 다시 집어넣는 방법도 있지만,   

**우선순위 큐**라는, 여기에 아주 걸맞는 자료구조가 있으므로,   
이용하지 않을 이유가 없다.

## 구현.

연습도 할 겸, 겉멋으로 스트림을 이용했다.

```java
PriorityQueue<Integer> foods 
         = Arrays.stream(scoville)
               .boxed()
               .collect(Collectors.toCollection(PriorityQueue::new));
```

현재 `foods` 라는 우선순위 큐는 오름차순 정렬되어있으므로,  
하나씩 꺼낼때에는, 가장 낮은 숫자가 튀어나오게 된다.

첫 번째 꺼내는 녀석은 그대로,   
두 번째 꺼내는 녀석은 2를 곱한 뒤,   
다시 집어넣어주면 되는데

이 때 주의할 점은, 매 반복마다 2개의 음식을 꺼내므로,  
foods의 크기가 2 이상이어야 한다는 것이다.

```java
while (foods.size() > 1 && foods.peek() < K) {
   foods.offer(foods.poll() + foods.poll() * 2);
   count++;
}
```

그리고 마지막으로,  
**모두 잘 섞여서 반복이 중단된건지,**  
**아니면 K 이상으로 만들 수 없어서 중단된건지,**  

를 확인하기 위해, peek()한 값이 K보다 큰지 확인하고,  
맞으면 count를, 아니라면 -1을 return하면 된다.

```java
return foods.peek() >= K ? count : -1;
```

전체 코드는 다음과 같다.

```java
package programmers.lv2;

import java.util.Arrays;
import java.util.PriorityQueue;
import java.util.stream.Collectors;

//프로그래머스 - 더 맵게
public class p42626 {
    public static void main(String[] args) {
        int[] socville = {1, 5};
        int K = 7;
        int result = solution(socville, K);
        System.out.println(result);
    }

    public static int solution(int[] scoville, int K) {
        int count = 0;

        PriorityQueue<Integer> foods = Arrays.stream(scoville)
                .boxed()
                .collect(Collectors.toCollection(PriorityQueue::new));
        
        while (foods.size() > 1 && foods.peek() < K) {
            foods.offer(foods.poll() + foods.poll() * 2);
            count++;
        }

        return foods.peek() >= K ? count : -1;
    }
}
```