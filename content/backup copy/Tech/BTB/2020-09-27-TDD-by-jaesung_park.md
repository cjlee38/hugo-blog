---
# author: cjlee
categories: btb
# cover: /assets/covers/coding.png
date: "2020-09-27T21:31:00Z"
tags: ['null']
title: '# 박재성 - 의식적인 연습으로 TDD, 리팩토링 연습하기'
---

---


> 본 포스팅은 자바지기 박재성님의 강의를 듣고 정리하면서 작성한 글입니다.  
> 발표 동영상과 슬라이드는 하단의 Reference를 참고해주세요.

---

# 의식적인 연습
: 단순히 반복적인 연습을 함으로써 실력 향상을 기대하기는 어렵다.  
**"의식적인"** 연습이 필요하다.

## 의식적인 연습의 7가지 원칙.
1. 효과적인 훈련 기법이 수립되어 있는 기술 연마
2. 개인의 **"컴포트 존"을 벗어난 지점에서 진행, 자신의 현재 능력을 살짝 넘어가는 작업을 지속적으로 시도**
3. **명확하고 구체적인 목표**를 가지고 진행
4. 신중하고 계획적이다. 즉, 개인이 온전히 집중하고 '의식적'으로 행동할 것을 요구
5. **피드백과 피드백에 따른 행동 변경을 수반**
6. 효과적인 심적 표상을 만들어내는 한편으로 심적 표상에 의존
7. **기존에 습득한 기술의 특정 부분을 집중적으로 개선함으로써 발전시키고 수정**하는 과정을 수반.


# 연습 방법

## 1단계 : 단위 테스트 연습
: (TDD와 단위테스트를 만드는 것은 다르다.)내가 사용하는 API 사용법을 익히기 위한 학습 테스트에서 시작해라.
- 자바 String 클래스의 다양한 메소드 사용법
- 자바 ArrayList에 데이터를 추가,수정,삭제하는 방법

**Example**
```java
import static org.assertj.core.api.Assertion.asserThat;

public class StringTest {
    @Test
    public void split() {
        String[] values = "1".split(",");
        assertThat(values).contains("1");

        values = "1,2";
        assertThat(values).containsExactly("1", "2");
    }

    @Test
    public void substring() {
        String input = "(1,2)";
        String result = input.substring(1, input.length() -1);
        assertThat(result).isEqualsTo("1,2");
    }
}
```

**API 이외에는?**
- 내가 구현하는 메소드 중,   
  **Input과 Output이 명확한 클래스 메소드(보통 Util 성격의 메소드)**  
  에 대한 단위 테스트 연습


**효과**
- 단위테스트 방법을 학습할 수 있다.
- 단위테스트 도구(jUnit)의 사용법을 익힐 수 있다.
- 사용하는 API에 대한 학습 효과가 있다.

## 2단계 : TDD 연습
### 지켜야 할 원칙
- 회사 프로젝트가 아닌, 토이 프로젝트에 연습하자.
- 웹, 모바일UI 혹은 DB에 의존관계를 가지지 않은 요구사항으로 연습한다.

### Example
: 쉼표(,) 또는 콜론(:)을 구분자로 가지는 문자열을 전달하는 경우,  
구분자를 기준으로 분리한 각 숫자의 **합**을 반환

|입력(Input)|출력(Output)|
|:--:|:--:|
|null 또는 ""|0|
|"1"|1|
|"1,2"|3|
|"1,2:3"|6|

---

> Note. TDD LifeCycle  
![TDD_lifecycle](/assets/images/2020-09-27-23-31-51_2020-09-27-TDD-by-jaesung_park.md.png){: .alignCenter}
> 실패하는 테스트를 먼저 만들고, 컴파일을 성공시키고, 테스트를 패스하기 위해 Production Code(실제 동작하는 코드)를 만들고, 마지막에 Refactoring.  
> 한번에 다 연습하지 말고, Test Fails와 Test Passes까지만 해보자.

---

**Test Code Example**
```java
public class StringCalculatorTest {
    @Test
    public void nullOrEmpty() {
        assertThat(StringCalculator.splitAndSum(null).isEqualTo(0));
        assertThat(StringCalculator.splitAndSum("").isEqualTo(0));
    }

    @Test
    public void singleValue() {
        assertThat(StringCalculator.splitAndSum("1").isEqualTo(1));
    }

    @Test
    public void delimiter_comma() {
        assertThat(StringCalculator.splitAndSum("1,2").isEqualTo(3));
    }

    @Test
    public void delimiter_comma_colon() {
        assertThat(StringCalculator.splitAndSum("1,2:3").isEqualTo(6));
    }
}
```

**Production Code Example**
```java
public class StringCalculator {
    public static int splitAndSum(String text) {
        int result = 0;
        if (text == null || text.isEmpty()) {
            result = 0;
        } else {
            String[] values = text.split(",|:");
            for (String value : values) {
                result += Integer.parseInt(value);
            }
        }

        return result;
    }
}
```

## 3단계 : Refactoring 연습
: 위 코드를, 어떻게 Refactoring 할 수 있을까?  
Test Code는 변경하지 말고, Production Code를 개선하는 연습을 한다.  
이를 Refactoring 하려고 하면, 막막하지만, 앞서 언급한  
**"의식적인 연습 7가지 원칙"** 을 떠올리자

- **명확하고 구체적인 목표**를 가지고 진행
- **피드백과 피드백에 따른 행동 변경을 수반**

Plus, 정성적인 기준보다는 정량적이고 측정 가능한 방법으로 연습하자. How?

## <u>A. 메소드 분리</u>
### Method Point. 1 : 한 메소드에 오직 한 단계의 들여쓰기(indent)만 한다.
위 Production Code에서 두 단계의 indent가 있는 곳은  
```java
for (String value : values) {
    result += Integer.parseInt(value);
}
```
이 부분의 `result +=` 부분임.

따라서, for loop 부분을 `result = sum(values)` 로 **대체**하고,  
이에 걸맞는 메소드 작성!

```java
private static int sum(String[] values) {
    int result = 0;
    for (String value : values) {
        result += Integer.parseInt(value);
    }
    return result;
}
```

### Method Point. 2 : else Keyword를 쓰지 않는다.
: if statement 안에, `result = 0;` 을 `return 0;` 로 대체함.  
`result = 0;`이 남아 있다면, return이 나올때까지 **코드를 직접 눈으로 봐야함.**
그렇다면 다음과 같은 모습이 됨.

```java
public static int splitAndSum(String text) {
    if (text == null || text.isEmpty()) {
        return 0;
    }

    String[] values = text.split(",|:");
    return sum(values);
}
```

### Method Point. 3 : 메소드가 한 가지 일만 하도록 구현하기.
: 현재 sum() 메소드는 두 가지 일을 하고 있음   
1. 문자열을 숫자로 변환하기
2. 숫자의 합을 계산하기

따라서, `toInts()` 라는 이름의 메소드를 만든다

```java
private static int[] toInts(String[] values) {
    int[] numbers = new int[values.length];
    for (int i = 0; i < values.length; j++) {
        numbers[i] = Integer.parseInt(values[i]);
    }

    return numbers;
}
```

따라서, `sum()` 함수도 parameter를 `String[]` 이 아닌 `int[]` 로 받게 한다.

```java
private static int sum(int[] numbers) {
    int result = 0;
    for (int number : numbers) {
        result += number;
    }

    return result;
}
```

이렇게 하고 나면, `splitAndSum()` 함수는 다음과 같다.

```java
public static int splitAndSum(String text) {
    if (text == null || text.isEmpty()) {
        return 0;
    }

    String[] values = text.split(",|:");
    int[] numbers = toInts(values);
    return sum(numbers);
}

```

### Method Point. 4 : 필요없는 로컬변수 제거
이때, `values`와 `numbers` 라는 **로컬 변수가 정말 필요한가?**  
필요하지 않다면, 다음과 같이 한 줄에 작성한다.

```java
return sum(toInts(text.split(",|:")));
```

### Method Point. 5 : Compose Method 패턴 적용
: 메소드의 의도가 잘 드러나도록 **동등한 수준의 작업**을 하는 여러 단계로 나눈다.  

**Like This.**
```java
public class StringCalculator {
    public static int add(String text) {
        if (isBlank(test)) {
            return 0;
        }

        return sum(toInts(split(text)));
    }

    private static boolean isBlank(String text) { ... }
    private static String[] split(String text) { ... }
    private static int[] toInts(String[] values) { ... }
    private static int sum(int[] numbers) { ... }
}

이전 코드와 비교해보자.

```java
public class StringCalculator {
    public static int splitAndSum(String text) {
        int result = 0;
        if (text == null || text.isEmpty()) {
            result = 0;
        } else {
            String[] values = text.split(",|:");
            for (String value : values) {
                result += Integer.parseInt(value);
            }
        }

        return result;
    }
}
```

처음 읽는 사람에겐, `add()` 메소드는(이전 코드에서는 `splitAndSum()`)   **Refactoring 된 경우가 더 보기 좋다.**

이러한 Refactoring을, **한번에 하나씩 하자.**   
이 외에도 다른 연습 방법? : 15 Line의 코드를 10 Line으로 줄여보자.

이러한 극단적인 연습을 하면, **"설계에 대한 Insight가 보인다."**  
또한, 이러한 연습이 가능한 이유는? **테스트가 뒷받침해주기 때문에.**

## <u> B. 클래스 분리</u>

(조건 추가) : 위 로직에서, 만약 **음수를 전달하는 경우**  
**RuntimeException을 Throw 한다.**

**Test Code Example(Plus)** 
```java
public class StringCalculatorText {
    
    ...

    @Test(expected = RuntimeExcept)
    public void negative() {
        StringCalculator.splitAndSum("-1,2:3");
    }
}
```

**Production Code Example(Plus)**
```java
public class StringCalculator {
    
    ...

    private static int[] toInts(String[] values) {
        int[] numbers = new int[value.length];
        for (int i = 0; i < values.length; i++) {
            numbers[i] = toInt(values[i]);
        }

        return numbers;
    }
    private static int toInt(String value) {
        int number = Integer.parseInt(value);
        if (number < 0) {
            throw new RuntimeException();
        }
        return number;
    }
}
```

### Class Point. 1 : 모든 Primitive Type과 String을 포장(Wrap)한다.
즉, 새로운 Class를 만든다.

```java
public class Positive {
    private int number;
    
    public Positive(String value) {
        this(Integer.parseInt(value));
    }
    public Positive(int number) {
        if (number < 0) {
            throw new RuntimeException();
        }
        this.number = number;
    }
}

```

이에 맞춰서, 다른 코드들도 **Production Code**도 수정하면 된다.

```java
private static Positive[] toInts(String[] values) {
    Positive[] numbers = new Positive[values.length];
    for (int i = 0; i< values.length; i+) {
        numbers[i] = new Positive(values[i]);
    }
    return numbers;
}

private static int sum(Positive[] numbers) {
    Positive result = new Positive(0);
    for (Positive number : numbers) {
        result = result.add(number);
    }

    return result.getNumber();
}
```

이에 더해, Positive Class의 add와 getNumber를 구현한다.

```java
public class Positive {
    
    ...

    public Positive add(Positive other) {
        return new Positive(this.number + other.number);
    }

    public int getNumber() {
        return number;
    }
}
```
### Class Point. 2 : 1급 Collection을 쓴다
: 1급 컬렉션이란? 
- 하나의 Collection을 유일한 Field로 갖고 있는 클래스.
- Collection의 불변성을 보장(내부의 값도 변경 불가능)
- 상태와 행위를 한 곳에서 관리.

### Class Point. 3 : 3개 이상의 instance 변수를 가진 class를 쓰지 않는다.
- 즉, Class의 Field가 3개 이상을 넘어가면 안된다.

---

> Note. 위 Class point 2, 3에 대한 내용은 다음의 링크 참조  
> [일급 컬렉션 (First Class Collection)의 소개와 써야할 이유](https://jojoldu.tistory.com/412)

---


## 4단계 : 토이 프로젝트 난이도 높이기
: 연습하기 좋은 프로그램의 요구사항들
- 게임과 같이 요구사항이 명확한 프로그램으로 연습
- 의존관계(모바일 UI, 웹 UI, 데이터베이스, 외부 API와 같은 의존관계)가 없이 연습
- 약간은 복잡한 로직이 있는 프로그램

**example**
- 로또
- 사다리 타기
- 볼링 게임 점수 판
- 체스 게임
- 지뢰 찾기 게임
(단, UI는 콘솔)

**앞에서 지켰던 기준을 지키면서 프로그래밍 연습을 하자.**

## 5단계 : 의존관계 추가를 통한 난이도 높이기
: 웹, 모바일 UI, 데이터베이스와 같은 의존관계를 추가

**이 때 필요한 역량**
- 테스트하기 쉬운 코드와 어려운 코드를 보는 **눈**
- 테스트하기 어려운 코드를, 테스트 하기 쉬운 코드로 설계하는 **감각**(sense)


## Plus Alpha
: 한 단계 더 나아간 연습을 하고 싶다면?
- 컴파일 에러를 최소화하면서 리팩토링하기
- ATDD 기반으로 응용 애플리케이션 개발하기
- 레거시 애플리케이션에 테스트 코드 추가해 리팩토링하기

: TDD, 리팩토링 연습을 위해 필요한 것은?
- 조급합 대신 마음의 여유
- 나만의 토이 프로젝트
- 같은 과제를 반복적으로 구현할 수 있는 인내력
  

## 마무리
: 하드웨어 비용보다 사람이 더 비싸기 때문에,  
**사람이 읽기 좋은 코드를 만드는 것이 중요하다.**

## 질의응답
Q1. Comfort Zone을 극복하는 노하우는 무엇일까요?

A1. 시간, 에너지가 있어야 한다. 이를 만들어야 한다.  
(농담) 친구와 여자친구를 만나는 빈도수를 줄여라.  
1-2년은 과도기가 필요하다. 

Q2. 요구사항이 변경되었을 때, 기존의 테스트 코드가 발목을 잡는 경우에 어떻게 해야 할까요?

A2. 본인도 그런 경험을 많이 해봤다. 이는 요구사항의 변경이 문제가 아니라,   
테스트코드를 잘못 작성했을 경우, 설계가 잘못되었을 경우 발생하는 문제다.  
시간이 없으면, 할 수 있는 만큼만 하거나, 혹은 데드라인의 연장을 요구하자.



### Reference
[유튜브 링크](https://www.youtube.com/watch?v=cVxqrGHxutU&ab_channel=OKKY)  
[Slide](https://www.slideshare.net/OKJSP/okkycon-tdd)