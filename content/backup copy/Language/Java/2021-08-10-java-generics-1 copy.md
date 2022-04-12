---
# author: cjlee
categories: java
# cover: /assets/covers/coding.png
date: "2021-08-10T19:26:00Z"
tags: ['null']
title: '# 제네릭 톺아보기 1'
---

# 0. 들어가며

제네릭은 `Java Collection Framework` 를 사용하면 필연적으로 마주치는 문법 중 하나입니다. 처음 보았을 때에는 `<>` 형태의 다이아몬드가 당황스럽지만, 몇 번 써보고 나면 그리 어려운 개념이 아니라는걸 알게 되죠. 하지만 이건 사용자 관점일 뿐, 제네릭을 한 번이라도 만들어보려고 하면 손가락이 멈칫하게 됩니다.(특히 제가 그렇습니다.) 매 번 제네릭과 관련한 무언가를 만들어보려고 할 때마다 다시 찾아보는 것이 지겨워서, 직접 한 번 정리해볼까 합니다.

# 1. Why Use Generics ?

제네릭은 클래스(혹은 인터페이스, 메소드)를 정의할 때,  `Type` 을 파라미터로 넘겨줄 수 있도록 하는 녀석입니다. 조금 바꿔 말하면, 같은 클래스여도 그 내부에 들어있는 필드나 메소드의 속성을 다르게 정의 할 수 있다는 뜻이 됩니다. 간단하게 예시를 살펴보겠습니다.

`Java Collection Framework` 안에 있는 `ArrayList`를 간단하게 직접 만들어보겠습니다.

```java
public class MyList {
    private final int capacity = 10;
    private int size;
    private Object[] array;

    public MyList() {
        this.array = new Object[capacity];
    }

    public void add(Object o) {
        array[size++] = o;
    }

    public Object get(int index) {
        return array[index];
    }
}
```

해당 MyList 안에는, 객체가 담길 수 있는 `Object` 형의 배열이 있고, 크기는 10 으로 할당해주었습니다. 이는 해당 리스트 안에는 어떠한 객체도 들어올 수 있다는 말이 되죠. 또 다시 바꿔 말하면, 이 안에는 `String`이 들어올수도, `Integer`가 들어올 수도 있다는 말이 됩니다. 따라서, 다음과 같이 사용해야 합니다.

```java
public class Main {
    public static void main(String[] args) {
        MyList myList = new MyList();
        myList.add("hello world!");
        myList.add(123);
        String str = (String) myList.get(0);
        Integer integer = (Integer) myList.get(1);
    }
}
```

이렇게 매 번 값을 꺼내올 때마다 타입을 캐스팅 해줘야 하죠. 의도한 것이라면 다행이지만, 의도하지 않았다면 실수할 여지가 다분합니다. `String` 만 넣고 싶은데, 실수로 `Integer`를 넣는다면 상당히 골치 아파지겠죠. 그렇다면 이를 방지하기 위해서, 각 타입마다 MyList를 만들어주어야 할까요?

```java
public class MyStringList {
    private final int capacity = 10;
    private int size;
    private String[] array;

    public MyStringList() {
        this.array = new String[capacity];
    }

    public void add(String o) {
        array[size++] = o;
    }

    public String get(int index) {
        return array[index];
    }
}

public class MyIntegerList {
    private final int capacity = 10;
    private int size;
    private Integer[] array;

    public MyIntegerList() {
        this.array = new Integer[capacity];
    }

    public void add(Integer o) {
        array[size++] = o;
    }

    public Integer get(int index) {
        return array[index];
    }
}
```

얼핏 보아도, 중복되는 코드가 상당히 거슬립니다. `String`, `Integer` 와 같은 녀석을 동적으로 넘겨줄 수 있다면 좋을텐데요. 앞서 했던 말을 다시 살펴보겠습니다.

> 제네릭은 클래스(혹은 인터페이스, 메소드)를 정의할 때,  `Type` 을 파라미터로 넘겨줄 수 있도록 하는 녀석입니다.  

다시 보니 제네릭이 어떤 부분에서 도움을 줄 수 있는지 알 수 있을 것 같습니다. 이렇게 특정 타입을 동적으로 넘겨주어 제한시킬 수도 있지만, 또 다른 강점은 컴파일 타임에 이를 검사할 수 있다는 점입니다. 만약 이러한 기능이 없었다면, 캐스팅을 시도하는 코드가 실행되기 전까지는 올바른 코드인지 **기계적으로** 알아낼 수 있는 방법이 없겠죠.

# 2. Generics Basic

그렇다면 제네릭을 왜 써야 하는지까지는 배웠습니다. 그렇다면 어떻게 만들 수 있을지 한번 살펴보겠습니다.

```java
public class MyList<T> {
    private final int capacity = 10;
    private int size;
    private Object[] array;

    public MyList() {
        this.array = new Object[capacity];
    }

    public void add(T o) {
        array[size++] = o;
    }

    public T get(int index) {
        return (T) array[index];
    }
}
```

변화한 점은 다음과 같습니다.

1. 클래스의 이름 뒤에 `<T>` 라는 녀석이 생겼습니다. 이 `T`가 의미하는 바가 무엇인지는 차차 살펴보기로 합시다. 우선은 `T` 라는 이름으로 `Type`을 제한한다는 점만 알아둡시다. 어쨌든 이 녀석은 "`T` 라는 이름의 제네릭을 사용하겠다" 라는 의미를 갖게 됩니다.
2. `add` 메소드와 `get` 메소드에 T 가 붙었습니다. 이는 클래스 이름 뒤에 새로 생겼던 `T` 를 활용하는 부분입니다. 만약 MyList의 `Type` 을 `String`으로 지정한다면, `add` 메소드는 파라미터로 `String o` 라는 녀석을 파라미터로 받게 될겁니다.

여기서 한 가지 신경쓸 점은, `array` 배열은 여전히 `Object` 형 이라는 점 입니다. 이 부분은 꽤 복잡한 개념을 다루어야 하므로 2편에서 다루겠습니다.

> **Type Parameter Naming Convetions**  
> 앞서 클래스의 이름 뒤에 `<T>`가 새로 생겼다고 말씀드렸는데요. 이 `T` 라는 이름은, 정해진 문법이 아닌 일종의 "약속" 입니다. 즉, Convetion 이죠. 오라클 공식 문서에서는 다음과 같이 설명하고 있습니다.  
>
> E - Element (used extensively by the Java Collections Framework)  
> K - Key  
> N - Number  
> T - Type  
> V - Value  
> S,U,V etc. - 2nd, 3rd, 4th types  

이렇게 만든 제네릭은, 다음과 같이 사용하면 됩니다. 간단하네요.

```java
public class Main {
    public static void main(String[] args) {
        MyList<String> temp = new MyList<String>();
        temp.add("World Hello ?");
        System.out.println(temp.get(0));
    }
}
```

한 가지 기존 `ArrayList`와 다른 점이 있다면, `new MyList<String>()` 인데요. 자세히 보면 `<String>` 이라는 녀석이 두 번이나 들어갔습니다. 기존의 `ArrayList<>()` 와는 대비되죠. 사실, `MyList<>()`라고 작성해도 됩니다. 이는 컴파일러가 갖고 있는 `타입 추론` 이라는 기능에 의한 것입니다. 여기서는 자세히 다룰 내용이 아니니, 궁금하신 분은 [여기](https://docs.oracle.com/javase/tutorial/java/generics/genTypeInference.html) 를 참고해보시면 되겠습니다.

또한, 제네릭에 사용할 `Type Parameter`는 다음과 같이 여러 개가 될 수 있습니다. 

```java
class Pair<K, V> {
    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey() { return key; }
    public V getValue() { return value; }
}
```

# 3. Generic Method

제네릭은 클래스 뿐만 아니라, 메소드에 한정해서 사용할 수도 있습니다.

```java
public class Util {
    public static <T> void print(T t) {
        System.out.println(t.toString());
    }

    public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2) {
        return p1.getKey().equals(p2.getKey()) &&
                p1.getValue().equals(p2.getValue());
    }
}
```

위와 같이 ReturnType 바로 앞에 Type Parameter를 명시함으로서 제네릭 메소드임을 알리고, 메소드 파라미터 혹은 그 내부에서 다음과 같이 제네릭을 사용할 수 있죠. 역시 마찬가지로, 사용할 때에는 아래와 같이 사용하면 됩니다.

```java
public class Main {
    public static void main(String[] args) {
        String str = "Hello world!";
        Util.test(str);
        
        Pair<String, Integer> pair1 = new Pair<>("FOO", 1);
        Pair<String, Integer> pair2 = new Pair<>("BAR", 2);
        Util.<String, Integer> compare(pair1, pair2);
        Util.compare(pair1, pair2); // 타입 추론에 의해 생략 가능
    }
}
```

지금까지 기본적인 제네릭의 생성, 그리고 사용법에 대해 알아봤습니다. 다음에는 제네릭을 사용할 때의 주의점에 대해 알아보겠습니다.

