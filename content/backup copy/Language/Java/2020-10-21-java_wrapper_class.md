---
# author: cjlee
categories: java
# cover: /assets/covers/coding.png
date: "2020-10-21T20:26:00Z"
tags: ['null']
title: '# (Java) Wrapper Class는 Call by Value 일까?'
---

# 0. 들어가며
: Java 는 Call by Value일까, Call by Reference 일까?   

지금까지는, 함수의 parameter로 primitive type을 넘겨줄 때에는 call by value,  
객체를 넘겨줄 때에는 call by reference로 알고 있었다.

그런데, 구선생님의 말씀에 따르면, 그렇지는 않은 것 같다.

# 1. Call in Java
: Java 에서는 언제나 Call by Value 라고 한다.   
[약 12년 된 스택오버플로우의 한 질문글](https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value?page=1&tab=oldest#tab-top)에 달린 답변은 다음과 같다.

---

> 자바는 예외 없이, 항상 pass by value 입니다.   
> 그렇다면, 왜 사람들이 이것을 혼란스러워 하고, Java는 pass by reference라고 믿고 있을까요? 키 포인트는, Java가 어떠한 상황에서도, 객체의 값에 직접적인 접근을 제공하지 않기 때문입니다. 객체에 접근하는 유일한 방법은 그 객체의 참조를 통하는 것뿐입니다. Java 객체가 직접 접근되지 않고 참조에 의해 접근되기 때문에, 현학적으로 필드, 변수, 그리고 메소드 인자를 객체인 것처럼 이야기 되는데, 이것들은 객체에 대한 참조일 뿐입니다. 이러한 혼란은 (엄밀히 말하면, 틀린) 명명(命名)의 변화에서 기인합니다.  
> 따라서, 메소드를 호출할 때,
> - Primitive인 경우 : pass-by-value 는 primitive의 실제 값의 전달입니다.
> - 객체인 경우 : pass-by-value는 객체에 대한 참조 "값"의 전달입니다.  
> ... 하략

---

또 다른 답변의 경우, 친절하게 예시도 같이 달아주었다.

```java
public static void main(String[] args) {
    Dog aDog = new Dog("Max");
    Dog oldDog = aDog;

    // we pass the object to foo
    foo(aDog);
    // aDog variable is still pointing to the "Max" dog when foo(...) returns
    aDog.getName().equals("Max"); // true
    aDog.getName().equals("Fifi"); // false
    aDog == oldDog; // true
}

public static void foo(Dog d) {
    d.getName().equals("Max"); // true
    // change d inside of foo() to point to a new Dog instance "Fifi"
    d = new Dog("Fifi");
    d.getName().equals("Fifi"); // true
}
```

이 예제를 보고 나니, 위 답변이 무슨 말을 하는지 이해할 수 있었다.

코드를 보면, `aDog` 라는 변수를 `foo()` 메소드로 넘겨주었다.  
`foo()` 메소드 안에서, `getName()` 을 찍어보면 `"Max"`로 찍히는 것은 당연하다.  
**그러나, 주의깊게 봐야 할 부분은 `d = new Dog("Fifi")` 다.**

만약 참조에 의한 호출이었다면, `d` 라는 참조는 본래 `aDog`가 가리키는 객체를 가리킬 것이다.  
따라서, `aDog는` Fifi로 바뀌어야 한다.  

그러나 메소드를 실행하고 나서 다시 `aDog`의 `getName()`을 찍어보면,  
여전히 `"Max"`로 남아있다.

왜냐하면, `aDog` 라는 **참조** 를 넘겨준 것이 아니라, **복사본** 을 넘겨주었기 때문이다.

여전히 이해하기 힘들다면, 다음의 그림을 보자.

![](/assets/images/2020-10-27-20-59-29_2020-10-21-java_wrapper_class.md.png)

```java
Dog aDog = new Dog("Max");
Dog oldDog = aDog;
```
위 코드를 실행하면, 그림과 같이 `aDog과` `oldDog는` Max라는 녀석을 가리키고 있다.  
여기서, `foo()` 메소드를 실행하면 어떻게 될까?

![](/assets/images/2020-10-27-21-06-02_2020-10-21-java_wrapper_class.md.png)

위 그림과 같이, `d`는 `aDog`의 복사본(call-by-value)이 된다.  
만약, 참조에 의한 호출이었다면 다음과 같은 모양새였을 것이다.

![](/assets/images/2020-10-27-21-11-08_2020-10-21-java_wrapper_class.md.png)

그러나, 실제로 `d`는 복사본이므로, `new Dog("Fifi")`를 수행하면 다음과 같이 된다.

![](/assets/images/2020-10-27-21-08-22_2020-10-21-java_wrapper_class.md.png)

따라서, `foo()` 메소드 안에서 `getName()` 을 찍어보면 `"Fifi"`로 나오지만,  
메소드가 끝난 이후, 메인 메소드에서 다시 `getName()`을 찍어보면 `"Max"`로 나오게 된다.

반면, `foo()` 메소드 안에서, `d.setName("Fifi")` 라고 사용한다면,  
다음 그림과 같이 메소드 종료 이후에 `aDog`의 Name은 `"Fifi"`가 될 것이다.

![](/assets/images/2020-10-27-21-15-17_2020-10-21-java_wrapper_class.md.png)


결국 Java는 Call-by-value 이지만, 
우리가 사용할 때에는 대부분 Call-by-Reference 처럼 인식하게 된다.  
Call-by-value라고 해서, 특별하게 걱정할 내용은 없다는 얘기다.

# 2. How about Wrapper classes?
: 하지만, Integer 와 같은 Wrapper class는 어떨까?  
다음 코드의 결과를 예상해보자.

```java
public class Main {

    public static void main(String[] args) {
        int intValue = 10;
        Integer integerValue = new Integer(10);
        MyInteger myIntegerValue = new MyInteger(10);

        System.out.println("before intValue = " + intValue);
        System.out.println("before integerValue = " + integerValue);
        System.out.println("before myIntegerValue = " + myIntegerValue.getValue());

        intMethod(10);
        integerMethod(integerValue);
        myIntegerMethod(myIntegerValue);

        System.out.println("after intValue = " + intValue);
        System.out.println("after integerValue = " + integerValue);
        System.out.println("after myIntegerValue = " + myIntegerValue.getValue());

    }

    public static void intMethod(int value) {
        value = value + 1;
    }

    public static void integerMethod(Integer value) {
        value = value + 1;
    }

    public static void myIntegerMethod(MyInteger value) {
        value.increase();
    }

}

class MyInteger {
    private int value;

    public MyInteger(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }

    public void increase() {
        value = value + 1;
    }
}
```

결과는 다음과 같다.

```
before intValue = 10
before integerValue = 10
before myIntegerValue = 10
after intValue = 10
after integerValue = 10
after myIntegerValue = 11
```

int 값의 경우, 위에서 이야기한 것처럼 **call-by-value** 이므로,  
10이 그대로 유지되는 것은 당연하다.

MyInteger의 경우 또한, `myIntegerValue` 라는 참조값을 복사해서 전달했으므로,  
11로 늘어나는 것 또한 타당하다.   
이는 앞서 이야기한 코드 예시에서, `setName("Fifi")` 를 실행한 것과 비슷하다.

**주목할 부분은, `integerValue` 다.**  
Integer는 primitive가 아닌 class이고,   
따라서 `integerValue` 는 `new Integer(10)` 을 가리키고 있던 것이 아닐까?  
그렇다면, `value = value + 1` 을 실행하면 11이 나와야 할 것 같은데,  
10으로 그냥 유지되어 버렸다.

왜 이렇게 되었을까?

# 3. Secret in Wrapper class
: 그 비밀을 파헤치기 위해, Integer 클래스를 살짝 뜯어보자.  
Integer 클래스의 (int 형을 넘겨주는 경우의) 생성자는 다음과 같이 생겼다.

```java
public Integer(int value) {
    this.value = value;
}
```

이 this.value 를 찾아가보면, 다음과 같이 final 로 선언된 것을 볼 수 있다.

```java
private final int value;
```

말인 즉슨, Integer 클래스는 **Immutable(불변)** 이라는 뜻이 된다.  
그렇다면, 아까 작성했던 `value = value + 1` 은 어떻게 동작했을까?  

Wrapper class에 대해서 배웠다면, `autoboxing`, `unboxing` 이라는 용어에 대해서 들어봤을 것이다.  
이 `boxing`이 어떻게 일어나는지, 바이트 코드를 들여다보자.

```java
public static void main(String[] args) {
    Integer integerValue = 10;
}
```

```properties
  public static void main(java.lang.String[]);
    Code:
       0: bipush        10
       2: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       5: astore_1
       6: return
```

`2: invokestatic  #2     // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;`   
이라는 부분을 보면, valueOf() 라는 static 메소드를 호출하고 있음을 볼 수 있다.

`valueOf()` 메소드는 다음과 같이 생겼다.

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

따라서, `autoboxing`은 내부적으로 컴파일러가 `valueOf()` 메소드를 호출했고,   
이에 따라 새로운 Integer 객체를 생성해서, 이를 return 하고 있음을 볼 수 있다.

그렇다면, 덧셈은 어떻게 동작할까?

```java
public static void main(String[] args) {
    Integer integerValue = 10;
    integerValue = integerValue + 1;
}
```

```
 public static void main(java.lang.String[]);
    Code:
       0: bipush        10
       2: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       5: astore_1
       6: aload_1
       7: invokevirtual #3                  // Method java/lang/Integer.intValue:()I
      10: iconst_1
      11: iadd
      12: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
      15: astore_1
      16: return
```

0, 2, 5 번호의 byte code는 위의 내용과 같다.  
`valueOf()` 메소드를 실행한 뒤에,   
그 결과인 Integer 객체를 지역변수 Array의 1번에 할당했다.

6 번에서, 만들었던 이 Integer 객체를 꺼내왔고,  
7 번에서, `intValue()`라는 메소드를 호출했다.    
참고로 `intValue()`는 다음과 같이 생겼다.

```java
public int intValue() {
    return value;
}
```

다음으로 10번에서, 1 이라는 상수 값을 가져왔다.  
(`iconst_1`은 `integerValue = interValue + 1` 에서, `1`을 담당한다.  
만약, `1` 대신 `3`을 더했다면, 바이트 코드는 `iconst_3`이 된다.)

이렇게 까지 하면,  
아까 Integer 객체에서 꺼낸 int 값과,   
상수 1이 Stack에 쌓여 있다.

11번에서 이 두개를 더한다(`iadd`)

그리고, 다시 `Integer.valueOf()` 의 static method를 실행해서,  
다시 Integer 객체로 만들고, 이를 다시 지역변수 Array의 1번에 저장한다.

와우, 긴 여정이 끝났다. 정리하면 다음과 같다. 

(Integer 덧셈코드)
```java
Integer integerValue = 10;
integerValue = integerValue + 1;
```

1. int 값(10)을 boxing(Integer 객체로 만듦)하고, 저장한다.
2. 저장한 Integer 값을 unboxing 한다.
3. unboxing 된 int 값에, 1을 더한다.
4. 더한 결과(11)를 다시 boxing 하고, 저장한다.

--- 

> Note. 이런 동작방식을 증명하는 또 다른 방법은,   
> 다음과 같은 코드를 실행해보는 것이다.
> ```java
> Integer value = 10;
> System.out.println("value.getClass() = " + value.getClass());
> System.out.println("(value+1).getClass() = " + (value+1).getClass());
> ```
> 당연하게도, (value+1)은 unboxing된 int형이므로,   
> getClass()라는 메소드가 없다는 에러를 뱉는다.

---

이 과정을, 아까 수행한 `integerMethod()`에 대입해서 생각해보자.  
다시 스크롤해서 올려보기 귀찮을테니, 코드를 여기에 다시 적겠다.

```java
public static void integerMethod(Integer value) {
    value = value + 1;
}
```

아직 덧셈 연산을 하기 전에는, 다음과 같은 형태를 갖고 있을 것이다.

![](/assets/images/2020-10-27-22-14-26_2020-10-21-java_wrapper_class.md.png)

그러나, 방금 정리했듯이,   
**더한 결과를 다시 Boxing하므로**  
다시 말해, **새로운 new Integer() 객체를 생성하므로,**  
덧셈을 하고나면, 다음의 그림과 같이 된다.

![](/assets/images/2020-10-27-22-16-00_2020-10-21-java_wrapper_class.md.png)

value 라는 변수는 위와 같이 11 이라는 **새로운 객체**를 가리키고 있으므로,  
`integerMethod()` 를 수행하더라도, 메소드의 종료 이후 결과값은 변하지 않는다.

# 4. 마치며
: 길고도 험난한 여정이었다.  
C언어는 포인터라는 개념이 있기에,   
메소드를 호출해서 그 내부의 값을 변경하려면 무조건 이를 이용해야 하는데,  
Java에서는 primitives 와 object 의 동작방식이 다르고,   
이에 더해 Wrapper class는 클래스임에도 불구하고 primitives 처럼 동작해서 자주 헷갈렸다.

이번 포스팅을 통해 확실하게 개념을 잡을 수 있었던 것 같다.  
이상으로 포스팅을 마칩니다.

## Reference
- [Is Java “pass-by-reference” or “pass-by-value”?](https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value?page=1&tab=oldest#tab-top)
- [Wrapper classes and call by reference in java [duplicate]](https://stackoverflow.com/questions/20804862/wrapper-classes-and-call-by-reference-in-java)
- [Java bytecode instruction listings](https://en.wikipedia.org/wiki/Java_bytecode_instruction_listings)