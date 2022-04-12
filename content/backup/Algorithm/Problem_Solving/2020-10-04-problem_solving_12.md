---
# author: cjlee
categories: problem-solving
# cover: /assets/covers/coding.png
date: "2020-10-04T22:11:00Z"
tags: ['programmers']
title: '# 프로그래머스[Lv.3] 단속카메라 ( Java )'
---

# Problem

**문제 설명**  

고속도로를 이동하는 모든 차량이 고속도로를 이용하면서 단속용 카메라를 한 번은 만나도록 카메라를 설치하려고 합니다.  

고속도로를 이동하는 차량의 경로 routes가 매개변수로 주어질 때, 모든 차량이 한 번은 단속용 카메라를 만나도록 하려면 최소 몇 대의 카메라를 설치해야 하는지를 return 하도록 solution 함수를 완성하세요.  

**제한사항**  

* 차량의 대수는 1대 이상 10,000대 이하입니다.  
* routes에는 차량의 이동 경로가 포함되어 있으며 routes[i][0]에는 i번째 차량이 고속도로에 진입한 지점, routes[i][1]에는 i번째 차량이 고속도로에서 나간 지점이 적혀 있습니다.  
* 차량의 진입/진출 지점에 카메라가 설치되어 있어도 카메라를 만난것으로 간주합니다.  
* 차량의 진입 지점, 진출 지점은 -30,000 이상 30,000 이하입니다.  

**입출력 예**

|routes|return|
|:--:|:--:|
|[[-20,15], [-14,-5], [-18,-13], [-5,-3]]|2|

**입출력 예 설명**

-5 지점에 카메라를 설치하면 두 번째, 네 번째 차량이 카메라를 만납니다.

-15 지점에 카메라를 설치하면 첫 번째, 세 번째 차량이 카메라를 만납니다.

# Answer
: 입출력 예에서, **15가 아니라 -15다**. **오타**인 듯 싶다.

문제 예시를 대충 그려보면 다음과 같다.

![problem](/assets/images/2020-10-05-06-26-23_2020-10-04-problem_solving_12.md.png)

지난번에 풀었던 [공주님의 정원](https://cjlee38.github.io/problem_solving/problem_solving_6) 문제와 흡사하다. 

핵심 키포인트는, **모든 차량이 한번이라도 단속카메라를 만나야 한다는 것**,  
그리고 **최소한의 카메라를 구해야 한다는 것**이다.

가장 먼저, **진입지점을 우선순위**로, **진출지점을 차순위**로 하여 차량을 내림차순 정렬했다.
(이 방법이 답을 구하는 유일한 방법은 아니다.)  

다음으로, **진입지점이 가장 큰 차량**을 우선 기준으로 삼은 뒤,   
index 1번부터 loop를 돌면서 **"진출지점이 기준에 해당하지 않는"** 차량을 발견한다면,  
**그 차량의 진입지점**을 **다음 기준**으로 삼는다.

**문제 예시**로 설명하면 다음과 같다.

우선, 정렬을 하고 나면 배열은 다음과 같이 구성된다.

-5 -3  
-14 -5  
-18 -13  
-20 -15  

이후, 먼저 -5를 기준으로 삼고, loop를 시작한다.

-14 -5 는 진출지점인 -5가 기준에 해당한다(즉, 카메라에 찍힌다.)   
-18 -13 은, 진출지점인 -13이 기준에 해당하지 않으므로, 이 차량의 진입지점인 -18을 기준으로 삼는다(즉, 카메라를 설치한다.)  
-20 -15 는 진출지점인 -15가 -18인 기준에 해당한다.(즉, 카메라에 찍힌다.)

카메라는 **첫 기준인 -5**와, **두번째 기준인 -18** 의 두개로, 2를 return한다.

코드는 다음과 같다.

```java
package programmers;

import java.util.Arrays;

// 프로그래머스 - 단속카메라
public class p42884 {
    public static void main(String[] args) {
        int[][] routes = {
                {-20, 15},
                {-14, -5},
                {-18, -13},
                {-5, -3}
        };

        int answer = solution(routes);
        System.out.println(answer);
    }

    public static int solution(int[][] routes) {
        Arrays.sort(routes, (o1, o2) -> {
            if (o1[0] == o2[0]) return o1[1] < o2[1] ? 1 : -1;
            else return o1[0] < o2[0] ? 1 : -1;
        });

        int std = routes[0][0];
        int answer = 1;
        for (int i=1; i<routes.length; i++) {
            System.out.println("std : " + std);
            if (routes[i][1] < std) {
                std = routes[i][0];
                answer++;
            }
        }



        return answer;
    }
    
}

```