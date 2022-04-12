---
# author: cjlee
categories: btb
# cover: /assets/covers/coding.png
date: "2020-10-16T05:44:00Z"
tags: ['null']
title: '# 재귀함수 주무르기 with 피보나치 수열 ( feat. Java )'
---

# 0. 들어가며
: 나는 재귀함수를 좋아하지 않는다.  
재귀함수는 몇 번을 써도 어렵고, 몇 번을 봐도 쉽사리 파악하기 어렵다.  
그럼에도 불구하고, 재귀함수는 유용하고, 신기하다.

이번 포스팅을 통해, 재귀함수가 어떻게 동작하는지 간단하게 살펴보고,  
재귀함수를 최적화하기 위한 두 가지 방법에 대해서 알아보자.

# 1. 재귀함수의 사전적 정의
: 재귀(再歸) 라는 말을 국어사전에 쳐보면, 다음과 같이 설명한다.  

<center> [명사] 원래의 자리로 되돌아가거나 되돌아옴. </center>

이를 프로그래밍에 적용하면, **함수가 자기 자신으로 되돌아옴** 정도로 표현할 수 있겠다.  
다시 말해, 호출한 함수가 자기 자신을 호출하는 행위를 의미한다.

# 2. 해결할 문제.
이러한 재귀함수를 활용하는 가장 대표적인 문제는, 역시 피보나치 수열이다.  
잡다한 이야기는 쫙 빼고, 피보나치 수열이 무엇인가 하면,  
**N번째 항의 값은, 그 전항(N-1) 과, 그 전전항(N-2)의 값의 합이다.**  
라는 것으로 정의되는 수열이다.

즉, 만약 N이 3이라면, 다시 말해 `fibonacci(3)`을 구하고자 한다면,  
전항인 `fibonacci(2)`와, `fibonacci(1)`의 값을 더해서 구할 수 있다.*(이하 F() 로 통일)*  

그렇다면 F(0)과 F(1)을 구하기 위해 F(-1)과 같은 음수로 가느냐?  
그렇지는 않고, 0과 1의 값은 각각 0과 1로 정의되어 있다.

![](/assets/images/2020-10-25-06-03-21_2020-10-16-recursive_function.md.png)

간단하게, `F(4)`를 구해보자.
이를 구하기 위해서는, `F(3)`과 `F(2)`를 알아야 한다.   
`F(3)`과 `F(2)`를 구하기 위해서는, `F(2)`와 `F(1)`을, 그리고 `F(1)`과` F(0)`을 구해야 한다.

`F(1)`과 `F(0)`은 1과 0으로 정의되어 있으니, 다시 거꾸로 타고 올라가면 된다.

**F(2) = F(1) + F(0) = 1 + 0 = 1**  
**F(3) = F(2) + F(1) = 1 + 1 = 2**  
**F(4) = F(3) + F(2) = 2 + 1 = 3**  

답이 나왔다. F(4)의 답은 **3** 이다.

# 3. 해결법
: 여러 가지 해결 방법이 있지만, 반복문과 오늘의 핵심인 재귀함수 두가지 방법을 이용해보자.
## 1) 반복문
: 반복문을 이용하는 방법은 간단하다.  
배열을 만들어놓고, 전항의 값, 그리고 전전항의 값을 더해서 넣으면 된다.

```java
public static int fibonacci(int n) {
    int[] fibo = new int[n+1];
    fibo[0] = 0;
    fibo[1] = 1;

    for (int i = 2; i < fibo.length; i++) {
        fibo[i] = fibo[i-1] + fibo[i-2];
    }

    return fibo[n];
}
```

보는 바와 같이, 0번째 항과 1번째 항은 0과 1로 값을 미리 채워두고,  
반복문을 통해 더해가면서 구할 수 있다.

## 2) 재귀함수
: 그렇다면, 재귀함수는 어떨까?

```java
public static int fibonacci(int n) {
    if (n == 0) return 0;
    else if (n == 1) return 1;

    return fibonacci(n-1) + fibonacci(n-2);
}
```

삼항연산자로도 풀어낼 수 있지만,  
가장 기본에 충실하게 작성하면 위와 같이 작성할 수 있다.  

n에 3이 들어오는 경우,
`fibonacci(2)`와 `fibonacci(1)` 이 각각 호출되고,  
`fibonacci(2)`는 `fibonacci(1)`과 `fibonacci(0)`을 호출한다.

`fibonacci(1)`은 1을, `fibonacci(0)`은 0을 return하므로,    
이 둘을 더해 `fibonacci(2)`는 1이 된다.

따라서, `fibonacci(2) + fibonacci(1)`은 `1+1`이 되어,  
최종적으로 2가 return된다.

여기까지 모두가 행복하게 잘 살았습니다의 해피엔딩으로 끝나면 좋겠지만,  
**문제가 남아있다.**

# 4. 문제점
: 만약, 피보나치에 100을 요구하면 어떻게 될까?   
결과가 나타나지 않는다. 너무 오래 걸리는 탓이다.  
반토막을 내서 50을 넣어봐도, 약 65초가 걸렸다.  
(여담으로 값은 약 120억으로, int형의 최대값인 21억을 훨씬 넘어선다.)

왜 이렇게 오래 걸릴까?  
그 이유는, 재귀함수를 돌면서, Call Stack이 쌓이기 때문이다.

위 코드를 다시 보자.  
`fibonacci(n)`을 구하기 위해, `fibonacci(n-1)`, `fibonacci(n-2)`를 수행해야 한다.  
당연하게 보이지만, `fibonacci(n-1)`과 `fibonacci(n-2)`를 구하기 전까지는,  
`fibonacci(n)`은 대기하고 있는 상태이다.

즉, n이 0 또는 1이 되기 전까지는 계속해서 대기, 대기, 대기 하는 함수들이 쌓이고,  
0 또는 1이 되어서야 다시 돌아와서 값을 구하고, 더해야 하기 때문이다.

![](/assets/images/2020-10-25-06-44-21_2020-10-16-recursive_function.md.png)

<center> 디버깅을 해보면, Stack이 쌓이는 것을 볼 수 있다. </center>

# 5. 해결 방법
: 이는 다음의 두 가지 방법으로 해결할 수 있다.
## 1) Tail Recursion
: 먼저 활용할 수 있는 방법은 꼬리재귀를 이용하는 방법이다.  
꼬리재귀는, 재귀함수의 탈을 쓴 while문과 같다.  

앞서, 위와 같이 계속해서 Stack이 쌓였던 이유는,  
`fibonacci(n-1)`과 `fibonacci(n-2)`를 구하고 나서,  
이를 더하는 연산이 필요했기 때문이다.

즉, **아직 할 일이 남아있었다.**

그런데, 할 일이 없다면 어떨까?  
그냥 곧바로, 가장 최초로 함수를 실행한 지점으로 return을 쏴버리면 안될까?  
안될 것 없다.

코드를 다음과 같이 수정해보자.

```java
public static int fibonacci(int n) {
    return fibonacciTail(n, 0, 1);
}

public static int fibonacciTail(int n, int prev, int next) {
    if (n == 0) return prev;
    return fibonacciTail(n-1, next, prev+next);
}
```

앞서 코드는, fibonacci(n)을 구하기 위해서   
계산을 최대한 뒤로 미루는 듯한 느낌을 받았는데,  

새로운 코드는 계산을 한다음에 그 값을 넘겨준다.  
그리고, 직접 디버깅으로 하나하나 실행해보면 알겠지만,  
n이 되는 순간 곧바로 `fibonacciTail()`이 아닌 `fibonacci()` 로 쏴버린다.

반복 횟수는 어떨까?  
Fibonacci(10)을 기준으로,  
기존의 방식은 **177번**,  
새로운 방식은 **11번** 반복했다.

N이 늘어나면 늘어날수록, 이 차이는 더더욱 극명하게 드러날 것이다.

> Note. 좀 더 자세히 찾아보았는데, **진정한** 꼬리재귀는 Stack Frame 자체를 쌓지 않는다.  즉, **꼬리재귀함수를 지원하지 않는다는 Python** 에서도, 위와 같은 코드는 구현할 수 있고, 실제로 잘 동작하기도 한다. 그러나, **꼬리재귀함수를 지원하는** 컴파일러인 경우, 애초에 Stack Frame을 쌓지 않도록 이를 while문으로 애초에 **"바꾸어버린다"** 는 것이다. 따라서, 다음과 같이 정리할 수 있겠다.

> 꼬리재귀를 지원하는 경우 : 애초에 while문으로 바뀌기 때문에, Stack Frame 자체가 쌓이지 않음. 즉, StackOverflow 의 에러가 절대 발생하지 않는다. -> Tail Call Optimization  
> 꼬리재귀를 지원하지 않는 경우 : 위와 같은 로직으로 최적화는 되더라도, Stack은 쌓이기 때문에, StackOverflow가 발생할 수 있다.

## 2) 동적계획법(cache)을 이용
: 문제가 발생했던 코드의 경우, **같은 연산을 반복하는** 코드가 존재한다.  

즉, `F(4)`를 구하기 위해서, `F(3)`과 `F(2)`를 구해야 했는데,  
`F(3)`을 구하기 위해서, **또 다시** `F(2)` 에 대한 연산을 해야하는 것이다.

그러므로, 재귀함수를 반복하면서,  
**구하고자 하는 값이, 처음 구하는 경우에는 직접 연산**,  
**한번 구한적 있는 값이라면, 꺼내오기**  
를 이용하면, 좀 더 똑똑하게 가져올 수 있다.

다음과 같이 코드를 작성해보자.
```java
static int[] fibo;

public static void main(String[] args) {
    int n = 10;
    fibo = new int[n+1];
    int result = fibonacci(n);
    System.out.println(result);
}

public static int fibonacci(int n) {
    if (fibo[n] != 0) return fibo[n];
    if (n == 0) return 0;
    else if (n == 1) return 1;

    fibo[n] = fibonacci(n-1) + fibonacci(n-2);
    return fibo[n];
}
```

int[] 배열 fibo를 선언한 뒤, main method에서 n+1 만큼의 길이로 정의해주었다.  

다음으로, `fibonacci()` 함수가 실행될 때마다,   
값이 있는지 확인하고,   
**없는 경우에는 기존의 로직을**,    
**있는 경우에는 그 값을 곧바로 return하도록** 만들었다.

그리고 최종적으로, 원하는 fibonacci 값인 `fibo[n]`을 return했다.  
반복 횟수는, 10을 기준으로 19번을 반복했다.

# 6. 마치며
: 하나의 문제를 푸는데에 여러 가지 방법을 아는 것은 큰 도움이 된다.  
모든것을 한방에 해결해주는 만능 알고리즘은 없으니,  
상황에 맞게 적절하게 사용하는 것이 중요한데,  
이 때 내가 **어떤 무기, Tool**을 갖고 있느냐에 따라  
선택의 폭도 달라지고, 생각의 틀도 격변하기 때문인 것 같다.

위 피보나치를 구하는 방법에도,   
일반항을 이용하는 방법이나, 행렬을 이용하는 방법이 있다고 하는데,  
본 포스팅의 목적은 피보나치가 아닌 재귀함수 이므로 이에 대해서 따로 찾아보지는 않았다.  
궁금한 사람은 이하의 Reference를 참고하면 좋겠다.

이상으로 포스팅을 마칩니다.

## Reference
- [피보나치 수열 알고리즘을 해결하는 5가지 방법](https://shoark7.github.io/programming/algorithm/%ED%94%BC%EB%B3%B4%EB%82%98%EC%B9%98-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98%EC%9D%84-%ED%95%B4%EA%B2%B0%ED%95%98%EB%8A%94-5%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95.html)
- [Tail Recursion in Python](http://philosophical.one/posts/tail-recursion-in-python/)
- [StackOverflow - What is tail-recursion elimination?](https://stackoverflow.com/questions/1240539/what-is-tail-recursion-elimination)
- [Wikipedia - Tail call](https://en.wikipedia.org/wiki/Tail_call)