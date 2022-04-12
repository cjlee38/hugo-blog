---
# author: cjlee
categories: problem-solving
# cover: /assets/covers/coding.png
date: "2020-09-28T20:13:00Z"
tags: ['programmers']
title: '# 프로그래머스[Lv.2] N진수 게임( Java )'
---

[문제 링크](https://programmers.co.kr/learn/courses/30/lessons/17687)
# Problem
**N진수 게임**

튜브가 활동하는 코딩 동아리에서는 전통적으로 해오는 게임이 있다.   
이 게임은 여러 사람이 둥글게 앉아서 숫자를 하나씩 차례대로 말하는 게임인데, 규칙은 다음과 같다.  


1. 숫자를 0부터 시작해서 차례대로 말한다. 첫 번째 사람은 0, 두 번째 사람은 1, … 열 번째 사람은 9를 말한다.  
2. 10 이상의 숫자부터는 한 자리씩 끊어서 말한다. 즉 열한 번째 사람은 10의 첫 자리인 1, 열두 번째 사람은 둘째 자리인 0을 말한다.  

이렇게 게임을 진행할 경우,  
`0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 1, 0, 1, 1, 1, 2, 1, 3, 1, 4, …`  
순으로 숫자를 말하면 된다.  

한편 코딩 동아리 일원들은 컴퓨터를 다루는 사람답게 이진수로 이 게임을 진행하기도 하는데, 이 경우에는  
`0, 1, 1, 0, 1, 1, 1, 0, 0, 1, 0, 1, 1, 1, 0, 1, 1, 1, …`  
순으로 숫자를 말하면 된다.  

이진수로 진행하는 게임에 익숙해져 질려가던 사람들은 좀 더 난이도를 높이기 위해   
이진법에서 십육진법까지 모든 진법으로 게임을 진행해보기로 했다.   
숫자 게임이 익숙하지 않은 튜브는 게임에 져서 벌칙을 받는 굴욕을 피하기 위해,   
자신이 말해야 하는 숫자를 스마트폰에 미리 출력해주는 프로그램을 만들려고 한다.  

튜브의 프로그램을 구현하라.

**입력 형식**  

진법 n, 미리 구할 숫자의 갯수 t, 게임에 참가하는 인원 m, 튜브의 순서 p 가 주어진다.  

- 2 ≦ n ≦ 16  
- 0 ＜ t ≦ 1000  
- 2 ≦ m ≦ 100  
- 1 ≦ p ≦ m  

**출력 형식**
튜브가 말해야 하는 숫자 t개를 공백 없이 차례대로 나타낸 문자열.     
단, 10~15는 각각 대문자 A~F로 출력한다.  

**입출력 예제**

|n|t|m|p|result|
|:--:|:--:|:--:|:--:|:--:|
|2|4|2|1|"0111"|
|16|16|2|1|"02468ACE11111111"|
|16|16|2|2|"13579BDF01234567"|


# Answer
: N 진수를 구하는것만 해결한다면, 어렵지 않게 해결할 수 있는 문제다.  
처음에는 2,8,10,16 의 기본적인 진수만 고려했는데,   
문제를 다시 읽어보니 2부터 16까지의 진수가 나올 수 있다는 것을 알았다.  
즉, 9진수, 13진수 등과 같은 경우도 생길 수 있다는 뜻이다.

직접 구하기 위해서, 다음과 같이 작성하였다.


```java
final static private String[] BASE = {"0","1","2","3","4","5","6","7","8","9","A","B","C","D","E","F"};

public static String getNumberString(int n, int num) {
    StringBuilder sb = new StringBuilder();

    do {
        sb.append(BASE[num%n]);
        num /= n;
    } while (num != 0);

    return sb.reverse().toString();
}

```
0부터 F까지의 숫자를 구해놓고, num이 0 이 아닐때까지 해당 진수의 값을 구하는 방법이다.

즉, 가령 16진수가 조건으로 들어왔고, 10진수로 `10` 이라는 숫자가 들어왔다면,  
10%16 = 10 이므로, "A"가 StringBuilder에 append 된다.

만약 31 이라는 숫자가 들어왔다면,   
31 % 16 =  "F" *(append)*   
31 / 16 = 1   
1 % 16 = "1" *(append)*  
1 / 16 = 0 *(break)*  

StringBuilder("F1").reverse().toString = "1F"

가 된다.

이 때, **num이 0부터 시작**하므로   
while loop 가 아닌 **do while loop** 를 사용하였다.

2~16 진수에 대한 값을 구할 수 있다면, 나머지는 간단하다.


```java
package programmers;

public class p17687 {
    final static private String[] BASE = {"0","1","2","3","4","5","6","7","8","9","A","B","C","D","E","F"};
    public static void main(String[] args) {
        int n = 16;
        int t = 16;
        int m = 2;
        int p = 1;
        String result = solution(n, t, m, p);
        System.out.println(result);
    }

    public static String solution(int n, int t, int m, int p) {
        StringBuilder sb = new StringBuilder();
        
        // T*M = 튜브가 말해야 하는 숫자 T 개 * 게임에 참가하는 인원 M 명
        // 자릿수가 두 개 이상 넘어가면 사용하지 않는 영역이 생기지만,
        // 어쨌든 부족할 일은 없다.
        for (int i=0; i<t*m; i++) { 
            sb.append(getNumberString(n, i));
        }



        StringBuilder tube = new StringBuilder();
        for (int i=0; i<sb.length(); i++) {
            if (tube.length() == t) {
                break;
            } else if (i % m == (p-1)) { // p가 1부터 시작하므로 p-1로 0으로 맞춰줌.
                tube.append(sb.charAt(i));
            }
        }

        return tube.toString();
    }

    public static String getNumberString(int n, int num) {
        StringBuilder sb = new StringBuilder();

        do {
            sb.append(BASE[num%n]);
            num /= n;
        } while (num != 0);

        return sb.reverse().toString();
    }
}
```
