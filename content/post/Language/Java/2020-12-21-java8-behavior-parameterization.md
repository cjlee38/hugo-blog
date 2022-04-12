---
# author: cjlee
categories: java
# cover: /assets/covers/coding.png
date: "2020-12-21T14:33:00Z"
tags: ['null']
title: '# (Java8) 동작(≒메소드) 파라미터화 '
---

> 모던 자바 인 액션(라울-게이브리얼 우르마 저, 한빛미디어)을 기반으로 작성하였습니다.


# 0. 들어가며
: 스트림이나 람다 같은 프로그래밍 테크닉들을 사용하면서, "이게 어떻게 동작하지?" 라는 그 원리에 대해서는 전혀 생각을 해보지 못했었다. 모던 자바 인 액션을 보면서 이런 동작 원리에 대해서 새로운 시야가 트인 느낌이라, 각 내용을 내 입맛에 맞게 정리해보고자 한다. 그런 기념에서, 첫 번째 포스팅은 2장의 **동작(동적,Dynamic이 아니다.) 파라미터화 코드 전달하기** 이다.

# 1. 예제
: 가령 예를들어서, 과일 재고 목록을 관리하는 애플리케이션이 있다고 해보자. 그리고, 처음에는 "녹색 사과를 모두 찾고 싶어요" 라고 요구사항이 들어왔다. 

## step.1
이에 대한 메소드는, 지금까지는 아마 당연하게도 이런식으로 작성했을 것이다.
```java
public List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if ("GREEN".equals(apple.getColor())) {
            result.add(apple);
        }
    }

    return result;
}
```

그런데, 요구사항이 변경되어서, 녹색사과 뿐만이 아니라 빨간색 사과도 찾고 싶어졌다고 해보자. 

## step.2
빨간색 사과를 찾는, 위와 같은 형태의 `filterRedApples()` 메소드를 만드는 것은 꽤나 바보같은 짓일 것 같다. 그렇다면, 조금 더 현명하게 처리해서, `"GREEN"` 이라고 하드코딩되어 있는 부분을, 파라미터에 의해 결정되도록 수정할 수 있을 것이다.

```java
public List<Apple> filterApplesByColor(List<Apple> inventory, String color) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (apple.getColor().equals(color)) {
            result.add(apple);
        }
    }

    return result;
}
```

그런데, 이걸로 끝일까? 흔히 말해, **어떤 것이 와도 두렵지 않으려면,** 이정도 코드로는 아마 감당하기 힘들 것이다. 가령, 무게가 150g 이상인 사과를 찾고싶다면 어떻게 해야할까?

```java
public List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (apple.getWeight() > weight) {
            result.add(apple);
        }
    }

    return result;
}
```
이런 코드를 추가하는 것만으로 만족할 수 있을까? 게다가, if 조건문을 제외하면 나머지 코드가 색깔로 필터링하는 코드와 전부 동일하다는 것을 확인할 수 있다. 중복되는 코드를 지양해야 하는 개발자에게는, 이는 딱히 좋은 선택지로는 보이지 않는다.

## step.3

심지어, 다음과 같은 코드는 더욱 최악이다.
```java
public List<Apple> filterApples(List<Apple> inventory, String color, boolean flag) {
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory) {
        if (flag && apple.getColor().equals(color) ||
        (!flag && apple.getWeight() > weight)) {
            result.add(apple);
        }
    }

    return result;
}
```

이런 코드는 도대체 무엇을 의미할까? 게다가, 사용하는 입장에서도 이는 껄끄럽다.
```java
List<Apple> greenApples = filterApples(inventory, "GREEN", 0, true);
```

이 true와 false는 무엇을 의미하는걸까? 이 또한 알 수 없다. 

결국 지금까지 문제가 되었고, 우리가 개선해야 할 부분은 저 if문이고, 따라서 우리가 원하는 것은 **요구사항을 밖에서 전달하는 것** 이다. 그렇다면 요구사항을 밖에서 전달하기 위해, 전략패턴을 사용할 수 있다. 

## step.4

이를 위해, **프레디케이트를 갖는 인터페이스를 만들어서 이를 활용해보자.**

---

> Note. 참 또는 거짓을 반환하는 메소드를 Predicate라 칭한다.

---

```java
public interface ApplePredicate {
    boolean test(Apple apple);
}
```

그리고, 이를 구현하는 클래스를 만들어보자.

```java
// 무게가 150 초과면 참, 아니면 거짓.
public class AppleHeavyWeightPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}

// 사과의 색깔이 초록색이면 참, 아니면 거짓
public class AppleGreenColorPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return "GREEN".equals(apple.getColor());
    }
}
```

그리고, 이에 맞게 기존의 filter 메소드를 수정해보자.

```java
public List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory) {
        if(p.test(apple)) result.add(apple);
    }

    return result;
}
```

기존 코드와의 차이점이 보이는가? 우린 앞으로 어떤 조건이 주어지더라도, 저 `filterApples()` 라는 메소드는 **전혀 고칠 필요가 없다.** 새로운 요구사항이 주어지는 대로, `ApplePredicate`를 구현하는 새로운 클래스를 만들어서 그 인스턴스를 파라미터로 전달해주면 된다. 가령 다음과 같다.

```java
AppleGreenColorPredicate p = new AppleGreenColorPredicate();
List<Apple> greenApples = filterApples(inventory, p);
```

그런데 위 코드는, 다음과 같이 수정해볼 수 있다.

```java
List<Apple> greenApples 
        = filterApples(inventory, new AppleGreenColorPredicate());
```

이러한 코드 작성은 우리가 아주 자연스럽게 해왔던 것이다. `p` 라는 녀석이 앞으로 재활용될 가능성이 없다면, 변수를 할당할 필요 없이 곧바로 메소드의 인자로 인스턴스를 넘겨주는것. 같은 말을 또 하자면, **우리는 인스턴스를 넘겨주었다.** 

## step.5

인스턴스를 넘겨주었다는 사실에 기반해, 우리는 또 하나의 아이디어를 떠올릴 수 있다. 그것은 바로 **익명 클래스**이다. 익명 클래스는 말 그대로, 이름이 없는 클래스이다. 그리고 이 녀석은, 클래스의 선언과 인스턴스화가 **동시에 수행된다.**

우리는 `ApplePredicate` 를 구현하는 `AppleGreenColorPredicate` 라는 **이름의** 클래스를 만들었다. 그렇다면, 익명 클래스라는 녀석은 ApplePredicate를 구현하지만, 위와 같은 **이름이 존재하지 않는 클래스**를 말하는 것이다. 어떻게 생겼는가 하면, 다음과 같이 생겼다.

```java
List<Apple> greenApples = filterApples(inventory, new ApplePredicate() {
    public boolean test(Apple apple) {
        return "GREEN".equals(apple.getColor());
    }
})
```

보다시피, `ApplePredicate` 를 구현하는 클래스이긴 하지만, 어떤 이름을 갖지는 않는다. 이와 더불어, 기존의 전략패턴에서 사용했던 클래스들이 **만약 일회용으로 사용되고 버려질 경우 생기는** 코드의 낭비에 대해서도 나름대로 대처할 수 있다.

## step.6
그런데, 여기서도 아직 불만족스럽다. 개발자란 종족은 끝없이 편의성을 추구하고, 쓸데없는 짓을 하면 몸에 두드러기가 난다. 다시 위 코드를 보자. `new ApplePredicate()` 라는 코드, 그리고 `public boolean test(Apple apple)` 이라는 코드는 꼭 필요한가? 익명 클래스를 여러 개 사용한다면, 딱히 중요해보이지는 않는데 저 코드를 또 써야 한다.

똑똑한 intellij 또한 `Anonymous new ApplePredicate() can be replaced with lambda` 라는 안내 문구를 보여준다. 따라서 이를 람다로 수정하면 다음과 같이 수정할 수 있다.


```java
List<Apple> greenApples 
        = filterApples(inventory, apple -> "GREEN".equals(apple.getColor()));
```

## step.7
마지막으로는, 사과가 아니라, 오렌지 등의 다른 경우에 대처할 수 있도록 업그레이드 해보자. 즉, 메소드 이름이 `filterApples()` 가 아니라 `filter`가 되는 것이다. 특별할건 없고, 제네릭을 이용하기만 하면 된다.

Predicate 인터페이스와 filter 메소드를 다음과 같이 수정하면 된다.

```java
public interface Predicate<T> {
    boolean test(T t);
}

public <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> result = new ArrayList<>();
    for(T e : list) {
        if(p.test(e)) {
            result.add(e);
        }
    }
    
    return result;
}
```

그리고, 사용할 때는 다음과 같이 사용하면 된다.
```java
// 사과
List<Apple> greenApples = filter(appleInventory, apple -> "GREEN".equals(apple.getColor()));

// 오렌지
List<Orange> heavyOranges = filter(orangeInveotry, orange -> orange.getWeight() > 200);
```

# 2. 생각해볼 점.
: 이렇게 각 단계별로 코드가 유연함과 간결함의 두 마리 토끼를 잡을 수 있도록 발전해나가면서, 요구사항에 우아하게 대처할 수 있는 모습을 볼 수 있었다. 그런데, 단순히 이렇게 감탄하고 끝날 것이 아니라, 이 과정 속에서 한 가지 거대한 격변이 있었음을 기억해야 한다.

프로그래밍에서 가장 핵심이 되는 것은, **값을 변경하는 것**이다. Query가 아닌 Command의 특성을 갖는 메소드에서, 기존에 우리는 **변화할 수 있는 값**을 넘겨주었다. int, double 등의 primitive type의 데이터부터 시작해서, 객체(인스턴스) 또한 변화할 수 있는 값이다. 그리고 이런 녀석들을 우리는 **일급 값** 이라고 부른다.

그리고 이렇게 인자로 전달할 수 없는 녀석은 **이급 값**이라고 부른다. 무엇이 해당할까? 바로 **클래스와 메소드**이다.

이전에는, 우리는 다음과 같은 코드는 본 적이 없다.
```java
List<Apple> result = filter(inventory, public boolean my_function(...) {
    // do something
})
```

메소드의 인자로 메소드를 넘기는 경우는 본 적이 없다. 그런데, step.6 를 보면, 람다라는 **익명 함수**를 넘겨줄 수 있다. 이는 기존의 **이급 값**이었던 메소드를, **일급 값으로 만들 수 있게 되었음을 의미**한다!

사실, step.6 에서 **어떻게 함수를 파라미터로 받을 수 있는가?** 에 대해서는 다루지 않았는데, 이는 다음 장의 **람다 표현식의 원리**에 대해서 자세히 다뤄야할 내용이라 생략했다. 역시, 다음 포스팅에서 마저 알아보기로 하자. 

이상으로 포스팅을 마칩니다.