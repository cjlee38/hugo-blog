---
# author: cjlee
categories: java
# cover: /assets/covers/coding.png
date: "2021-08-29T23:44:00Z"
tags: ['null']
title: '# 제네릭 톺아보기 2'
---

# Java Generic 2

# 0. 들어가며

지난 시간에는 제네릭이 왜 필요한지, 어떻게 사용하는지, 그리고 어떻게 만들 수 있는지 등에 대해 살펴봤습니다.
그런데 한 가지, List 를 흉내낸 MyList 의 경우 생성자에서 여전히 Object 형을 사용했었습니다. 지난 시간의 코드를 다시 가져와보겠습니다. 

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

`private Object[] array` 그리고  `this.array = new Object[capacity];`  이 부분인데요. 얼핏 생각하면 `private T[] array` `this.array = new T[capacity];` 이렇게 생성하면 더 깔끔할 것 같은데, 왜 그렇게 하지 않을까요? 여기에는 (저에게) 복잡한 엔지니어링의 철학이 스며들어있는데요. 이를 자세히 살펴보겠습니다.

# 1. Basis (1) `Covariant` vs `Contravariant` vs `Invariant`

갑작스럽게 익숙치 않은 두 단어가 나타납니다.  `covariant`는 공변, `contravariant`는 반공변, `invariant`는 무공변 이라는 단어로 해석되는데, 단어만 놓고 봤을때는 무슨 의미인지 파악하기가 조금 어렵습니다. 간략하게 정리하자면 다음과 같습니다.

> A, B 가 타입이고, f는 타입의 변경이라고 가정합니다.  
>
> * B가 A의 서브타입일 때, f(B)는 f(A)의 서브타입이다. -> 이 때 f는 공변입니다.  
> * B가 A의 서브타입일 때, f(A)는 f(B)의 서브타입이다. -> 이 때 f는 반공변입니다.  
> * A와 B가 아무런 관계를 갖지 않는다 -> 이 때 f는 무공변입니다.  

헷갈리니, 조금 더 자세히 풀어봅시다. 자바에서 배열은 다음과 같이 작성할 수 있습니다.

```java
Object[] array = new String[10];
```

Object와 String은 하나의 타입이고, 따라서 배열의 경우 f(Object)는 `Object[]` 로 만드는 것이고, f(String)은 `String[]` 이 되겠죠. 이러한 경우에, `String[]` 은 `Object[]` 의 서브타입이 될 수 있으므로, 공변입니다.

한편, 제네릭은 어떨까요?

```java
List<Object> list = new ArrayList<String>();
```

위와 같은 코드는 컴파일 되지 않습니다. 당연히, `List<String> list = new ArrayList<Object>();`와 같은 반공변 코드도 안되겠죠. 따라서 제네릭은 무공변입니다. 즉, 제네릭은 타입을 가지고 상속관계를 결정지을 수 없다는 얘기가 되죠. List<Object>와 List<String>은 얼핏 생각하기에 상속관계를 가질 것 같지만, 실제로 둘은 관계가 없습니다.

> Note 1. 하지만 같은 타입이라면 상속관계가 성립합니다. 즉, Collection<String>과 List<String>, 그리고 ArrayList<String>은 [추이적 관계](https://ko.wikipedia.org/wiki/%EC%B6%94%EC%9D%B4%EC%A0%81_%EA%B4%80%EA%B3%84)를 갖습니다.  

> Note 2. 공변, 반공변, 무공변 등은 프로그래밍 언어의 설계적인 특성입니다. 가령, 다음과 같은 코드는 자바에서 올바르게 Override 할 수 있습니다.  
>
> ```java  
> class Super {  
> 	Object getSomething() {}  
> }  
> class Sub extends Super {  
> 	String getSomething() {}  
> }  
> ```
>
> 이를 **Covariant Return Type** 이라고 부르고, JDK 1.5 부터 생겨난 기능입니다.  
> 한편, 다음과 같은 코드는 자바에서 Override가 아닌, Overload 됩니다. 타 언어에서는 Override가 가능합니다.  
>
> ```java  
> class Super {  
> 	void doSomething(String param) { ... }  
> }  
> class Sub extends Super {  
> 	void doSomething(Object param) { ... }  
> }  
> ```

# 2. Basis (2) Type Erasure

Type Erasure는 제네릭이 JDK 1.5부터 도입되었기 때문에, 이전 버전에 작성된 코드와의 호환성을 위해 도입된 기능입니다.  Type Erasure의 기능은 다음 세 가지로 요악할 수 있습니다.

1. 제네릭의 타입 파라미터(e.g. T)를 일반적인 클래스, 인터페이스 등으로 교체합니다. 만약 bound(경계)가 명시되어 있는 경우, 해당 bound로 교체하고, unbounded(경계가 없는) 인 경우 Object로 교체합니다.
2. 필요하다면, 타입 캐스팅을 집어 넣습니다.
3. 다형성을 유지하기 위해, `브릿지 메소드`를 생성합니다.

2번은 간단하니, 1번과 3번만 살펴봅시다

## 2-1. Replace Type Parameters

다음과 같은 코드가 있다고 가정해봅시다.

```java
public class Node<T> {

    private T data;
    private Node<T> next;

    public Node(T data, Node<T> next) {
        this.data = data;
        this.next = next;
    }

    public T getData() { return data; }
    // ...
}
```

여기서 T는 unbounded, 즉 경계가 없기 때문에 다음과 같이 Object로 교체됩니다.

```java
public class Node {

    private Object data;
    private Node next;

    public Node(Object data, Node next) {
        this.data = data;
        this.next = next;
    }

    public Object getData() { return data; }
    // ...
}
```

만약 T가 다음과 같이 Comparable이라는 경계를 갖고있다면, 

```java
public class Node<T extends Comparable<T>> {

    private T data;
    private Node<T> next;

    public Node(T data, Node<T> next) {
        this.data = data;
        this.next = next;
    }

    public T getData() { return data; }
    // ...
}
```

해당 bound의 클래스(혹은 인터페이스) 로 교체됩니다.

```java
public class Node {

    private Comparable data;
    private Node next;

    public Node(Comparable data, Node next) {
        this.data = data;
        this.next = next;
    }

    public Comparable getData() { return data; }
    // ...
}
```

여기서 말하는 "경계" 가 무엇인지 모르셔도 괜찮습니다. 다음 3편에서(😂) 설명하겠습니다.

## 2-2. Bridge Method

Bridge Method는 타입 삭제로 인해 발생할 수 있는 문제를 해결하기 위해 나타난 기능입니다. 다음과 같은 코드가 있다고 가정해봅시다.

```java
public class Node<T> {

    public T data;

    public Node(T data) { this.data = data; }

    public void setData(T data) {
        System.out.println("Node.setData");
        this.data = data;
    }
}

public class MyNode extends Node<Integer> {
    public MyNode(Integer data) { super(data); }

    public void setData(Integer data) {
        System.out.println("MyNode.setData");
        super.setData(data);
    }
}
```

그리고, 위와 같이 정의된 클래스를 다음과 같이 사용하겠습니다.

```java
MyNode mn = new MyNode(5);
Node n = mn;            // A raw type - compiler throws an unchecked warning
n.setData("Hello");     // Causes a ClassCastException to be thrown.
Integer x = mn.data;
```

타입 제거가 발생한다면, 컴파일 이후는 다음과 같은 모습일텐데요.

```java
MyNode mn = new MyNode(5);
Node n = (MyNode)mn;         // A raw type - compiler throws an unchecked warning
n.setData("Hello");          // Causes a ClassCastException to be thrown.
Integer x = (String)mn.data; 
```

실제로 실행하려고 보면, ClassCastException 이라는, 다소 당황스러운 에러 메시지가 발생하게 됩니다. 그 이유는, 위에 정의된 `Node`, 그리고 `MyNode`의 타입 제거 이후의 모습과, 브릿지 메소드가 생성되었을 때의 모습을 살펴보면 알 수 있습니다. 가장 먼저, 타입 제거가 된 모습은 다음과 같습니다.

```java
public class Node {

    public Object data;

    public Node(Object data) { this.data = data; }

    public void setData(Object data) {
        System.out.println("Node.setData");
        this.data = data;
    }
}

public class MyNode extends Node {

    public MyNode(Integer data) { super(data); }

    public void setData(Integer data) {
        System.out.println("MyNode.setData");
        super.setData(data);
    }
}
```

위 코드를 보면, MyNode가 Node 클래스를 상속하고 있는데, `Node` 클래스의 `setData()` 메소드와 `MyNode` 클래스의 `setData()` 메소드의 시그니처가 다릅니다. 하나는 Object를, 하나는 Integer를 파라미터로 받고 있는데요. 이렇게 될 경우 `setData` 메소드는 override가 아닌 **overload** 가 되겠죠. 코드가 여기까지만 변환되었다면, `n` 변수가 호출하는 `setData` 메소드는 `mn` 변수가 호출하는 `setData`는 다른 메소드이니 문제가 발생하게 됩니다. 즉, 다형성을 유지할 수 없어지게 되는데요. 이러한 문제를 해결하기 위해, 컴파일러는 다음과 같은 메소드를 클래스 내에 삽입합니다.

```java
public void setData(Object data) {
    setData((Integer) data);
}
```

이러한 메소드를 브릿지 메소드라 부르고, 해당 메소드내에 있는 `(Integer)` 캐스팅으로 인해 `ClassCastException`이 발생하게 됩니다.

# 3. So ...

본격적으로 왜 Object[] array를 만들수 없는지에 대해 알아보겠습니다. 타입 파라미터 형태의 배열을 만들 수 없는 이유를 말하는데 뭐 이리 잡설이 기냐 라고 얘기할 수도 있겠습니다만, 이러한 요소들을 이해하고 있지 않으면, 그 이유를 이해하기가 어렵기 때문입니다.

앞서, 자바의 배열은 공변이라고 말씀드렸습니다. 따라서, 다음과 같은 코드는 문제가 없습니다.

```java
Object[] array = new String[10];
```

하지만, 반공변은 아니기 때문에, 다음과 같은 코드는 당연히 실행할 수 없겠죠.

```java
Object[] array = new Object[10];
Integer[] iarray = array;
```

그렇다면 지난 시간의 MyList 코드를 가져와서, 타입 파라미터의 배열을 만들 수 있다고 가정해보겠습니다. 그리고, 해당 배열을 가져오는 함수도 하나 만들어보죠.

```java
public class MyList<T> {
    private final int capacity = 10;
    private int size;
    private T[] array;

    public MyList() {
        this.array = new T[capacity];
    }

    public void add(T o) {
        array[size++] = o;
    }

    public T get(int index) {
        return (T) array[index];
    }

    // 새로 추가한 코드
    public T[] getArray() {
        return array;
    }

}
```

그리고, 다음과 같이 사용해보겠습니다.

```java
MyList<String> myList = new MyList<>();
String[] array = myList.getArray();
```

겉보기에는 멀쩡해보이는데요. myList에서 얻어온 getArray() 는 타입 파라미터로 넣어준 String[]의 배열이고, 이를 `String[] array` 에서 받아내고 있으니까요. 

그런데, 앞서 이야기 했던 Type Erasure에 대해 다시 생각해보겠습니다. Type Erasure는 **Unbounded인 경우 타입 파라미터를 모두 Object로 교체한다**고 했었는데요. 그렇다면 실제 컴파일 이후 MyList의 생성자 부분 코드는 `this.array = new Object[capacity];` 이겠네요. 역시 마찬가지로, getArray() 함수 또한 `Object[]` 배열을 돌려줄 것이구요. 그런데 이를 `String[]` 배열에서 받아내고 있습니다. 이는 금방 보았던 공변성에 어긋나게 되고, 문제가 발생하게 됩니다.

# 4. Plus

## 4-1. Casting to Type Parameter's array

타입 파라미터의 배열 생성이 불가능하다면, Object의 배열을 생성한 뒤, 타입 파라미터로 캐스팅하는 것은 어떨까요? 즉 다음과 같습니다.

```java
T[] array = (T[]) new Object[10];
```

이 또한 괜찮은 선택지인 것 처럼 보이지만, 실제로는 잘못된 다운캐스팅(Downcasting) 으로 인해 실패하게 됩니다. 생성하는 객체가 Object 객체의 배열이기 때문에, 이를 다른 타입의 객체 배열로 캐스팅하는 것은 성립이 되지 않습니다. 다운캐스팅은 다음과 같이 실제로 생성하는 객체가 해당 타입이거나, 혹은 그 상위일때에만 가능합니다.

```java
Object[] array = new String[10];
String[] sArray = (String[]) array;
```

여기서 `new String[10];` 을 `new Object[10];` 으로 바꾸게 되면 `ClassCastException`이 발생합니다.


## 4-2. Array of Generic Class

 다음으로 살펴볼 예시는, 제네릭 클래스의 배열입니다.

```java
List<String>[] arrOfList = new ArrayList<String>[10];
```

이와 같은 코드는 실제로 컴파일 되지 않지만, 가능하다고 가정해봅니다. 

```java
Object[] objarr = arrOfList;
objarr[0] = new ArrayList<Integer>();
```

이 때, 위와 같이 코드를 작성하면, 문제가 발생하지 않습니다. Type Erasure로 인해 런타임 당시에는 결국 `List<String>[]` 이 아닌 `List[]` 니까요. 위에서는 List<String>만 받기로 했는데, List<Integer>를 할당하고 있으니 예외가 발생해야 하는데, 그렇지 않죠. 애초에 제네릭의 탄생 목적이 사용하는 타입의 안정성을 보장하기 위한 것인데(즉, 제한된 종류의 타입만 한정지어서 사용하도록 하고, 이에 관련된 문제는 컴파일 도중에 잡아낼 수 있도록이죠), String을 넣기로 약속한 곳에 Integer를 넣을 수 있다면 제네릭의 안정성이 전혀 보장될 수 없습니다. 그렇기 때문에 애초에 `Generic Array Creation` 이라는 경고 문구가 등장합니다.

예외가 발생한다고 해서, 다음과 같은 코드를 작성해서도 안됩니다.

```java
List<String>[] listOfArr = new ArrayList[10];
```

아예 제네릭을 빼버리는건데요. 그러면 다음과 같이 흐름이 이어질 경우 역시 타입 캐스팅 도중에 `ClassCastException`이 발생하게 됩니다.

```java
Object[] objarr = listOfArr;

List<Integer> iList = new ArrayList<>();
iList.add(123);
objarr[0] = ilist;

System.out.println(strlistarr[0].get(0)); // exception occurs here
```

억지를 부려서 `List<String>`만 받기로 약속한 `listOfArr` 에 `List<Integer>`를 넣어줬는데요. 꺼낼때에는 `listOfArr`은 "당연히 내 안에 들어있는 녀석들은 String이겠군" 하고 캐스팅을 하려다 문제가 발생하게 됩니다. 지난 편에 제네릭을 소개할 때 보여드렸던, 실수할 수 있는 케이스와 결국 같은 맥락입니다.

지금까지 제네릭 속에 들어있는 프로그래밍 이론에 관해서 다뤄보고, 이에 기인한 제네릭의 주의점에 대해서 알아보았습니다. 꽤나 복잡하고 많은 양을 다루었다고 생각했는데, 아직도 갈 길이 멉니다. 다음 시간에는 타입 경계, 그리고 와일드카드에 대해서 이야기해보고자 합니다. 감사합니다.

## 5. Reference

[공변성과 반공변성은 무엇인가? | edykim](https://edykim.com/ko/post/what-is-coercion-and-anticommunism/)
[java - How to create a generic array? - Stack Overflow](https://stackoverflow.com/questions/18581002/how-to-create-a-generic-array)
[Covariance and contravariance (computer science) - Wikipedia](https://en.wikipedia.org/wiki/Covariance_and_contravariance_%28computer_science%29)

