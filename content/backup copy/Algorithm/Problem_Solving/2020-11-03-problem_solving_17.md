---
# author: cjlee
categories: problem-solving
# cover: /assets/covers/coding.png
date: "2020-11-03T10:52:00Z"
tags: ['programmers']
title: '# 프로그래머스[Lv.2] [1차] 뉴스 클러스터링 ( Java )'
---

[문제 링크](https://programmers.co.kr/learn/courses/30/lessons/17677)

# Problem

**문제 설명**

뉴스 클러스터링

여러 언론사에서 쏟아지는 뉴스, 특히 속보성 뉴스를 보면 비슷비슷한 제목의 기사가 많아 정작 필요한 기사를 찾기가 어렵다. Daum 뉴스의 개발 업무를 맡게 된 신입사원 튜브는 사용자들이 편리하게 다양한 뉴스를 찾아볼 수 있도록 문제점을 개선하는 업무를 맡게 되었다.

개발의 방향을 잡기 위해 튜브는 우선 최근 화제가 되고 있는 카카오 신입 개발자 공채 관련 기사를 검색해보았다.

카카오 첫 공채..'블라인드' 방식 채용  
카카오, 합병 후 첫 공채.. 블라인드 전형으로 개발자 채용  
카카오, 블라인드 전형으로 신입 개발자 공채  
카카오 공채, 신입 개발자 코딩 능력만 본다  
카카오, 신입 공채.. 코딩 실력만 본다  
카카오 코딩 능력만으로 2018 신입 개발자 뽑는다  

기사의 제목을 기준으로 블라인드 전형에 주목하는 기사와 코딩 테스트에 주목하는 기사로 나뉘는 걸 발견했다. 튜브는 이들을 각각 묶어서 보여주면 카카오 공채 관련 기사를 찾아보는 사용자에게 유용할 듯싶었다.

유사한 기사를 묶는 기준을 정하기 위해서 논문과 자료를 조사하던 튜브는 자카드 유사도라는 방법을 찾아냈다.

자카드 유사도는 집합 간의 유사도를 검사하는 여러 방법 중의 하나로 알려져 있다. 두 집합 A, B 사이의 자카드 유사도 J(A, B)는 두 집합의 교집합 크기를 두 집합의 합집합 크기로 나눈 값으로 정의된다.

예를 들어 집합 A = {1, 2, 3}, 집합 B = {2, 3, 4}라고 할 때, 교집합 A ∩ B = {2, 3}, 합집합 A ∪ B = {1, 2, 3, 4}이 되므로, 집합 A, B 사이의 자카드 유사도 J(A, B) = 2/4 = 0.5가 된다. 집합 A와 집합 B가 모두 공집합일 경우에는 나눗셈이 정의되지 않으니 따로 J(A, B) = 1로 정의한다.

자카드 유사도는 원소의 중복을 허용하는 다중집합에 대해서 확장할 수 있다. 다중집합 A는 원소 1을 3개 가지고 있고, 다중집합 B는 원소 1을 5개 가지고 있다고 하자. 이 다중집합의 교집합 A ∩ B는 원소 1을 min(3, 5)인 3개, 합집합 A ∪ B는 원소 1을 max(3, 5)인 5개 가지게 된다. 다중집합 A = {1, 1, 2, 2, 3}, 다중집합 B = {1, 2, 2, 4, 5}라고 하면, 교집합 A ∩ B = {1, 2, 2}, 합집합 A ∪ B = {1, 1, 2, 2, 3, 4, 5}가 되므로, 자카드 유사도 J(A, B) = 3/7, 약 0.42가 된다.

이를 이용하여 문자열 사이의 유사도를 계산하는데 이용할 수 있다. 문자열 FRANCE와 FRENCH가 주어졌을 때, 이를 두 글자씩 끊어서 다중집합을 만들 수 있다. 각각 {FR, RA, AN, NC, CE}, {FR, RE, EN, NC, CH}가 되며, 교집합은 {FR, NC}, 합집합은 {FR, RA, AN, NC, CE, RE, EN, CH}가 되므로, 두 문자열 사이의 자카드 유사도 J("FRANCE", "FRENCH") = 2/8 = 0.25가 된다.

**입력 형식**  

입력으로는 str1과 str2의 두 문자열이 들어온다. 각 문자열의 길이는 2 이상, 1,000 이하이다.
입력으로 들어온 문자열은 두 글자씩 끊어서 다중집합의 원소로 만든다. 이때 영문자로 된 글자 쌍만 유효하고, 기타 공백이나 숫자, 특수 문자가 들어있는 경우는 그 글자 쌍을 버린다. 예를 들어 ab+가 입력으로 들어오면, ab만 다중집합의 원소로 삼고, b+는 버린다.
다중집합 원소 사이를 비교할 때, 대문자와 소문자의 차이는 무시한다. AB와 Ab, ab는 같은 원소로 취급한다.

**출력 형식**

입력으로 들어온 두 문자열의 자카드 유사도를 출력한다. 유사도 값은 0에서 1 사이의 실수이므로, 이를 다루기 쉽도록 65536을 곱한 후에 소수점 아래를 버리고 정수부만 출력한다.

**예제 입출력**

|str1|str2|answer|
|:--:|:--:|:--:|
|FRANCE|french|16384
|handshake|shake hands|65536
|aa1+aa2|AAAA12|43690
|E=M*C^2|e=m*c^2|65536

# Solve
: 이해한대로, 그대로 시뮬레이션 하면 되는 문제이다.  
문제를 풀고 나서 보니 대부분의 사람들이 ArrayList로 풀었는데, 나는 **HashMap**을 이용했다.

## 접근방법
: 해결해야 하는 부분은 크게 두가지다.

1. 문제에서 요구하는 다중집합 만들기
2. 다중집합을 기반으로, 자카드 유사도 구하기

### 1. 다중집합
: 다중집합을 만들기는 크게 어렵지 않다. 다음의 순서를 따라가면 된다.

1. 문자열 전체를 대문자, 혹은 소문자로 만들어준다.(나는 소문자로 만들었다.)
2. 1씩 증가하면서, 2칸 단위의 temp String(=key)을 만든다.(e.g. ABC -> AB, BC)
3. 해당 temp String이 영문자로만 구성되어 있으면, HashMap에 추가한다.
4. 이 때, HashMap에 key가 이미 존재한다면, value를 1 증가시킨다.

즉, ABCBC 라는 문자열이 들어온다면,  

1. abcbc로 만듦.
2. ab, bc, cb, bc 로 나눔
3. 위 3, 4번에 따라서, map은 `{bc=2, ab=1, cb=1}`이 된다.

이렇게 map을 str1, str2 에 대해서 각각 만들어주면 된다.


### 2. 자카드 유사도 구하기
: 자카드 유사도를 구할 때에는, 교집합과 합집합을 각각 구해서,  
해당 집합들의 개수를 세면 된다.

#### 교집합
: 일반적인 집합에 대해서 교집합을 구하는 방법은 간단하지만,  
이 문제는 **다중집합** 이라는 것을 기억해야 한다.  
따라서, 두 map의 같은 원소의 **최소값** 이, 내가 **세야 할** 숫자이다.

위 입출력 예시에서, `aa1+aa2` 와 `AAAA12` 를 가져와보자.  
첫 번째 녀석의 map1은 `{aa=2}` 가 될 것이고,  
두 번째 녀석의 map2는 `{aa=3}` 이 될 것이다.

여기서 **다중집합의 교집합의 개수**는 `2`가 되어야 한다.  
만약 여기서 `{bb=1}` 과 `{bb=2}` 가 추가로 각각 있었다면,  
`1`을 골라서, `2+1`의 `3`이 최종 결과값이 된다.


여전히 이해가 되지 않는다면, 문제 설명의 내용을 다시 읽어보자.

---

> 다중집합 A = {1, 1, 2, 2, 3}, 다중집합 B = {1, 2, 2, 4, 5}라고 하면, 교집합 A ∩ B = {1, 2, 2}, 합집합 A ∪ B = {1, 1, 2, 2, 3, 4, 5}가 되므로, 자카드 유사도 J(A, B) = 3/7, 약 0.42가 된다.

---

#### 합집합
: 합집합의 경우에는, 교집합의 반대가 된다.     
위 `aa1+aa2` 와 `AAAA12` 예시를 다시 끌고오자면,  
내가 세야 할 녀석은 `2`가 아닌 `3`이 된다.

역시, 추가로 `{bb=1}` 과 `{bb=2}` 가 있었다면,  
세야 할 녀석은 `2` 이므로, `3+2`의 `5`가 된다.

---

그리고 위에서 구한 교집합의 개수와 합집합의 개수를 기반으로 자카드 유사도는 다음과 같다.

`{aa=2}`와 `{aa=3}` 인 경우 -> 2/3
`{aa=2, bb=1}`와 `{aa=3, bb=2}` 인 경우 -> 3/5

이 자카드 유사도에, 65536을 곱한 뒤 `Math.floor()` 함수를 통해 소수부를 날려주었다.

## 구현

### 다중집합
다중집합을 구하는 코드는 간단하다.

```java
public Map<String, Integer> createMap(String str) {
    Map<String, Integer> map = new HashMap<>();
    str = str.toLowerCase();

    for (int i = 2; i <= str.length(); i++) {
        String temp = str.substring(i - 2, i);
        if (isAlpha(temp)) map.compute(temp, (k, v) -> v == null ? 1 : v + 1);
    }

    return map;
}

public boolean isAlpha(String str) {
    return str.matches("^[a-z]*$");
}
```

적절히 for문을 돌면서, `temp` String을 만들어 주고,  
이 `temp`가 영문자로 구성된 경우에 한해서,   
첫 삽입이면 `1` 을, 아니라면 `기존값+1` 을 넣어주었다.


### 교집합
: 다음과 같이, stream을 이용하였다.

```java
public static int getIntersectionCount(Map<String, Integer> map1, Map<String, Integer> map2) {
    return map1.entrySet()
            .stream()
            .filter(x -> map2.containsKey(x.getKey()))
            .mapToInt(x -> Math.min(x.getValue(), map2.get(x.getKey())))
            .sum();
}
```

key가 공통인 경우에만 한해서 filtering 하고,  
각 value의 값을 구해서, 최소 값이 되는 녀석을 int로 mapping한 다음,  
이 intstream을 summation 해주었다.


### 합집합
: 합집합은, 진짜 합집합의 map을 만들어 주었다.

```java
    public static int getUnionCount(Map<String, Integer> map1, Map<String, Integer> map2) {
        Map<String, Integer> union = new HashMap<>(map1);
        map2.forEach((k, v) -> union.merge(k, v, (v1, v2) -> v1 > v2 ? v1 : v2));
        
        return union.values().stream().mapToInt(Integer::intValue).sum();
    }
```

합집합을 만들고 나서, stream을 통해 summation 해주었다.  
만약 교집합을 구할때 처럼 곧바로 stream을 쓸 경우,  
map1 에는 없으나 map2 에는 있는 원소에는 접근할 수 없다는 문제가 있다.


---


이렇게 두 개의 Count를 구하고 나서,   
위에서 언급한대로 적절히 값을 수정해서 return 하면 된다.

전체 코드는 다음과 같다.

```java
package programmers.lv2;

import java.util.HashMap;
import java.util.Map;

// 프로그래머스 - [1차] 뉴스 클러스터링
public class p17677 {
    public static void main(String[] args) {
        String str1 = "FRANCE";
        String str2 = "french";

        int result = solution(str1, str2);
        System.out.println(result);
    }

    public static int solution(String str1, String str2) {

        Map<String, Integer> map1 = createMap(str1);
        Map<String, Integer> map2 = createMap(str2);

        System.out.println("map1 = " + map1);
        System.out.println("map2 = " + map2);

        // 두 다중집합이 공집합인 경우, 자카드 유사도는 1로 판단.
        if (map1.isEmpty() && map2.isEmpty()) return 65536;

        int intersection = getIntersectionCount(map1, map2);
        int union = getUnionCount(map1, map2);

        System.out.println("intersection = " + intersection);
        System.out.println("union = " + union);

        return (int) Math.floor(((double) intersection / union) * 65536);
    }

    public static Map<String, Integer> createMap(String str) {
        Map<String, Integer> map = new HashMap<>();
        str = str.toLowerCase();

        for (int i = 2; i <= str.length(); i++) {
            String temp = str.substring(i - 2, i);
            if (isAlpha(temp)) map.compute(temp, (k, v) -> v == null ? 1 : v + 1);
        }

        return map;
    }

    public static boolean isAlpha(String str) {
        return str.matches("^[a-z]*$");
    }

    public static int getUnionCount(Map<String, Integer> map1, Map<String, Integer> map2) {
        Map<String, Integer> union = new HashMap<>(map1);
        map2.forEach((k, v) -> union.merge(k, v, (v1, v2) -> v1 > v2 ? v1 : v2));

        return union.values().stream().mapToInt(Integer::intValue).sum();
    }

    public static int getIntersectionCount(Map<String, Integer> map1, Map<String, Integer> map2) {
        return map1.entrySet()
                .stream()
                .filter(x -> map2.containsKey(x.getKey()))
                .mapToInt(x -> Math.min(x.getValue(), map2.get(x.getKey())))
                .sum();
    }

}
```