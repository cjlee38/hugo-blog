---
categories: java
date: "2023-03-05T00:00:00Z"
tags: ['java', 'jmm']
title: 'Java Memory Model explained (feat. Effective Java, Item 78)'
draft: false
---

### 개요

이펙티브 자바의 ‘아이템 78 공유 중인 가변 데이터는 동기화해 사용하라’ 에서는 멀티태스킹 환경에서 발생할 수 있는 동시성 문제들을 소개하고, 이에 대한 해결책을 제시한다. 아이템 78을 읽어본 사람이라면, 아래의 유명한 코드를 한 번쯤은 본 적이 있을 것이다.

```java
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

위 코드를 읽어보면, `stopRequested` 가 1초 뒤에 `true` 로 바뀌었으므로 백그라운드 스레드는 종료되어야 할 것 같지만, 실제로 무한 루프를 돌게 된다. 책에서는 이를 OpenJDK의 경우 hoisting에 의한 것일 수 있다고 간단하게만 짚고 넘어간다. 또한, 이와 관련한 많은 자료들을 서치해보면 이는 `CPU cache`에 의한 문제라고 진단하고 넘어간다. 물론 틀린 말은 아니지만, 그 내부를 들여다보면 이는 간단하지 않은 문제다. 

### JIT compiler

위 문제의 원인을 진단하기 위해서는, 우선 `JVM` 이 java 파일을 실행시키는 과정을 자세히 살펴보아야 한다. 우리가 `javac` 명령어를 통해 `.java` 확장자를 가진 파일을 컴파일하게 되면, 이는 곧바로 CPU에서 실행될 수 있는 코드로 변환되지 않는다. 해당 코드는 `JVM` 이 이해할 수 있는 [Java Bytecode](https://en.wikipedia.org/wiki/Java_bytecode) 로 변환되고, 이를 `java` 명령어로 실행할 때가 되어서야 비로소 기계어로 번역되어 실행된다.

기계어로 번역되는 과정에서 `JVM`은 Java Bytecode를 CPU가 이해할 수 있는 `machine code`로 한줄씩 번역하게 된다. 이 과정에서 자주 참조하는 코드가 보이면(이 과정을 `warm-up` 이라고 표현하기도 한다.) 해당 코드를 통째로 컴파일해 실시간으로 번역하지 않고 컴파일을 진행한다.

### Reordering

위와 같은 일련의 과정을 거치면서, JVM은 주어진 Java Bytecode에 대한 최적화를 진행한다. 일례로, 아래의 코드를 살펴보자.

```java
public class Reordering {
    private int a = 0;
    private int b = 0;

    public void increase() {
        a += 1;
        b += 2;
        a += 3;
    }
}
```

위와 같은 코드를 작성하면 우리는 `increase` 메소드가 호출되었을 때 다음과 같은 순서로 동작하기를 기대한다.

1. 처음 `Reordering` 인스턴스를 생성하면, `a`와 `b`는 `0`으로 초기화된다.
2. `increase` 메소드를 호출하면, 아래와 같은 흐름으로 연산을 진행한다.
    1. `a`에 `1`을 더한다.
    2. `b`에 `2`를 더한다.
    3. `a`에 `3`을 더한다.

이러한 흐름은 사람에게는 자연스럽지만, 조금 더 기계친화적으로 흐름을 살펴보면 아래와 같이 동작한다.

1. `Reordering` 인스턴스에 대한 생성을 요청하면
    1. `Reordering` 인스턴스의 클래스를 분석하고, 해당 인스턴스에 대한 공간을 할당받는다.
    2. 공간을 할당받은 뒤, `init` 메소드(생성자)를 실행하여 변수에 대한 초기화를 진행한다.
2. `increase` 메소드를 호출하면, 아래와 같은 흐름으로 연산을 진행한다.
    1. 변수 `a`로부터 값을 읽어 CPU register에 올린다.
    2. CPU register에 `1`을 더하는 연산을 수행한다.
    3. `1`을 더한 연산을 마친 후, `a` 변수가 위치한 공간에 값을 다시 write 한다.
    4. 변수 `b` 로부터 값을 읽어 CPU register에 올린다.
    5. CPU register에 `2` 를 더하는 연산을 수행한다.
    6. `2`를 더한 연산을 마친 후, b 변수가 위치한 공간에 값을 다시 write 한다.
    7. 변수 `a`로부터 값을 읽어 CPU register에 올린다.
    8. CPU register에 `3`을 더하는 연산을 수행한다.
    9. `3`을 더한 연산을 마친 후, `a` 변수가 위치한 공간에 값을 다시 write 한다.

내용이 조금 길긴 하지만, 자세히 읽어보면 그다지 어려운 내용이 아님을 알 수 있다. 그저, 값을 읽은 뒤 더하기 연산을 한 후, 그 값을 다시 원래 위치에 쓰기 연산을 했을 뿐이다. 하지만 위 과정을 다시 읽어보면, 더 효율적으로 계산할 수 있는 방법이 있다는 것 또한 알 수 있다. 

즉, `2.1` 에 해당하는 읽기 연산을 한 후, `2.2`에 해당하는 덧셈 연산 직후에 곧바로 `2.7` 의 덧셈 연산을 연달아 수행하고, `2.8` 의 쓰기 연산을 이어나가면 명령어 수행 횟수를 줄일 수 있다. 이를 코드로 표현하면 아래와 같이 변경된다.

```java
public class Reordering {
    private int a = 0;
    private int b = 0;

    public void increase() {
        a += 1;
        a += 3;
        b += 2;
    }
}
```

심지어는 1과 3을 더하는 연산을 묶어서 차라리 4를 더해버리는 연산을 진행할 수도 있다. JVM은 위와 같이 코드의 순서를 변경하는 작업을 수행할 수 있으며, 이를 `reordering` 이라고 부른다.

이 `reordering` 은 JVM이 판단하기에 순서를 바꿔도 영향이 없는 코드에 대해 진행된다. 가령, `b += a;` 와 같이 `a`에 대한 덧셈 연산 결과를 바탕으로 `b`에 대한 연산을 수행하는 경우 해당 코드는 재배치되지 않는다.

### Word Tearing

위에서 소개한 예제 뿐만 아니라, 또 다른 문제도 발생할 수 있다. 아이템 78의 첫 페이지 세 번째 문단을 살펴보면, 아래와 같이 이야기한다.

> 언어 명세상 `long`과 `double` 외의 변수를 읽고 쓰는 동작은 원자적(atomic)이다.(JLS, 174. 17.7)
> 

`long`과 `double`은 왜 제외되는가 하고 보면, 두 개의 원시타입은 다른 타입들과 다르게 `64bit` 라는 특성을 가지고 있다. 다음의 코드를 살펴보자.

```java
public class WordTearing {
    long a = 0L;
    
    void thread1() {
        a = 0x0000FFFF;
    }
    
    void thread2() {
        a = 0xFFFF0000;
    }
}
```

위 코드를 두 개의 스레드가 동시에 접근하여 수행하면 어떤 결과가 나타날까 ? 일반적으로 생각했을 때에는 `0x0000FFFF` 혹은 `0xFFFF0000` 둘 중 하나로 결과가 남을 것으로 기대해볼 수 있을 것이다. 하지만 32bit processor는 CPU register에 최대 32bit 만 올릴 수 있으므로, 총 64 bit 중 앞 쪽 32bit 의 쓰기연산과 뒤 쪽 32 bit의 쓰기 연산의 총 두 개의 연산으로 작업을 수행한다. 따라서 `0x0000FFFF`, `0xFFFF0000`, `0x00000000`, `0xFFFFFFFF` 의 총 4개의 결과가 가능한 결과 집합이 되고, 이러한 문제를 [Word Tearing](https://docs.oracle.com/javase/specs/jls/se17/html/jls-17.html#jls-17.6) 이라 부른다.

### JMM

하지만 그렇다고 해서 위 코드들을 실제로 로컬환경에서 수행해봐도 문제가 발견되지 않을 수 있다. 이는 하드웨어 아키텍쳐의 차이로 인해 발생하는 것인데, 가령 64bit processor를 이용하는 경우에는 위와 같은 원자성 문제가 발생하지 않는다.

![](/assets/images/2023-03-06-java-memory-model-explained/2023-03-06-01-25-00.png)

위 표는 각 CPU 아키텍쳐 별 optimization 지원 여부를 나타낸다.  가령, intel CPU의 경우 `load-load` 를 지원하지 않는다. `load-load` 는 하나의 변수를 읽고 다음 변수를 읽을 때 `reordering`이 발생하는지 여부를 의미한다. 한편, ARM CPU는 `load-load`를 지원하며, 이는 곧 두 개의 변수를 읽을 때 순서의 재배치가 발생할 수 있음을 의미한다.

> ARM 아키텍쳐의 CPU가 상대적으로 전력을 적게 소모하는 이유 중 하나이다.
> 

`Java Memory Model`(이하 `JMM`) 은 특정 순간에 어떠한 필드를 읽었을 때 우리가 어떠한 값을 관찰할 수 있는지를 설명하는 모델이다. `JMM`은 [약한 메모리 모델](https://preshing.com/20120930/weak-vs-strong-memory-models/)을 기반으로 설계되었기 때문에 각각의 CPU core가 바라볼 수 있는 값이 다를 수 있고, 따라서 이를 적절하게 제어하는 규칙에 대한 specification을 제공한다.

그리고 `JMM`에서 애플리케이션을 보호하는 개념 중 하나가 바로 오늘 중점적으로 다루게 될 [happens-before](https://docs.oracle.com/javase/specs/jls/se17/html/jls-17.html#jls-17.4.5) 관계이다. 여기서 한 가지 반드시 짚고 넘어가야 할 점은 `happens-before` 관계는 single thread를 기반으로 보장되었다는 점이다. 이를 다시 말하면 multi-thread 환경에서는 사용자가 어느정도는 의식적으로 염두에 두어야 한다는 말이다.

필드 영역에는 `final` , `volatile`, 메소드 영역에는 `synchronized`, `java.util.concurrent.locks.Lock` 를 이용해 이를 제어할 수 있으며, 해당 키워드/API 를 사용함으로서 JVM에게 동시성 이슈를 발생시킬 수 있는 최적화를 억제하도록 유도할 수 있다. 하지만 반대로 이야기하면 최적화를 진행하지 않음으로 인해 성능저하를 유발할 수도 있다는 뜻이 된다.

하나씩 살펴보자.

### volatile

앞서 맨 처음 보았던 무한루프를 돌았던 코드와 비슷한 코드를 다시 한번 살펴보자.

```java
class DataRace {
    boolean ready = false;
    int answer = 0;
    
    void thread1() {
        while (!ready);
        assert answer == 42;
    }
    
    void thread2() {
        answer = 42;
        ready = true;
    }
}
```

두 개의 스레드가 동시에 thread1 메소드와 thread2 메소드를 수행하면 어떻게 될까? `assert` 부분이 통과할 수 있을까? 앞서 문제점들을 인지했다면, 위 코드는 실패할 가능성이 있다는 것을 알 것이다. 왜냐하면, thread2 의 `answer = 42;` 코드와 `ready = true;` 코드의 순서가 바뀌어서 `ready`가 먼저 `true`가 되버릴 수 있기 때문이다. 혹은, 앞서 보았던 예시와 마찬가지로 `ready` 변수가 CPU cache에 저장되어 `ready`가 `true`로 변경되었음에도 불구하고 여전히 `false`로 읽을 수도 있다.

이 문제를 해결하기 위해서는 아래와 같이 `ready` 변수에 [volatile](https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.3.1.4) 키워드를 사용할 수 있다. 

`volatile boolean ready = false;`

JLS 에 나와있듯이, volatile 키워드가 붙은 변수는 모든 스레드가 ‘일관된 값’을 볼 수 있음을 보장해준다.(A *field may be declared `volatile`, in which case the Java Memory Model ensures that all threads see a consistent value for the variable*). 뿐만 아니라, volatile에 대한 쓰기 연산은 읽기연산에 선행(happens-before) 된다.

한 가지 더 주목할 점은, `volatile` 키워드를 `ready` 변수에만 붙여주었다는 점이다.이 또한 JMM이  `volatile` 변수에 대한 쓰기연산 이전에 선행된 쓰기 연산은, `reordering` 되지 않음을 보장하기 때문이다. (읽기 연산 또한 마찬가지로, volatile 변수에 대한 읽기가 선행되고 이후에 다른 변수에 대한 읽기가 진행됨이 보장된다.)

마지막으로, 앞서 보았던 Word Tearing 또한 [발생하지 않는다.](https://docs.oracle.com/javase/specs/jls/se17/html/jls-17.html#jls-17.7)

> 많은 사람들이 volatile 키워드는 메모리에 곧바로 쓸 수 있도록 하는 역할이라고 설명한다.
이것이 틀린 설명은 아니지만, JLS 상에서는 volatile 키워드가 ‘consistence’(일관성)을 유지하도록 보장한다고 설명한다. 즉, ‘명세’ 상으로는 항상 일관된 값을 관찰할 수 있어야 한다는 것이고, 이에 대한 구현(implementation)이 바로 메인 메모리에 대한 flush 라고 볼 수 있다.
> 
> **이는 `volatile`뿐만이 아닌 모든 `happens-before` 관계를 보장하는 (e.g. `synchronized`) 키워드에도 해당된다.**


### synchronized

`synchronized` 키워드 또한 `happens-before` 관계를 성립시키는 도구로 사용될 수 있다. 다음은 역시 마찬가지로 이펙티브 자바 아이템 78 에서 인용한 내용이다.

> 많은 프로그래머가 동기화를 배타적 실행, 즉 한 스레드가 변경하는 중이라서 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막는 용도로만 생각한다. 
> 
> … 중략
> 
> 맞는 설명이지만, 동기화에는 중요한 기능이 하나 더 있다. **동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다.** 동기화는 일관성이 깨진 상태를 볼 수 없게 하는 것은 물론, 동기화된 메소드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다.
> 

즉, `synchronized` 키워드에는 상호배제(mutual exclusion) 이외에도 **스레드 간 통신**(앞서 보았던 `happens-before`) 의 기능을 제공한다는 뜻이다.

위 `DataRace` 코드를 `synchronized` 키워드를 이용해 수정하면 다음과 같다.

```java
class DataRace {
    boolean ready = false;
    int answer = 0;

    synchronized void thread1() { // <- 데드락 발생 가능
        while (!ready);
        assert answer == 42;
    }

    synchronized void thread2() {
        answer = 42;
        ready = true;
    }
}
```

이 코드를 곧바로 실행하면, 아마 thread1 메소드가 먼저 수행되는 경우에는 데드락이 발생하게 된다. 여기서는 “thread2 코드가 먼저 실행되었다” 라고 가정해보자. 그러면 아래와 같은 흐름으로 진행된다.

| 스레드 | Thread 1 | Thread 2 |
|:---:|:---:|:---:|
| 실행 |  | enter |
|  |  | answer = 42; |
|  |  | ready = true; |
|  |  | exit |
|  | enter |  |
|  | while (!ready); |  |
|  | assert answer = 42; |  |
|  | exit |  |

이펙티브 자바에서 설명했던 것과 마찬가지로, `monitor`를 통한 `critical section`의 진입/탈출 뿐만 아니라 `ready` 변수와 `answer` 변수에 대한 순서는 재배치되지 않고 순서대로 (`happens-before`)수행된다.

`synchronized` 키워드와 관련해서 한 가지 더 짚고 넘어가고 싶은 부분이 있다. 가장 먼저 봤던 `StopThread` 클래스를 다시 살펴보자.

```java
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested) {
                i++;
                System.out.println("STOP ?"); // (*)
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

위 코드에서, 주석으로 별표(*)를 달아놓은 부분을 새로이 추가하였다. 그리고 이 코드를 실행되면 어떻게 될까? 놀랍게도 이번에는 무한 루프를 돌지 않고 약 1초 뒤 종료된다. 출력문 하나를 추가하였을 뿐인데 왜 갑자기 1초뒤에 중단되었을까? 이는 `sout` 메소드를 살펴보면 알 수 있다.

```java
public void println(String x) {
    synchronized (this) {
        print(x);
        newLine();
    }
}
```

보는 바와 같이 `synchronized` 키워드가 존재한다. 그런데 여기서 한 가지 궁금증이 생긴다. `synchronized` 는 `this` 에 걸려있고, 이 `this`는 결국 `System.out` 인데, 왜 난데없이 전혀 관련 없어보이는 `stopRequested` 변수가 초기화되었을까 ? [이는 synchronized block에 진입하는 순간, 해당 스레드가 볼 수 있는 모든 변수가 메모리로 flush 되었기 때문이다.](https://jenkov.com/tutorials/java-concurrency/java-happens-before-guarantee.html)

아래 코드는 이펙티브 자바에서 보인 `StopThread` 의 `synchronized` 키워드를 이용한 개선 버전인데, 이 부분도 자세히 보면 처음에는 “왜 `synchronized` 키워드를 붙인게 효과가 있지?” 라는 생각이 들었다가, 스레드 간 통신을 지원한다는 점을 떠올리면, 곧 고개를 끄덕일 수 있을 것이다.

```java
public class StopThreadSynchronized {
    private static boolean stopRequested;

	private static synchronized void requestStop() {
		stopRequested = true;
	}

	private static synchronized boolean stopRequested() {
		return stopRequested;
	}

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested())
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

### final

자바에서 `final`은 변수에 대한 재할당을 막는 역할로 널리 알려져 있다. 하지만 final 키워드 또한 `happens-before` 관계와 연관이 있다. 다음 코드를 보자.

```java
public class UnsafePublication {
    private int a;

    private UnsafePublication() {
        a = 42;
    }
    
    static UnsafePublication instance;
    
    static void thread1() {
        instance = new UnsafePublication();
    }
    
    static void thread2() {
        if (instance != null) {
            assert instance.a == 42;
        }
    }
}
```

흔히 싱글톤 패턴을 구현할 때 자주 볼 수 있는 코드 형태이다. (물론 동시성 문제를 고려하여 개선버전이 존재한다.) 마찬가지로 `thread1` 과 `thread2` 메소드가 동시에 수행되었을 때, 해당 코드는 항상 통과할 수 있을까?

정답은 ‘그렇지 않다’ 이다. 앞서 `Reordering` 절에서도 언급했듯이, `new` 키워드를 이용해 객체를 생성하는 행위는 두 가지로 구분된다.

1. 객체를 생성하기 위한 공간의 할당
2. 객체 변수의 초기화

따라서 1. 객체를 생성하기 위한 공간의 할당이 먼저 선행되고, `thread2` 메소드가 수행되는 경우, instance는 `null`을 가리키지 않으므로 if 절 안으로 진입하게 된다. 하지만, 객체 변수의 초기화 (`a = 42;`) 는 아직 수행되지 않았으므로(int 변수는 초기화되지 않으면, `0`을 갖는다.) 해당 assert 절은 실패하게 된다.

> 참고로 파이썬에서는 생성자로 사용되는 `__init__` 메소드 이외에도 `__new__` 라는 메소드가 있는데, 이는 메모리를 할당받을 때 사용된다. 즉, `__init__` 메소드보다 반드시 선행된다.
> 

이를 해결하기 위한 방법으로 `final` 을 사용할 수 있다. 위 코드에서 `private final int a;` 과 같이 `final`을 붙여주게 되면, 생성자 메소드 `init` 이 끝난 직후에 `freeze` 라는 작업을 하게 된다. 생성자 내에서 또한 `happens-before` 관계가 보장되며, 생성자 체이닝(e.g. 부생성자 → 주생성자)이 발생한다면 최종 생성자가 끝나는 시점에 `freeze` 작업이 수행된다.

### 마무리

JVM의 구현체는 명세보다 더 엄격하게 관리한다. 따라서 구현체를 믿고 프로그램을 작성하다보면, Java 진영에서 이야기하는 `Cross-Platform Compatibility`, 즉  `Write Once, Run Anywhere` 이 어려워질 가능성이 높다. 따라서 보수적으로 코드를 작성해야 함에 유의하자.

---

### Reference
- [https://youtu.be/qADk_tj4wY8](https://youtu.be/qADk_tj4wY8)
- [https://jenkov.com/tutorials/java-concurrency/java-happens-before-guarantee.html](https://jenkov.com/tutorials/java-concurrency/java-happens-before-guarantee.html)
- [https://www.geeksforgeeks.org/happens-before-relationship-in-java/](https://www.geeksforgeeks.org/happens-before-relationship-in-java/)
- [https://docs.oracle.com/javase/specs/jls/se17/html/jls-17.html](https://docs.oracle.com/javase/specs/jls/se17/html/jls-17.html)
- [https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=jjoommnn&logNo=130037479493](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=jjoommnn&logNo=130037479493)
