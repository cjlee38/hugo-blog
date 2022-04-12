---
# author: cjlee
categories: java
# cover: /assets/covers/coding.png
date: "2020-10-25T01:52:00Z"
tags: ['null']
title: '# (Java) Parameter를 활용하는 방법 ( feat. varargs, kwargs )'
---

# varargs
: varargs 는 variable arguments의 약자로, 한국어로는 "가변인자" 로 해석된다.  
이는 지난번에 포스팅했던 [파이썬의 Asterisk(*) 다루기](https://cjlee38.github.io/python/asterisk_of_python) 와 비슷한데,  
arguments를 몇 개를 받을지 지정하지 않고, 호출하는 쪽에서 정해주겠다는 의미가 된다.

다음의 예시를 보자.  
구현하고자 하는 코드는, 입력받은 여러 개의 문자열을 하나씩 출력하고자 한다.

**가변인자를 사용하지 않는 방식**
```java
public class Main {
    public static void main(String[] args) {
        Main main = new Main();
        String[] strs = {"hello", "world", "java"};
        main.foo1(strs);
    }

    public void foo1(String[] strs) {
        for (String str : strs) {
            System.out.println("str = " + str);
        }
    }
}

// 결과
// str = hello
// str = world
// str = java
```

정상적으로 잘 동작하는 모습을 볼 수 있다.  
그러나, 이렇게 작성하지 않고, 가변인자를 사용하면 다음과 같다.

**가변인자를 사용하는 방식**
```java
public class Main {
    public static void main(String[] args) {
        Main main = new Main();
        main.foo2("hello", "world", "java");
    }

    public void foo2(String ... strs) {
        for (String str : strs) {
            System.out.println("str = " + str);
        }
    }
}
// 결과
// str = hello
// str = world
// str = java
```

`foo2()` 라는 메소드를 호출할 때, Array가 아닌 3개의 문자열을 따로 전달해주었다.  
그리고 이를 받는 메소드 `foo2()`는 `...` 표시를 통해 배열이 아닌 varargs를 받도록 했다.

코드는 변하였지만, 결과는 기존의 방식과 차이가 없다.  
그렇다면 어떤 효용성이 있을까?

#### 첫 번째로는, **formatting**에 있다.  
`System.out.printf()` 혹은, `String.format()` 이 이러한 방식으로 동작한다.

```java
String[] strs = {"hello", "world", "java"};
String sout = String.format("numOfStrings = %d, first String = %s ", strs.length, strs[0]);
System.out.println(sout);
```

위 코드에서, sout의 인자 전달을 보면,  
첫 번째로는 ""로 작성된 `String literal`을,   
두 번째로는 `strs.length` 라는 숫자를,  
세 번째로는 `strs[0]`, 즉 `"hello"` 를 넘겨주었다.  

뒤에 몇 개를 전달하더라도,   
앞 literal의 이스케이프 시퀀스와 짝을 맞춰주면 정상적으로 동작한다.

내부는 다음과 같이 작성되어 있다.

```java
public static String format(String format, Object... args) { 
    return new Formatter().format(format, args).toString();
}
```

#### 두 번째로는, Arrays.asList()가 있다.  

배열을 List로 전환하는 가장 쉬운 방법은,  
루프를 돌면서 하나씩 더하는 것이다.

```java
String[] strs = {"hello", "world", "java"};
List<String> list = new ArrayList<>();

for(String str : strs) {
    list.add(str);
}
```

그러나, 좀 더 심플하게 넣는 방법은 `Arrays.asList()` 메소드를 활용하는 것이다.

```java
String[] strs = {"hello", "world", "java"};
List<String> list = new ArrayList<>(Arrays.asList(strs));
```

한 발 더 나가서,  
이 `Arrays.asList()` 메소드 또한, varargs를 받기 때문에,   
다음과 같이 작성해도 무리가 없다.

```java
List<String> list = new ArrayList<>(Arrays.asList("hello", "world", "java"));
```

**이 때**, 눈치가 빠르면 알겠지만,   
`Arrays.asList()`를 첫 번째 방식과, 두 번째 방식으로 사용할 때,   
**넘겨준 인자가 서로 다르다.**

첫 번째 녀석은 `strs` 라는 배열을 넘겨줬지만, 두 번째는 3개의 문자열을 넘겨주었다.

이것이 가능한 이유는, 자바 컴파일러가 `...` 라는 syntax를 보고,  
이를 array로 바꾸어주기 때문이다.  

||원본|컴파일 이후|
|:--:|:--:|:--:|
|메소드|public void method(String ... args)|public void method(String[] args)|
|호출|method("hello", "world");|method(new String[]{"hello", world"});|


varargs 를 사용할 때 주의할 점은,  
이를 파라미터의 가장 마지막에 넣어야 한다는 것이다.

다음과 같은 메소드가 있다고 생각해보자.

```java
public void method(String ... args, String specific) {
    // do something
}
```

이 메소드를 호출할 때, specific 이라는 곳에 값을 넣어줄 수 있는 방법이 없다.  
따라서, IDE는 곧바로 에러를 띄워준다.

`Vararg parameter must be the last in the list`

# keyword-arguments
: keyword-argument, keyword-parameter, named-argument, named-parameter...  
모두 같은 이야기이다.

이는, 함수를 호출 할 때에, 값만 넘겨주는 것이 아니라,   
명시적으로 이름까지 지정하는 것이다.

가령, 파이썬에서의 예시는 다음과 같다.

```python
def introduce(name, age):
    print("안녕하세요, 제 이름은", name, "이고, 나이는", age, "입니다.")

introduce("홍길동", 25) # without keyword
introduce(name = "홍길동", age = 25) # with keyword
```

어느 파라미터에 어떤 변수를 집어넣을 것인지 명확하게 볼 수 있다.  
이러한 기능의 장점은, 파라미터의 변수가 많아질 때 극명하게 드러난다.  

아쉽게도, **자바는 이러한 기능을 지원하지 않는다.**  
그러나, 이를 유사하게 흉내낼 수 있는 방법을 몇 가지 소개한다.  

--- 

> Note. named parameter를 지원하지 않는 이유는, [여기](https://softwareengineering.stackexchange.com/questions/219593/why-do-many-languages-not-support-named-parameters)를 읽어보자.

---

위 파이썬 같은 코드를, 자바로 구현했다고 가정해보자.

```java
class Foo {
    public void introduce(String name, int age) {
        System.out.println("안녕하세요, 제 이름은 " + name + " 이고, 나이는 " + age + " 입니다." );
    }
}
```

기본적으로 사용할 때에는, 다음과 같이 사용한다.
```java
public class Main {
    public static void main(String[] args) {
        Foo foo = new Foo();
        foo.introduce("홍길동", 25);
    }
}
```

---


첫 번째로는, 변수를 lazy 하게 정의하는 방법이 있다.  

```java
public class Main {
    public static void main(String[] args) {
        Foo foo = new Foo();
        String name;
        int age;

        foo.introduce(name = "홍길동", age = 25);
    }
}
```

두 번째 방법은, 받은 값을 그대로 돌려주는 static method를 이용하는 방법이다.

```java
import static com.company.Foo.*;

public class Main {
    public static void main(String[] args) {
        Foo foo = new Foo();

        foo.introduce(name("홍길동"), age(25));
    }
}

class Foo {
    public void introduce(String name, int age) {
        System.out.println("안녕하세요, 제 이름은 " + name + " 이고, 나이는 " + age + " 입니다." );
    }

    public static String name(String name) {
        return name;
    }
    
    public static int age(String age) {
        return age;
    }
}
```

마지막 세 번째는 다른 방법들과 약간 궤를 달리하지만,  
마치 stream을 사용하듯이, Builder pattern을 사용하는 것이다.

```java
public class Main {
    public static void main(String[] args) {
        Foo foo = Foo.builder()
                .setName("홍길동")
                .setAge(25)
                .build();
        
        foo.introduce();
    }
}

class Foo {
    private final int age;
    private final String name;

    private Foo(Builder builder) {
        age = builder.age;
        name = builder.name;
    }

    public static Builder builder() {
        return new Builder();
    }

    public void introduce() {
        System.out.println("안녕하세요, 제 이름은 " + name + " 이고, 나이는 " + age + " 입니다." );
    }

    public static class Builder {
        private int age;
        private String name;

        public Foo build() {
            return new Foo(this);
        }

        public Builder setAge(int age) {
            this.age = age;
            return this;
        }


        public Builder setName(String name) {
            this.name = name;
            return this;
        }
    }


}
```

클래스의 instance를 직접 생성하지 않고,   
`Builder` 라는 static class 를 통해서 만든다.

이 `Builder`는 setter method 가 void가 아니고,   
자기 자신(instnace)를 return 해서,   
계속 set, set, set.. 을 할 수 있도록 한다.   

최종적으로 `build()` 메소드를 호출하면,  
`Foo` 클래스의 생성자로 지금까지 만든 `Builder`를 넘겨주어,  
`Foo` 의 instance를 만들도록 하는 방법이다.

약간 난해하게 생겼는데, 차분히 코드를 따라가면 이해할 수 있을 것이다.



## Reference
- [Java Varargs](https://www.programiz.com/java-programming/varargs)
- [Named Parameters in Java](https://dzone.com/articles/named-parameters-java-0)
- [Stackoverflow - Named Parameter idiom in Java](https://stackoverflow.com/questions/1988016/named-parameter-idiom-in-java)