---
# author: cjlee
categories: problem-solving
# cover: /assets/covers/coding.png
date: "2020-11-18T12:43:00Z"
tags: ['programmers']
title: '# 프로그래머스[Lv.4] - 숫자 블록 ( Java )'
---

[문제 링크](https://programmers.co.kr/learn/courses/30/lessons/12923)

# Problem

**문제 설명**  
그렙시에는 0으로 된 도로에 숫자 블록을 설치하기로 하였습니다. 숫자 블록의 규칙은 다음과 같습니다.

블록의 번호가 n 일 때, 가장 처음 블록은 n * 2번째 위치에 설치합니다. 그다음은 n * 3, 그다음은 n * 4, ...로 진행합니다.만약 기존에 블록이 깔려있는 자리라면 그 블록을빼고 새로운 블록으로 집어넣습니다.

예를 들어 1번 블록은 2,3,4,5, ... 인 위치에 우선 설치합니다. 그다음 2번 블록은 4,6,8,10, ... 인 위치에 설치하고, 3번 블록은 6,9,12... 인 위치에 설치합니다.

이렇게 3번 블록까지 설치하고 나면 첫 10개의 블록은 0, 1, 1, 2, 1, 3, 1, 2, 3, 2이됩니다.

그렙시는 길이가 1,000,000,000인 도로에 1번 블록부터 시작하여 10,000,000번 블록까지 위의 규칙으로 모두 놓았습니다.

그렙시의 시장님은 특정 구간의 어떤 블록이 깔려 있는지 알고 싶습니다.

구간을 나타내는 두 수 begin, end 가 매개변수로 주어 질 때, 그 구간에 깔려 있는 블록의 숫자 배열(리스트)을 return하는 solution 함수를 완성해 주세요.

**제한 사항**  
- begin, end 는 1 이상 1,000,000,000이하의 자연수 이고, begin는 항상 end보다 작습니다.
- end - begin 의 값은 항상 10,000을 넘지 않습니다.

**입출력 예**

|begin|end|result|
|:--:|:--:|:---:|
|1|10|[0, 1, 1, 2, 1, 3, 1, 4, 3, 5]|

**입출력 예 설명**

입출력 예 #1  
다음과 같이 블럭이 깔리게 됩니다.

![](/assets/images/2020-11-22-12-43-36_2020-11-18-problem_solving_20.md.png)


# Solve

## 접근 방법.
: LV.4 문제를 푸는 정도의 사람이라면, 당연히 위에서 설명하는 흐름대로 시뮬레이션해서 구현하지는 않을 것이다. 

길게 볼 것도 없이, 문제의 예시를 보면, 각각 최종적으로 깔리게 되는 블록의 값은 해당 숫자가 가질 수 있는 약수 중, 자기 자신을 제외한 최대 값이다.

즉, 10 이라는 숫자를 보면, 10 이 가질 수 있는 약수는 [1, 2, 5, 10] 이며, 이 중 자기 자신을 제외한 최대 값은 5 이므로, 5 가 채워지게 된다.

그런데 이 때, **주의할 점**이 있는데, 길이가 10억 인 도로에, **1000만 번 블록**까지 이러한 규칙으로 놓았다고 하였으므로, 해당 숫자의 블록은 1000만을 넘어설 수 없다.

따라서, **자기 자신을 제외하면서, 동시에 1000만 이하인 약수 중 최대값** 이 올바른 숫자가 되겠다.

별로 어렵지 않다 싶어, 곧바로 코드를 작성했다.

```java
public int[] solution(long begin, long end) {
    int n = (int) (end - begin + 1);
    int[] road = new int[n];

    for (int i = 0; i < n; i++) {
        long cur = begin + i;
        road[i] = getGD(cur);
    }

    // begin이 1인 경우, 1 번째는 0으로 채워야 한다. 문제 예시 참고
    if (begin == 1) road[0] = 0; 
    
    return road;
}

private int getGD(long cur) {
    for (long i = 2; i <= Math.sqrt(cur); i++) {
        if (cur % i == 0 && (cur / i) < 10000000) {
            return (int) (cur / i);
        }
    }

    return 1;
}
```

지난 번에 [소수를 찾는 방법](https://cjlee38.github.io/btb/ways_to_find_prime_number) 에서 다뤘듯이, square_root 까지만 살펴보면 되기 때문에, 시간 복잡도도 그리 높지 않을 것이라 예상했다.


![](/assets/images/2020-11-22-13-01-38_2020-11-18-problem_solving_20.md.png)

???

효율성 테스트에서 떨어졌다.  

다른 방법으로, 약수를 구할 때 2 를 먼저 검사하고, 이후 for loop에서 3을 시작점으로 잡은 뒤, i += 2 씩 더하는 방법으로 for 문의 수행을 절반으로 토막내더라도, 답을 구할 수 없다. 가령, 9억 9000만 2 라는 숫자는, 2로 약분되지만, 1000만으로는 약분될 수 없다. 대신, **142로 나누었을 때 나머지가 0 이 되면서, 동시에 몫이 1000만 이하인 6971831 이 답**이 된다.

**(틀린 코드)**
```java
private int getGD(long cur) {
    if (cur % 2 == 0) {
        return cur / 2 < 10000000L ? (int) cur / 2 : 10000000;
    }

    for (long i = 3; i <= Math.sqrt(cur); i += 2) {
        if (cur % i == 0 && (cur / i) < 10000000) {
            return (int) (cur / i);
        }
    }

    return 1;
}
```

바꿔 말하면, 2로 한번 검사 했더라도, 4, 6, 8 ... 또한 검사해줘야 한다는 의미가 된다.

그렇다면, 이렇게 고친 코드가 아닌, 기존의 코드에서 답을 구해야 한다는 이야기가 된다. 이것저것 시도해보다가, 다음과 같이 **long 이 아닌 int로 비교해줘야 한다** 는 답을 얻어내었다.


```java
private int getGD(int cur) {
    for (int i = 2; i <= Math.sqrt(cur); i++) {
        if (cur % i == 0 && (cur / i) < 10000000) {
            return cur / i;
        }
    }

    return 1;
}
```

어차피 문제에서 제시하는 최대 값은 10억이고, int 형은 약 21억까지 표현할 수 있으므로, long 변수를 int 로 캐스팅해주어도 문제가 되지 않는다.   

전체 코드는 다음과 같다.

```java
package programmers.lv4;

public class p12923 {
    public static void main(String[] args) {
        long begin = 990000000;
        long end = 990010000;

        p12923 p = new p12923();
        int[] answer = p.solution(begin, end);
    }

    public int[] solution(long begin, long end) {
        int n = (int) (end - begin + 1);
        int[] road = new int[n];

        for (int i = 0; i < n; i++) {
            int cur = (int) begin + i;
            road[i] = getGD(cur);
        }

        if (begin == 1) road[0] = 0;
        return road;
    }

    private int getGD(int cur) {
        for (int i = 2; i <= Math.sqrt(cur); i++) {
            if (cur % i == 0 && (cur / i) < 10000000) {
                return cur / i;
            }
        }

        return 1;
    }

}
```

호기심에 차이가 얼마나 나는지 궁금해서 시간을 측정해보았다.

begin은 990,000,000 으로, end 는 990,010,000 으로 설정한 뒤,   
10번을 실행해서 평균을 내본 결과는 다음과 같다.

- long 으로 비교한 경우 : 1021.5 ms
- int 로 비교한 경우 : 364.0 ms

약 3 배에 가까운 차이를 보였다.   

[링크](http://herongyang.com/JVM/Benchmark-Long-Performance-Comparison-Int-and-Long.html)를 참고하면, 이는 내 착각이 아니라 실제로 long이 int형보다 느리게 동작함을 보여주고 있다.

따라서, 간단한 프로그램이라면 괜찮지만, 위와 같이 무거운 작업을 수행해야 할 때에는 해당 값의 최대 범위를 잘 고려해서 int 형으로 캐스팅 하는 것을 고려해봐야 한다.