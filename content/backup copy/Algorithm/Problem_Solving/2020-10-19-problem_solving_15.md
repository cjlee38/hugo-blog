---
# author: cjlee
categories: problem-solving
# cover: /assets/covers/coding.png
date: "2020-10-19T19:45:00Z"
tags: ['programmers']
title: '# 프로그래머스[Lv.2] 숫자의 표현 ( Java )'
---

[문제 링크](https://programmers.co.kr/learn/courses/30/lessons/12924)

# Problem
**문제 설명**

Finn은 요즘 수학공부에 빠져 있습니다. 수학 공부를 하던 Finn은 자연수 n을 연속한 자연수들로 표현 하는 방법이 여러개라는 사실을 알게 되었습니다. 예를들어 15는 다음과 같이 4가지로 표현 할 수 있습니다.

- 1 + 2 + 3 + 4 + 5 = 15
- 4 + 5 + 6 = 15
- 7 + 8 = 15
- 15 = 15


자연수 n이 매개변수로 주어질 때, 연속된 자연수들로 n을 표현하는 방법의 수를 return하는 solution를 완성해주세요.

**제한사항**

n은 10,000 이하의 자연수 입니다.

**입출력 예**  

|n|result|
|:--:|:--:|
|15|4|

# Solve
: 이런 문제를 푸는 방법은 두 가지 중 하나인 것 같다.  
수학을 엄청 잘해서, 머릿속에 수식을 만들어 낸 뒤 이를 코드로 풀어내거나,  
혹은 깊은 고민 끝에 패턴을 찾아내거나.

당연히 나는 후자에 속해서, 어떻게 하면 패턴을 찾아낼 수 있을까 고민 끝에,  
다음과 같은 방법을 찾아냈다.

흐름을 설명하면, 다음과 같다.

1. 숫자가 가감되는 **sum**, 1,2번에서 설명할 **head**, **tail**, 그리고 정답의 **count**를 정의한다.
2. 목표 숫자 n이 될 때까지, factorial을 계산하듯이 **head**를 1씩 늘려가며 **sum**에 더한다.
3. sum이 n보다 같거나 크면, **n보다 작아질 때까지** 1씩 증가하는 **tail**을 뺀다.
4. 와중에, n과 같으면, count를 증가시킨다.
5. 1~3번의 과정을 n까지 반복한다.

역시나, 말로 설명하면 이해하기 힘들다. 예시를 보자.

1부터 5까지 head를 늘려나가면서, sum 에 더하면, 다음과 같이 된다.

**1 + 2 + 3 + 4 + 5(head = 5, tail = 1, count = 0)**

이 때, sum이 n보다 크거나 같으므로, n보다 작아질때까지 tail을 뺀다.
와중에, n과 sum이 같으므로 count를 1 증가시킨다.

**2 + 3 + 4 + 5(head = 5, tail = 2, count = 1)**

sum이 n보다 작아졌으므로, 다시 2번의 더하기를 시작한다.

**2 + 3 + 4 + 5 + 6(head = 6, tail = 2, count = 1)**

sum이 n보다 크거나 같으므로, 다시 3번의 빼기를 시작한다.

**3 + 4 + 5 + 6(head = 7, tail = 3, count = 1)**    
**4 + 5 + 6(head = 7, tail = 4, count = 1)**  
**5 + 6(head = 7, tail = 5, count = 2)**  

중간에 4+5+6 에서, sum이 n과 같으므로, count가 하나 증가한 모습을 볼 수 있다.

이러한 과정을 반복하면, 답을 손쉽게 구할 수 있다.

전체 코드는 다음과 같다.

```java
package programmers.lv2;

public class p12924 {
    public static void main(String[] args) {
        int n = 15;
        int result = solution(n);
        System.out.println(result);
    }

    public static int solution(int n) {
        int sum = 0, count = 0, tail = 1, head = 1;

        for ( ; head <= n; head++) {
            sum += head;
            while(sum >= n) {
                if (sum == n) count++;
                sum -= tail;
                tail++;
            }
        }

        return count;
    }
}
```

# 다른 사람의 풀이
: 다른 분들의 풀이를 보니, 대개 두가지로 나뉘는 것 같았다.

1. "**주어진 자연수를 연속된 자연수의 합으로 표현하는 방법의 수는 주어진 수의 홀수 약수의 개수와 같다라는 정수론 정리가 있습니다"** 라는 멘트가 달려있는, 문제 풀이와 전혀 연관이 없어 *보이는*... 놀라운 코드

2. i를 num까지 반복하면서, 각 i마다 i부터 시작하는 j변수가 num까지 달려가면서 정답여부를 확인하는 (아마 시간복잡도로 O(NlogN)이었던 것 같은데) 코드

잠시 구글링을 해봤더니, 효율성이 2번의 코드는 약 1.5ms ~ 2 ms 인 반면,  
내 코드는 0.4ms ~ 0.5ms 정도가 나왔다. 

1번코드의 효율성은... 따질 필요도 없지않을까 싶다.
