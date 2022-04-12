---
# author: cjlee
categories: java
# cover: /assets/covers/coding.png
date: "2020-10-10T20:41:00Z"
tags: ['null']
title: '# (Java) Static Field, Method, 그리고 Block'
---

# Concept of Static
: static을 한글로 해석하면 정적이라는 뜻인데, 크게 와닿지는 않는 해석이다.

흔히, 객체지향에서 자주 하는 비유 중 하나가 붕어빵 틀과 붕어빵이다.  
클래스는 붕어빵 틀로, 붕어빵을 찍어낼 수 있고, 인스턴스는 찍혀진 붕어빵이다.  
따라서, 찍힌 붕어빵은 서로 다른 객체로 인식해야한다.

그러나, 붕어빵을 만들기 위해 사용하는 **반죽**은 어떨까?   
붕어빵 장사를 하는 사람이, 아주 기가막힌 비율의 반죽을 만들어내서,  
*(말은 안되지만)* 이미 만들어진, 그리고 앞으로 만들 모든 붕어빵의 반죽을  
new 반죽으로 바꾸고 싶다고 해보자. 

모든 instance의 반죽을 하나하나 전부 변경해야할까?  
이를 instance로는 해결하려 하기 보다는, 모든 객체가 바라보는 class 자체에 박아넣고,   
이를 수정하는 것이 나을 것이다.

다른 비유를 들자면 **Class계의 공지사항** 정도가 될 수 있을 것 같다.

# Static Field
: 붕어빵 예시를 코드로 한번 작성해보자.

```java
class 붕어빵 {
    private String 반죽;
    private String 속재료;
    private String 크기;

    public 붕어빵(String 반죽, String 속재료, String 크기) {
        this.반죽 = 반죽;
        this.속재료 = 속재료;
        this.크기 = 크기;
    }
    // Getter, Setter, toString 메소드 생략
}

public class Main {
    public static void main(String[] args) {
        붕어빵 b1 = new 붕어빵("기본", "팥", "중간");
        붕어빵 b2 = new 붕어빵("기본", "슈프림", "중간");

        System.out.println(b1);
        // 붕어빵{반죽='기본', 속재료='팥', 크기='중간'}
        System.out.println(b2);
        // 붕어빵{반죽='기본', 속재료='슈프림', 크기='중간'}
    }
}
```


여기서 `"기본"` 이라는 반죽이 `"혁신"` 이라는 혁신적인 반죽으로 바뀌어야 한다면 어떨까?

```java
b1.set반죽("혁신");
b2.set반죽("혁신");
...
```
벌써부터 끔찍하다.

하지만, 반죽 앞에 static 이라는 키워드를 붙일 수 있다.  
(`private String 반죽` -> `static String 반죽`)  
이 때, 생성자와 Setter에 다음과 같은 Warning이 나온다.

`Static member 'com.company.붕어빵.반죽' accessed via instance reference `

따라서, 생성자와 Setter를 다음과 같이 수정하면 된다.

```java
public 붕어빵(String 반죽, String 속재료, String 크기) {
    붕어빵.반죽 = 반죽; // 혹은 
    this.속재료 = 속재료;
    this.크기 = 크기;
}

public void set반죽(String 반죽) {
    붕어빵.반죽 = 반죽;
}
```

다시 Main Method로 돌아가서, 한줄만 추가해서 실행해보자.
```java
public class Main {
    public static void main(String[] args) {
        붕어빵 b1 = new 붕어빵("기본", "팥", "중간");
        붕어빵 b2 = new 붕어빵("기본", "슈프림", "중간");

        b2.set반죽("혁신");

        System.out.println(b1);
        // 붕어빵{반죽='혁신', 속재료='팥', 크기='중간'}
        System.out.println(b2);
        // 붕어빵{반죽='혁신', 속재료='슈프림', 크기='중간'}
    }
}
```

b2를 통해서 실행했을 뿐인데, b1또한 혁신으로 바뀌는 것을 볼 수 있다.

# Static Method
: 그런데, 코드를 다시 보면 이상한 점이 느껴진다.  
모두 `"혁신"` 으로 잘 바뀌기는 했는데,   
b2를 통해서 `set반죽()` 메소드를 호출하는 것이 어울릴까?    
그렇지는 않아보인다.  

잠깐 다른 이야기를 하자면, 우리가 출력을 위해 사용하는
`System.out.println()` 이라는 메소드는,   
우리가 **instance를 만들어서** 실행하지 않는다.  
그저 곧바로 System.out.println() 을 실행할 수 있다.  

이렇게 사용할 수 있는 이유는, println() 이라는 메소드가 static method 이었기 때문이다.

따라서, `set반죽()` 메소드 또한 static으로 바꾸어서,  
(`public void set반죽()` -> `public static void set반죽()`)  
**클래스로 접근해 사용할 수 있도록 만들면 좋겠다**는 생각이 든다.

`b2.set반죽("혁신");` 을 `붕어빵.set반죽("혁신");` 로 바꿔서 실행해보자.

```java
public class Main {
    public static void main(String[] args) {
        붕어빵 b1 = new 붕어빵("기본", "팥", "중간");
        붕어빵 b2 = new 붕어빵("기본", "슈프림", "중간");

        // b2.set반죽("혁신");
        붕어빵.set반죽("혁신");

        System.out.println(b1);
        // 붕어빵{반죽='혁신', 속재료='팥', 크기='중간'}
        System.out.println(b2);
        // 붕어빵{반죽='혁신', 속재료='슈프림', 크기='중간'}
    }
}
```

똑같은 결과가 나타나는 것을 볼 수 있다.

# Static Block
: C++과 다르게,   
자바는 클래스의 static 초기화에 사용될 수 있는 static block,   
혹은 static clause 라고 불리는 특별한 block을 지원한다.

지금껏 우리는 생성자, 혹은 set반죽() 메소드를 통해서,  
반죽을 결정할 수 있었다.

하지만, 붕어빵의 반죽이라는 것은, 붕어빵을 **만들 때**  결정되는 것이 아니고,   
붕어빵 장사를 하는 사람이 **집에서부터**  만들어서 준비를 해놓는다.  
다시 말해, 반죽은 붕어빵에 종속되지 않는다.

그러나, 지금 코드는 붕어빵이 있어야, 그제서야 붕어빵의 반죽이 결정되는 꼴과 다름없다.

잠깐의 테스트를 위해, `반죽` 변수의 접근제어자를 public으로 바꾸고,   
`System.out.println(붕어빵.반죽)`을 가장 먼저 호출해보면,  
null이 return 되는 것을 볼 수 있다.  
이를, `"기본"` 이라는 문자열로 초기화 될 수 있도록 해보자.

**붕어빵의 생성과 관계 없이 반죽은 미리 결정되어야 하므로,** 생성자에서도 빼버리자.

그리고 나면, 붕어빵의 반죽을 초기화 할 수 있는 방법은 총 세가지가 된다.

1. 처음 변수를 선언할 때, 정의하기
2. 처음 변수를 선언할 때, private static method를 콜 해서, 그 결과값을 받기
3. 선언해놓고, Static Block 이용하기

1번의 방법은, 초기화가 복잡한 경우에는 쓰고싶어도 쓸 수 없는 방법이다.   
(e.g. 1부터 100까지의 숫자가 담긴 ArrayList를 받아야 한다면?)

2번과 3번은 거의 유사하다. 다음의 예제를 보자.

Case. 2
```java
class 붕어빵 {
    static public String 반죽 = 반죽만들기();
    // 중간 생략
    private static String 반죽만들기() {
        System.out.println("static method called");
        return "기본";
    }
}
```

Case. 3
```java
class 붕어빵 {
    static public String 반죽;
    // 중간 생략
    static {
        System.out.println("static block called");
        반죽 = "기본";
    }
}
```

Main
```java
public class Main {
    public static void main(String[] args) {
        System.out.println(붕어빵.반죽);
        // static method called 또는 static block called
        // 기본
    }
}
```

이렇게 단순한 경우에는 거의 차이가 없지만,  
여러 개의 static 변수를 초기화해야하고, 이 변수들이 **의존적**이라면,  
static block을 쓰는것이 훨씬 편할 것이다.

가령, 위 예시에서, `String 반죽2` 는 기본 반죽이 두개,   
즉 `기본기본`을 만들고 싶다고 해보자.

Case. 2
```java
class 붕어빵 {
    static public String 반죽 = 반죽만들기();
    static public String 반죽2 = 반죽만들기2(반죽);

    private static String 반죽만들기() {
        System.out.println("static method called");
        return "기본";
    }

    private static String 반죽만들기2(String 반죽) {
        System.out.println("static method 2 called");
        return 반죽 + 반죽;
    }
}
```

Case. 3
```java
class 붕어빵 {
    static public String 반죽;
    static public String 반죽2;

    static {
        System.out.println("static block called");
        반죽 = "기본";
        반죽2 = 반죽 + 반죽;
    }
}
```

Main
```java
public class Main {
    public static void main(String[] args) {
        System.out.println(붕어빵.반죽2);
        // static method called , static method2 called 
        // 또는 static block called
        // 기본기본
    }
}
```

2번의 방법을 사용하나, 3번의 방법을 사용하나 차이는 거의 없지만,  
상황에 따라 편리한 방법을 골라서 사용하는 것이 좋다고 본다.


# 사용 시 주의점
: static의 붙은 변수는, 메모리의 static 영역에 할당되어서, **프로그램이 끝날 때 까지 생존한다.**  
다시 말해, Garbage Collection의 대상이 되지 않는다.  
그렇기에, static 변수를 남발하는 것은 좋은 선택은 아니다.

> 이에 더해, [스택오버플로우의 한 질문글](https://stackoverflow.com/questions/7026507/why-are-static-variables-considered-evil%22) 에서는, static 키워드를 사용하는 것 **자체**를 금기시 하고 있다.  
> 한줄 요약하면, static 사용은  **객체지향적이지 않다.** 라는 것이 요지이다.


## Reference
- [static block vs private static method for static member initialization](https://stackoverflow.com/questions/11618551/static-block-vs-private-static-method-for-static-member-initialization)
- [Why are static variables considered evil?](https://stackoverflow.com/questions/7026507/why-are-static-variables-considered-evil%22)
- [Static blocks in Java](https://www.geeksforgeeks.org/g-fact-79/)
- [[Java] static method, static block](https://onsil-thegreenhouse.github.io/programming/java/2017/11/12/java_static_method_block/)