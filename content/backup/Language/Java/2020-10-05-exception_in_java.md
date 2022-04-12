---
# author: cjlee
categories: java
# cover: /assets/covers/coding.png
date: "2020-10-05T06:42:00Z"
tags: ['null']
title: '# (Java) 예외처리를 위한 Try-Catch'
---

# 0. 들어가며
: 예외처리는, 그 이름만 들어도 두려움이 생긴다.  
간단하게 생각하자면 if else문의 연장선인 것 같으면서도,  
또 뭔가 시스템의 장애를 덕지덕지 막아내는 것 같은 찝찝함도 동시에 든다.

예외처리문을 쓸때면 항상  
"애초에 코드를 잘 짜면 되지 않나?" 라는   
바보같은 의문이 남기도 한다.

아무튼, 오늘 다루고자 하는 내용은  
"예외 처리는 이렇게 해야한다!" 하는 거창한 내용은 못되고(쓸 실력도 안되고),  
기본적으로 알고 있었던, 그리고 글을 쓰면서 새로 배운,  
이러이러한 것들이 존재한다~ 정도의 Introduction 느낌만 가져가려고 한다.  

# 1. 예외란 무엇인가?
: Error와 Exception은 다르다.   

### - Error
: 시스템 레벨에서 발생하는 **심각한** 수준의 오류. 프로그램 내에서 처리할 수 없음.  
e.g. OutOfMemoryError, StackOverflowError

### - Exception
: 개발자의 설계 미숙 등으로 인해, 로직 상의 문제. 미리 예측하고, 처리할 수 있다.  
e.g. NullPointerException, IndexOutOfBoundException

# 1. try - catch - finally
: 아마 try-catch 문을 배웠다면,   
가장 먼저 배우는 기본적인 형태는 다음과 같을 것이다.

```java
public class Main {

    public static void main(String[] args) {
        myException();
    }

    public static void myException() {
        try {
            System.out.println("try here");
//            throw new IllegalStateException("exception here");
        } catch(Exception e) {
            System.out.println("catch here");
            System.out.println("e : " + e.getMessage());
        } finally {
            System.out.println("finally here");
        }
    }

}
```
중간에 주석처리된 throw new ... 는 예외를 발생시키는 코드다.  
저 키워드에 대해서는 후술하겠다.

어쨌든, myException() 이라는 함수가 실행되면,  
try 구문에서 `try here`를 출력하고,  
에러가 발생한다면(**주석이 해제된다면**) `catch here` 와 `e : exception here` 를,  
그리고 마지막으로 finally의 `finally here`를 출력하고 함수가 종료된다.

만약 이러한 exception에 대한 처리를 해주지 않았더라면,  
NullPointerException, ArrayIndexOutOfBoundsException 등이 발생했을 때  
프로그램이 개복치처럼 죽어버렸을 것이다.

---

> Note. 적절한 예외처리에 대한 내용은 잘 모르기도 하고, 이 포스팅의 취지에 맞지 않아  
> 최대한 빼려고 했는데, 그래도 이 부분은 중요한 것 같아서 꼭 넣어야 할 것 같다.   
> 무엇이냐 하면, **모든 예외를 Handling 해버리는, 상위 클래스인 Exception으로 예외처리하지 않는 것을 권장한다.**  
> 다시 말해, **특정 예외만을 받아내는 Exception으로 처리해야 한다.**

---


또한, finally가 **언제까지 살아있느냐?**를 주의깊게 봐야한다.  
다음의 코드를 보자.

```java
public class Main {

    public static void main(String[] args) {
        myException();
    }

    public static int myException() {
        try {
            System.out.println("try here");
            throw new IllegalStateException("exception here");
        } catch(Exception e) {
            System.out.println("catch here");
            System.out.println("e : " + e.getMessage());
            return -1;
        } finally {
            System.out.println("finally here");
        }
    }

}
```

위에서 작성한 코드와의 차이는 딱 두 가지다.  
1. try문에서, throw keyword의 주석을 해제했다.
2. myException() 함수의 return을 int로 바꾼 뒤, -1 *(임의의 값)* 을 return하도록 해서, 이를 출력하도록 했다.

try-catch가 익숙치 않은 사람이라면, finally가 있다 하더라도 return이 있기 때문에  
`finally here`는 출력되지 않을 것으로 생각할 수 있지만,  
실제로는 `finally here`까지 출력이 되고, return 이 이루어진다.

따라서, 실제 출력은 다음과 같이 이루어진다.

```
try here
catch here
e : exception here
finally here
-1
```

함수를 바로 종료시켜버리는 막강한 return조차 기다리게 만드는 finally라니, 도대체 어떻게 가능한 것일까?  
이를 알아보기 위해, 위 코드를 컴파일 시킨뒤, IDE를 이용하여 다시 디컴파일 시켜보자.

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//
public class Main {
    public Main() {
    }

    public static void main(String[] var0) {
        int var1 = myException();
        System.out.println(var1);
    }

    public static int myException() {
        byte var1;
        try {
            System.out.println("try here");
            throw new IllegalStateException("exception here");
        } catch (Exception var5) {
            System.out.println("catch here");
            System.out.println("e : " + var5.getMessage());
            var1 = -1;
        } finally {
            System.out.println("finally here");
        }

        return var1;
    }
}

``` 

디컴파일 시킨 코드를 보니, 우리가 작성했던 return 문이 catch 안에서 실행되는 것이 아니고,  
var1 이라는 byte 변수에 담아 놓은 뒤에, finally 이후에 return 되는 것을 볼 수 있다.

즉, 우리가 catch안에 return을 작성하더라도, **컴파일러에 의해 뒤로 미뤄진다.**

# 2. Exception class
: 예외 클래스는 다음과 같은 구조를 가진다.

![](/assets/images/2020-10-20-19-38-58_2020-10-05-exception_in_java.md.png)

(인터페이스의 네이밍을 가진) Throwable이라는 클래스 하위에,  
아까 보았던 Exception과 Error가 구분되고,  
그 하위에 Exception을 직접 상속하는 클래스들,  
그리고 Exception을 상속하는 RunTimeException을 상속하는 클래스들로 이루어진다.

이 때, Checked와 Unchecked는 다음과 같은 차이가 있다.

### - Checked Exception
: 컴파일시에 "에러를 처리하는지"의 여부를 확인한다.  
예외 발생 시 transaction을 roll-back 하지 않는다.

### - Unchecked Exception
: 예외가 런타임시에 발생할수도, 발생하지 않을수도 있으므로,  
컴파일할 때에는 에러 처리 여부를 확인하지 않는다.  
예외 발생 시 transaction을 roll-back 한다.


# 3. throw와 throws
: 이 두가지 키워드가 수행하는 역할은 다르지만, 비슷하게 생겨서 묶었다.


### - throw

: 위에서 적어놓았던 `throw new IllegalStateException();` 은,  
**IllegalStateException 이라는 예외를 "발생"시킨다.**  
즉, throw는 예외를 발생시키는 역할을 한다.

### - throws
: 반면, throws는 예외를 **던진다.**
무슨말인가 하면, 예외가 발생했을 때 이를 메소드 자기 자신이 처리하지 않고,  
자기 자신을 **호출한** 메소드에게 "이거 처리해주세요" 하고 던진다.

계속해서 stack을 타고 가다보면, 최종적으로는 main 메소드를 실행한 JVM 에게 던진다.

다음의 코드의 흐름을 잘 살펴보면, 위 두 키워드의 차이에 대해서 알 수 있으리라 생각한다.

```java
public class Main {
    public static void main(String[] args) {
        try {
            myException();
        } catch(IllegalStateException e) {
            System.out.println(e.getMessage());
        }

    }

    public static int myException() throws IllegalStateException {
        throw new IllegalStateException("exception");
    }
}
```


# 4. try-with-resources
: 버퍼 입출력을 지원하는 클래스인 BufferedWriter는 Checked Exception중 하나이다.

```java
Path file =  Paths.get(" ... ");
BufferedWriter writer = null;

writer = Files.newBufferedWriter(file); // IOException
writer.write(" ... "); // IOException
writer.close(); // IOException
```

위 세 곳에서 IOException이 발생하므로,   
IDE에서 Unhandled Exception이 있다면서 빨간 줄을 쭉 그어준다.

그러나, close()의 경우 **반드시 실행되어야 하기 때문에,**  
try가 아닌 finally block에 들어가야 하는데, 이 또한 IOException이 발생할 수 있다.  
따라서, finally에 try-catch를 한번 더 써야하는데, 이는 꽤 귀찮은 일이다.

이를 위해, try-with-resources 라는 것이 생겼다.  
형태는 다음과 같다.

```java
try(BufferedWriter writer = Files.newBufferedWriter(file)) {
    writer.write(" ... ");
} catach(IOException e) {
    // do something
}
```
마치, 파이썬에서 Context Manager라고 불리는 with 과 비슷하다.  
파이썬의 Context Manager는 close() 함수를 호출하지 않아도,  
with 구문이 끝나면 알아서 호출해준다.
```python
with open("foo.txt", "r") as f:
    file = f.read()
```

아무튼, 이러한 try-with-resources 의 resources가 되기 위해서는  
`java.lang.AutoCloseable` 인터페이스를 구현해야 한다.

