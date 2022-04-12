---
# author: cjlee
categories: problem-solving
# cover: /assets/covers/coding.png
date: "2020-09-23T22:12:00Z"
tags: ['programmers']
title: '# 프로그래머스[Lv.2] - 카펫 ( java )'
---
<script type="text/javascript" 
src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS_HTML">
</script>
[문제 링크](https://programmers.co.kr/learn/courses/30/lessons/42842)

# PROBLEM
: **문제 설명**  
Leo는 카펫을 사러 갔다가 아래 그림과 같이 중앙에는 노란색으로 칠해져 있고 테두리 1줄은 갈색으로 칠해져 있는 격자 모양 카펫을 봤습니다.

![carpet](/assets/images/2020-09-23-22-55-11_2020-09-23-problem_solving_9.md.png)

Leo는 집으로 돌아와서 아까 본 카펫의 노란색과 갈색으로 색칠된 격자의 개수는 기억했지만, 전체 카펫의 크기는 기억하지 못했습니다.

Leo가 본 카펫에서 갈색 격자의 수 brown, 노란색 격자의 수 yellow가 매개변수로 주어질 때 카펫의 가로, 세로 크기를 순서대로 배열에 담아 return 하도록 solution 함수를 작성해주세요.

**제한사항**  

갈색 격자의 수 brown은 8 이상 5,000 이하인 자연수입니다.
노란색 격자의 수 yellow는 1 이상 2,000,000 이하인 자연수입니다.
카펫의 가로 길이는 세로 길이와 같거나, 세로 길이보다 깁니다.

**입출력 예**  

|brown|yellow|return|
|:--:|:--:|:--:|
|10|2|[4, 3]|
|8|1|[3, 3]|
|24|24|[8, 6]|


# SOLVE
: 문제를 해결하는 방법은, 두 가지가 있을 것으로 보였다.

1. 방정식을 통해 해결하는 방법
2. 약수를 구해 해결하는 방법.


### 1) 약수 이용
문제가 완전 탐색이니 이는 방정식의 방법은 나중으로 미뤄두고,  
약수를 통해 구하는 방법을 먼저 고민해봤다.

입출력 예를 보면, **brown+yellow = width*height** 임을 알 수 있다.  
따라서, brown+yellow 를 나누었을 때 0이 남는, 즉 **약수를 이용**하면 될 것으로 보였다.  

이 때, brown 10, yellow 2, 즉 **합이 12일 때**를 생각해보면,  
생성될 수 있는 약수는 **(1, 12), (2, 6), (3, 4), (4, 3), (6, 2), (12, 1)** 이 된다.

카펫의 가로 길이는 세로 길이와 같거나, 더 길다고 하였으므로,  
대충 반 잘라내서 **(4, 3), (6, 2), (12, 1)** 중 하나를 골라야 한다.

물론, **width 혹은 height는 최소 3 이상**이므로  
(yellow의 최소 값인 1일때, brown은 8개 이고, 따라서 width가 되었든, height가 되었든 **최소 3 이상**이어야 한다.)  
이에 근거해 2와 1을 포함하고 있는 (6, 2), (12, 1)을 쳐낼 수도 있지만,   
카펫이 좀 넓어진다면, 이러한 방법론은 절대 통하지 않을 것으로 보였다.

그러나, 고민을 하던 중 하나의 패턴을 발견했는데,  
**약수의 합이 최소가 되는 순간**이 답이 될 것으로 보였다.

```java
public static int solution2(int brown, int yellow) {
        int sum = brown+yellow;
        int xySum = Integer.MAX_VALUE;
        int x;
        int y;
        for(int i=(sum/2)-1; i>3; i--) {
            if (sum%i == 0 && i+(sum/i) < xySum) {
                x = i;
                y = (sum/i);
                xySum = i+(sum/i);
            }
        }

        if (x < y) {
            int tmp = x;
            x = y;
            y = tmp;
        }

        int[] answer = {x, y};
        return answer;
    }
```

신나서 후다닥 대충 쓰고 제출했는데,  
4, 6, 7 번을 통과하지 못했다.  

한참을 고민하다가, 결국 질문하기 탭에서 다음과 같은 글을 보았다.

![hint](/assets/images/2020-09-23-23-06-13_2020-09-23-problem_solving_9.md.png){: .alignCenter}

이걸 보고, 생각을 해보니, 50+22인 72의 약수 중, 합이 최소가 되는 값은 **(9, 8)**이고,  
9*8의 brown은 절대로 50이 될 수 없다는 사실을 깨닫게 되었다.   
Shit...

차분하게,  **brown = (width * 2) + (height * 2) - 4** 를 상기하면서  
코드를 다시 작성하였다.

```java
public static int[] solution(int brown, int yellow) {
        int x = -1;
        int y = -1;
        int sum = brown+yellow;

        for (int i=1; i<sum; i++) {
            if (sum%i == 0 && (i*2) + ((sum/i)*2) - 4 == brown) {
                x = i;
                y = sum/i;
                break;
            }
        }

        return new int[]{Math.max(x, y), Math.min(x, y)};
    }
```

sum%i 가 0 이라면, i와 sum/i 는 당연히 sum의 약수가 된다.  
그 약수 중에서도,  
**(i*2) + ((sum/i)*2) - 4 == brown** 즉  
**brown = (width * 2) + (height * 2) - 4** 를 만족하는 조건문을 추가하였다.

### 2) 방정식 이용.
: 이 또한, 질문하기 탭의 다른 분의 도움을 받았다.  
기본적으로, 다음과 같은 식 두개를 구할 수 있다.

*w = width, h = height, b = brown, y = yellow*  


$$w+h = (b+4) / 2$$

$$wh = b+y$$  

이를 다음과 같이 이차 방정식으로 유도해낼 수 있다.
여기서, 보기가 별로 안좋으니  
**(b+4)/2 는 X**로,  
**b+y 는 Y**로 치환하자.

$$w = X - h$$

$$ h*(X - h) = Y$$   

$$ h^2 - hX + Y = 0$$  

여기서 근의 공식을 이용하면,

$$ width = (X + \sqrt{X^2 + 4*1*Y}) / 2 $$

$$ height = (X + \sqrt{X^2 - 4*1*Y}) / 2 $$

풀어서 쓰면, 다음과 같다.

$$ width = (((b+4)/2) + \sqrt{((b+4)/2)^2 + 4*1*(b+y)}) / 2 $$

$$ height = (((b+4)/2) + \sqrt{((b+4)/2)^2 - 4*1*(b+y)}) / 2 $$

코드는 다음과 같다.

```java
public static int[] solution(int brown, int yellow) {
        int term = (int) Math.sqrt(Math.pow((brown+4),2) /4 - 4 * (brown + yellow));
        int w = ((brown + 4) / 2 + term) / 2;
        int h = ((brown + 4) / 2 - term) / 2;

        return new int[]{w, h};
    }
```