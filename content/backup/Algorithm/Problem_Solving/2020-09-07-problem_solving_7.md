---
# author: cjlee
categories: problem-solving
# cover: /assets/covers/coding.png
date: "2020-09-07T01:51:00Z"
tags: ['programmers']
title: '# 프로그래머스[Lv.2] - 튜플 ( java )'
---

[문제 링크](https://programmers.co.kr/learn/courses/30/lessons/64065)

# PROBLEM
{% raw %}
문제 설명

셀수있는 수량의 순서있는 열거 또는 어떤 순서를 따르는 요소들의 모음을 튜플(tuple)이라고 합니다. n개의 요소를 가진 튜플을 n-튜플(n-tuple)이라고 하며, 다음과 같이 표현할 수 있습니다.

(a1, a2, a3, ..., an)  
튜플은 다음과 같은 성질을 가지고 있습니다.

중복된 원소가 있을 수 있습니다. ex : (2, 3, 1, 2)  
원소에 정해진 순서가 있으며, 원소의 순서가 다르면 서로 다른 튜플입니다.  
ex : (1, 2, 3) ≠ (1, 3, 2)

튜플의 원소 개수는 유한합니다.  
원소의 개수가 n개이고, 중복되는 원소가 없는 튜플 (a1, a2, a3, ..., an)이 주어질 때(단, a1, a2, ..., an은 자연수), 이는 다음과 같이 집합 기호 '{', '}'를 이용해 표현할 수 있습니다.  

{{a1}, {a1, a2}, {a1, a2, a3}, {a1, a2, a3, a4}, ... {a1, a2, a3, a4, ..., an}}

예를 들어 튜플이 (2, 1, 3, 4)인 경우 이는  

{{2}, {2, 1}, {2, 1, 3}, {2, 1, 3, 4}}  
와 같이 표현할 수 있습니다. 이때, 집합은 원소의 순서가 바뀌어도 상관없으므로

{{2}, {2, 1}, {2, 1, 3}, {2, 1, 3, 4}}  
{{2, 1, 3, 4}, {2}, {2, 1, 3}, {2, 1}}  
{{1, 2, 3}, {2, 1}, {1, 2, 4, 3}, {2}}  
는 모두 같은 튜플 (2, 1, 3, 4)를 나타냅니다.

특정 튜플을 표현하는 집합이 담긴 문자열 s가 매개변수로 주어질 때, s가 표현하는 튜플을 배열에 담아 return 하도록 solution 함수를 완성해주세요.

#### [제한사항]
* s의 길이는 5 이상 1,000,000 이하입니다.
* s는 숫자와 '{', '}', ',' 로만 이루어져 있습니다.
* 숫자가 0으로 시작하는 경우는 없습니다.
* s는 항상 중복되는 원소가 없는 튜플을 올바르게 표현하고 있습니다.
* s가 표현하는 튜플의 원소는 1 이상 100,000 이하인 자연수입니다.
* return 하는 배열의 길이가 1 이상 500 이하인 경우만 입력으로 주어집니다.

#### [입출력 예]
* "{{2},{2,1},{2,1,3},{2,1,3,4}}" -> [2, 1, 3, 4]  
* "{{1,2,3},{2,1},{1,2,4,3},{2}}" -> [2, 1, 3, 4]
* "{{20,111},{111}}" -> [111, 20]
* "{{123}}"	-> [123]
* "{{4,2,3},{3},{2,3,4,1},{2,3}}" -> [3, 2, 4, 1]


# SOLVE
: 문제에서 설명하는대로, 튜플은 **"순서"**를 따져야 한다. 처음 30초간은 "그냥 가장 긴놈 return하면 되는거 아닌가?" 싶었는데, 예제 2번과 같이, "set 내에서의 순서는 무관" 하다는 걸 보고 아니라는 것을 알았다.

### 1. 접근 방법
: 즉, 어떻게 접근해야 하느냐?

"길이가 1인 set에서 등장하는 녀석" -> "길이가 2인 set에서 처음 등장하는 녀석" -> ... "길이가 n인 set에서 처음 등장하는 녀석" 순서로 담아줘야, 올바른 tuple이 된다.

다시, 입출력 예제 2번을 보자

"{{1,2,3},{2,1},{1,2,4,3},{2}}"

원소의 길이가 1인 set은 {2} 이므로, 처음에 2를 담는다.  
다음으로, 원소의 길이가 2인 set은 {2,1} 이므로, 처음 등장하는 1을 담는다.  
다음으로, 원소의 길이가 3인 set은 {1,2,3} 이므로, 처음 등장하는 3을 담는다.  
마지막으로, 원소의 길이가 4(n)인 set은 {1,2,4,3} 이므로, 처음 등장하는 4를 담는다.

따라서, 최종적으로 [2, 1, 3, 4]가 된다.

이를 위해선, 다음의 두 가지 과정이 필요하다.

A. 입력이 "문자열" 이므로, 이를 잘 나눠서 이중 배열에 담아야 한다.  
B. 위에서 언급한대로, tuple에 잘 담는다.

#### 문자열 파싱하기
: 간단하게, set을 구분하는 구분자는 "},{" 이므로, 이 구분자를 기준으로 split 한 뒤에, 남은 brackets를 삭제하고, 담으면 된다. 즉

```java
ArrayList<String[]> list = new ArrayList<>();
String str = "{{1,2,3},{2,1},{1,2,4,3},{2}}";
// String[] sets = str.split("},{"); // wrong.
String[] sets = str.split("\\},\\{"); // split 할 때, bracket을 character로 받으려면 이와 같이 escape를 붙여줘야 한다.
sets = sets.replace("{","").replace("}");
list.add(sets.split(","));
```

문자열을 파싱하는 방법은 이것 말고도 여러 방법이 있을 것이다.

### 2. 첫 번째 방법
위 순서를 그대로 따르는 전체 코드는 다음과 같다.

```java
package Programmers;

import java.util.ArrayList;
import java.util.LinkedList;

public class p64065 {
    public int[] solution(String s) {
        ArrayList<String[]> list = new ArrayList<>();
        LinkedList<String> tuple = new LinkedList<>();
        String[] sets = s.split("\\},\\{");
        
        for (String set : sets) {
            set = set.replace("{","").replace("}","");
            list.add(set.split(","));
        } // 문자열 파싱 & 배열형태로 ArrayList에 담는다.
        
        for(int i = 1; i <= sets.length; i++) { // set의 개수 만큼 i번 반복
            for (String[] set : list) { 
                if (set.length == i) { // set 중에서, 현재의 길이(i)와 같다면
                    for (String num : set) {
                        if (!tuple.contains(num)) { // 새로 등장하는 요소를 추가.
                            tuple.add(num);
                            break;
                        }
                    }
                }
            }
        }
        
        int[] answer = new int[tuple.size()]; // LinkedList를 Array로 변환해서 제출
        for(int i=0; i<tuple.size(); i++) {
            answer[i] = Integer.parseInt(tuple.get(i));
        }

        
        return answer;
    }
}
}
```

### 3. 두 번째 방법
: 근데, 뭔가 찝찝하다. 중간에 배열에 담는 과정을 보면, for문이 세번이나 중첩되어 있다. if문까지 보면 indentation이 5번이나 되는데, 이런 코드를 보고 있노라면 굉장히 심적으로 불안해진다. 

물론, list를 돌 때마다, 이미 확인한 set은 삭제하는 방법을 사용할 수도 있지만, 근본적인 해결책은 되지 않는다. 

문제를 해결하자마자, 다른 사람의 코드를 찾아 보니, HashMap을 쓰는 방법도 있다는 것을 알게 되었다.

json(혹은 dictionary) 형태로 HashMap을 들여다보면 다음과 같이 생겼다. Key는 원소로, Value는 원소의 등장 횟수로 보는 것이다.

```python
map = {
    2 : 4,
    1 : 3,
    3 : 2,
    4 : 1
}
```

처음에 주어진 문자열에서, curly brackets가 한 개이던, 두 개이던 신경쓰지 않는다(즉, flatten 한다.)

그리고, 등장하는대로, 없으면 새로 만들고, 있으면 ++ 시킨다.

이제 tuple이라는 이름의 배열에 담을 때에는, "숫자가 많은 순서"대로 담기만 하면 되는 것이다. 

아주 신묘한 해결책인 것 같다. 이를 코드로 표현하면 다음과 같다.(원 코드는 Java 8 문법을 사용해서 아주 우아하고 멋있게 짰지만, 그대로 copy & paste 하기엔 양심에 찔려서.. 동일 아이디어로, 나만의 아주아주 허접한 방식대로 코드를 작성했다.)

```java
package Programmers;

import java.util.HashMap;

public class p64065 {
    public int[] solution(String s) {
        HashMap<Integer, Integer> map = new HashMap<>();
        String[] elements = s.replace("{","").replace("}","").split(","); // 문자열 파싱
        
        // HashMap에 저장
        for (String e : elements) {
            Integer key = Integer.parseInt(e);
            Integer value = map.get(key);
            if (value == null) { map.put(key, 1); }
            else { map.put(key, value + 1); }
        }
        
        // Key-Value를 Reversing
        HashMap<Integer, Integer> reversed = new HashMap<>();
        for (Integer i : map.keySet()) {
            reversed.put(map.get(i), i);
        }
        
        // 배열의 0번부터 n번까지, element의 개수가 n개부터 1개인 녀석까지 담음.
        int[] answer = new int[reversed.size()];
        for(int i = 0; i < reversed.size(); i++) {
            answer[i] = reversed.get(reversed.size() - i);
        }
        return answer;
    }
}
```

# 마치며
: 아마 첫 번째 코드는 실제로 효율성까지 체크 했다면 바로 나가리였을 것이다. 항상 느끼지만, 문제를 곧이 곧대로 받아들이기보다는, 한 단계 더 Develop 해서 다른 각도에서 쳐다볼 수 있어야 진정한 PS 고수로 거듭날 수 있지 않을까 싶다.


{% endraw %}
