---
# author: cjlee
categories: problem-solving
# cover: /assets/covers/coding.png
date: "2020-10-11T23:45:00Z"
tags: ['programmers']
title: '# 프로그래머스[Lv.3] 야근지수 ( Java )'
---

[문제 링크](https://programmers.co.kr/learn/courses/30/lessons/12927)

# Problem

**문제 설명**  
회사원 Demi는 가끔은 야근을 하는데요, 야근을 하면 야근 피로도가 쌓입니다. 야근 피로도는 야근을 시작한 시점에서 남은 일의 작업량을 제곱하여 더한 값입니다. Demi는 N시간 동안 야근 피로도를 최소화하도록 일할 겁니다.Demi가 1시간 동안 작업량 1만큼을 처리할 수 있다고 할 때, 퇴근까지 남은 N 시간과 각 일에 대한 작업량 works에 대해 야근 피로도를 최소화한 값을 리턴하는 함수 solution을 완성해주세요.

**제한 사항**  
works는 길이 1 이상, 20,000 이하인 배열입니다.  
works의 원소는 50000 이하인 자연수입니다.  
n은 1,000,000 이하인 자연수입니다.  

**입출력 예**
works|n|result
|:--:|:--:|:--:|
[4, 3, 3]|4|12
[2, 1, 2]|1|6
[1,1]|3|0

**입출력 예 설명**  
**입출력 예 #1**  
n=4 일 때, 남은 일의 작업량이 [4, 3, 3] 이라면 야근 지수를 최소화하기 위해 4시간동안 일을 한 결과는 [2, 2, 2]입니다. 이 때 야근 지수는 22 + 22 + 22 = 12 입니다.

**입출력 예 #2**  
n=1일 때, 남은 일의 작업량이 [2,1,2]라면 야근 지수를 최소화하기 위해 1시간동안 일을 한 결과는 [1,1,2]입니다. 야근지수는 12 + 12 + 22 = 6입니다.

# Solve

## 접근방법
: 문제를 푸는 시간보다 문제를 이해하는데 더 오래 걸렸던 문제.  
문제 설명을 조금 더 친절하게 해줬으면 어떨까 싶은 생각이 들었다.

결국, 여기서 이야기하는 **야근 피로도의 최소화**란,  
**야근이 아닌 근무시간 중**에, 미리 어떤 일거리를 얼마나 해결해 놓아야,  
야근을 시작했을 때 **피로도가 최소화 되는가?**(이 때, 피로도는 각 작업 별 수치의 제곱)
가 핵심이다.

중학교 정규교육과정을 이수한 사람이라면, 제곱의 밑 수가 커지면 커질수록,  
그 결과값이 점점 더 큰 폭으로 상승하는 것을 알고 있을 것이다.  

다시 말해, 밑 수가 **2->3** 으로 갈 때에는 **5(3^2 - 2^2)**만큼 차이나지만,  
**3->4**로 갈 때에는 **7(4^2 - 3^2)**만큼 차이가 난다.   
따라서, 최대한 숫자를 평평하게(?) 만들어 주는것이 핵심이다.

## 구현
위에서 언급한 내용을 구현하기 위해서, 큰 수를 우선으로 하는PriorityQueue를 사용했다.  
```java
PriorityQueue<Integer> PQ = new PriorityQueue<>(Collections.reverseOrder());
```

1시간 당 가능한 작업량이 1이니,  

가장 큰 수를 먼저 꺼내고,  
1을 감소시키고,  
다시 집어넣는다.

이 과정을 **n 만큼** 반복하면 된다.

```java
for (int i = 0; i < n; i++) {
    PQ.offer(PQ.poll()-1);
}
```

그리고, 답을 구할 때에는, PQ 안에 들어있는 모든 값을 다 꺼내서 제곱해서 모으면 된다.  
이 때, **음수인 경우에는 0이어야 한다는 것**을 주의하자.

```java
while(!PQ.isEmpty()) {
    long p = PQ.poll();
    answer += p > 0 ? Math.pow(p, 2) : 0;
}
```

전체 코드는 다음과 같다.


```java
package programmers.lv3;

import java.util.Collections;
import java.util.PriorityQueue;

// 프로그래머스 - 야근 지수
public class p12927 {

    public static void main(String[] args) {
        int n = 4;
        int[] works = {4, 3, 3};
        long result = solution(n, works);
        System.out.println(result);
    }

    public static long solution(int n, int[] works) {
        long answer = 0;
        PriorityQueue<Integer> PQ = new PriorityQueue<>(Collections.reverseOrder());

        for (int i = 0; i < works.length; i++) {
            PQ.offer(works[i]);
        }

        for (int i = 0; i < n; i++) {
            PQ.offer(PQ.poll()-1);
        }

        while(!PQ.isEmpty()) {
            long p = PQ.poll();
            answer += p > 0 ? Math.pow(p, 2) : 0;
        }

        return answer;


    }
}

```