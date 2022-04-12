---
# author: cjlee
categories: problem-solving
# cover: /assets/covers/coding.png
date: "2020-08-29T17:16:00Z"
tags: ['programmers']
title: '# 프로그래머스[Lv.2] - 올바른 괄호 ( java )'
---

# PROBLEM

문제 설명  
괄호가 바르게 짝지어졌다는 것은 '(' 문자로 열렸으면 반드시 짝지어서 ')' 문자로 닫혀야 한다는 뜻입니다.   예를 들어  

* ()() 또는 (())() 는 올바른 괄호입니다.
* )()( 또는 (()( 는 올바르지 않은 괄호입니다.

'(' 또는 ')' 로만 이루어진 문자열 s가 주어졌을 때, 문자열 s가 올바른 괄호이면 true를 return 하고, 올바르지 않은 괄호이면 false를 return 하는 solution 함수를 완성해 주세요.

**제한사항**
* 문자열 s의 길이 : 100,000 이하의 자연수
* 문자열 s는 '(' 또는 ')' 로만 이루어져 있습니다.

# SOLVE

: 전형적인 Stack 문제이다. 순서는 다음과 같다.

1. 입력 받은 String을 char로 나누어 array에 담는다.
2. array를 돌면서, char가 "(" 이면 stack에 Push, ")" 이면 Pop 한다.
3. 이 때, **stack이 비어 있는데 pop**을 하려고 한다면, 잘못된 괄호이므로 answer는 false가 된다.
4. 또한, **array를 전부 돌았는데, stack이 비어 있지 않다면,** 이 또한 잘못된 괄호다.

Level2 인데, 오히려 생각할 내용은 [지난 프로그래머스 문제](https://cjlee38.github.io/problem_solving/2020/08/08/ps_1.html)보다 간단해서 살짝 당황했다. Level이 잘못 배치된 것 아닌가 싶었다.

해서, 처음에는 이렇게 짰다.


```java
import java.util.Stack;
class Solution {
    boolean solution(String s) {
        boolean answer = true;
        Stack<Character> stack = new Stack<>();
        
        for (int i = 0; i < s.length() ; i++) {
            char c = s.charAt(i);
            if (c == '(') { // 여는 괄호라면, 무조건 Push
                stack.push(c); 
            }
            else { // 닫는 괄호일 때,
                if ( !stack.empty() && stack.peek() == '(') { // stack이 비어 있지 않고, 최상단이 열린 괄호라면 pop
                    stack.pop();
                } else { // stack이 비어있거나, 최상단이 여는괄호가 아니라면 답은 무조건 false
                    answer = false;
                    break;
                }
            }
        }
        
        // 모든 array를 다 돌았음에도, stack이 비어 있지 않다면 답은 false (4번에 해당)
        if ( !stack.empty()) { answer = false; }
        return answer;
    }
}
```

근데, 시간 초과가 났다. 당황에 황당이 더해졌다. 시간복잡도는 어떻게 해도 O(N) 일텐데..

고민해보니, 문제가 될 부분은 아마 Stack Object 때문이라는 생각이 들어, 대신 int형으로 수정해서 제출해보았다.

___

(수정)
기존에 peek() 함수를 사용했더니, 시간 초과가 났다.  
생각해보면, peek() 함수를 사용할 일은 전혀 없는데, 왜 넣었을까..
peek() 함수를 빼고 실행하니 효율성 문제도 통과했다.

___

```java
class Solution {
    boolean solution(String s) {
        int iStack = 0;
        for (int i = 0; i < s.length() ; i++) {
            char c = s.charAt(i);
            if (c == '(') { iStack++; }
            else { iStack--; }
            if ( iStack < 0 ) { return false; }
        }
        if (iStack > 0) { return false; }
        else { return true; }
    }
}
```
이렇게 하니 성공했다. 아마 이 부분 때문에 Level 2로 선정되었나 싶기도 하다.  
근데, 이렇게까지 공간복잡도를 최소화하는 이유가 뭐지... 잘 모르겠다.

