---
categories: java
date: "2022-12-3T00:00:00Z"
tags: ['java', 'jvm']
title: '# Java의 static 메소드는 왜 overriding이 안되나요?'
draft: false
---

얼마전 면접을 봤는데, 갑자기 이런 질문을 받았습니다.

> Java의 static method는 왜 overriding이 안되나요 ?

static method의 overriding이라.. 너무나도 당연하게 안된다고 생각했는데, 갑작스럽게 질문을 받으니 머리가 백지상태가 되었습니다. 순간적으로 머릿속에서 JVM 구조가 얼핏 지나가면서, class loader, compile time, method area 등 여러 키워드가 지나갔으나 그 무엇도 속시원한 대답은 할 수 없을 것 같았습니다.

이번 포스팅은, JVM 에서 왜 static method를 overriding 할 수 없는지에 대해, 자세하게 알아보고자 합니다.

### 예제
아래 코드를 한번 살펴보겠습니다.(해당 코드는 반복해서 사용할 예정이니 구조를 눈여겨보시기 바랍니다.)

```java
public class StaticMethodOverride {
    public static void main(String[] args) {
        Parent whoAreU = new Child();
        whoAreU.myInstantMethod();
        whoAreU.myStaticMethod();
    }
}

class Parent {
    public static void myStaticMethod() {
        System.out.println("Parent.myStaticMethod");
    }

    public void myInstantMethod() {
        System.out.println("Parent.myInstantMethod");
    }
}

class Child extends Parent {

    public static void myStaticMethod() {
        System.out.println("Child.myStaticMethod");
    }

    @Override
    public void myInstantMethod() {
        System.out.println("Child.myInstantMethod");
    }
}
```

Parent와 Parent를 상속받는 Chid라는 두 개의 클래스를 만들었습니다. 그리고 `Child` 클래스를 구현체로 갖는 `Parent` 객체를 생성한 뒤, 각각 static method와 instance method를 호출했습니다. 그리고 그 결과는 예상했던대로, 아래와 같이 출력되었습니다.

```text
Child.myInstantMethod
Parent.myStaticMethod
```

왜 instance method는 구현체에 따라 호출되었고, static method는 타입에 따라 호출되었을까요? 그 이유는 java bytecode와 JVM이 메소드를 invoke 하는 방식을 살펴보면 알 수 있습니다.

### 바이트코드
자바 컴파일러가 `.java` 확장자를 가진 자바파일을 읽어들이면서, 작성된 자바코드를 자바 bytecode로 변환합니다. 그리고, `javap` 명령어를 통해 컴파일된 `.class` 파일, 즉 bytecode를 역어셈블할 수 있습니다.

사용한 명령어는 아래와 같습니다.

```
javac StaticMethodOverride.java && javap -c -verbose StaticMethodOverride.class
```

실행 결과를 살펴보면 꽤 복잡한 내용의 코드가 출력됩니다. 그 중, 가장 먼저 살펴볼 부분은 바로 bytecode 명령어가 담긴 부분입니다.

![](/assets/images/2022-11-13-override-static-method/2022-12-04-20-19-41.png)

알아보기가 힘들지만, 중간 즈음에 위치한 `public static void main(java.lang.String[]);` 부분이 우리의 main method가 호출되는 지점임을 확인할 수 있습니다. 

`Code:` 부분으로 넘어가서 살펴보죠. 객체를 생성하는 부분을 제외하고, 메소드를 호출하는 부분만 살펴보면 `9번`, `14번`에 위치한 `invokevirtual`과 `invokestatic`이 각각 호출했던 `instant method`와 `static method`에 대응합니다. 즉, 바꿔말하면 "호출하려는 메소드가 static metho인지, instance method인지"에 대해서는 컴파일타임에 알 수 있다는 점입니다.

> 각각의 명령어가 `invokevirtual`, `invokestatic` 이었다는 사실을 기억해주세요.

그런데 또 자세히 살펴보면, `#4` 와 `#5` 라는 숫자가 써있네요. 이 부분은 `constant pool` 이라고 하는 곳의 index를 나타냅니다. `constant pool`을 확인해보겠습니다.

![](/assets/images/2022-11-13-override-static-method/2022-12-04-20-30-09.png)

각각의 `#4`, `#5` 번은 `MethodRef` 라는 의미를 내포하면서, 각각 `#18.#19`, `#18.20` 에 대한 index를 다시 나타내고 있습니다. 해당 인덱스를 따라가보면, `#18`은 `Class`를, `#19`는 다시 `NameAndType`을 가리킵니다. 최종적으로 쭉 따라가다보면, 결국 `Utf8` 이라는 타입으로 귀결됩니다.

### static
위와 같은 과정을 통해, 우리는 java의 클래스나 메소드가 위와같이 재귀참조를 통해 시그니쳐를 나타낸다는 것까지 확인했습니다. 그리고, 호출하는 메소드의 클래스는 슈퍼클래스인 `Parent` 라는 것까지 확인했죠. 그렇다면 여기서 우리는 메소드의 호출은 "타입을 따라간다" 라고 추론할 수 있습니다. 이를 증명하기 위해, 서브클래스인 `Child`를 타입으로 객체를 생성하여 static method를 호출해보겠습니다.

```java
Child.myStaticMethod();
/* 결과
17: invokestatic  #6  // Method study/java/staticmethod/Child.myStaticMethod:()V
*/
```

위와같이 자식 타입에 대한 static method를 호출하고, 바이트코드를 살펴보면 아까와는 다르게 `Child`에 대한 메소드를 호출함을 볼 수 있습니다. 즉, 슈퍼클래스와 서브클래스의 static method signature는 일치할 수 있으며, 우리는 이를 `overriding`이 아닌 `hiding` 이라고 부릅니다.

### invokevirtual, invokestatic
그런데 이상한점이 있습니다. `invokestatic`이야 Parent 클래스를 가리키는 것은 그렇다 쳐도, `invokevirtual` 또한 constant pool을 따라가다보면 Parent 클래스를 가리킨다는 점입니다. 그런데 어떻게 `Child` 메소드가 호출될 수 있을까요? 이 때 JVM이 메소드를 호출하는 방식, 즉 `dynamic dispatch(혹은 dynamic binding)`이 동작하는 시점입니다.

JVM Specification에서는 다음과 같이 설명합니다.
> invokevirtual invokes an instance method of an object, dispatching on the (virtual) type of the object. This is the normal method dispatch in the Java programming language.

즉, `invokevirtual` 은 "객체"의 타입을 보고 메소드를 호출한다는 것입니다. 그런데 이것만으로도 설명은 부족합니다. "객체"의 타입을 어떻게 보고, 어떻게 메소드를 호출한다는 것일까요 ? 

> 여기서 `virtual` 이라는 키워드는 `C++` 의 `virtual`과 연관이 깊습니다. 이와 관련된 내용은 본 포스팅에서 다루지 않습니다. 단, `java`의 모든 메소드는 `final` 이나 `static` 키워드를 붙이지 않았다면 기본적으로 `virtual` 메소드라는 점만 짚고 넘어가겠습니다.

객체에 대한 reference인 `whoAreU` 변수 그 자체는 사실 내부적으로 두 개의 포인터를 갖고 있습니다. 하나는 우리가 일반적으로 알고 있는, 힙 영역에 존재하는 객체 데이터에 대한 포인터이고, 다른 하나는 **객체의 타입과 메소드 테이블을 가지고 있는 포인터**입니다.

그리고 후자에 해당하는 포인터를 따라, 간접적으로 메소드를 호출할 수 있게 되는것입니다. `invokevirtual`이 동작하는 방식 또한 역시 JVM specification에 명시되어 있습니다.


> Let C be the class of objectref. 
> 
> The actual method to be invoked is selected by the following lookup procedure:
> If C contains a declaration for an instance method m that overrides (§5.4.5) the resolved method, then m is the method to be invoked, and the lookup procedure terminates.
> 
> Otherwise, if C has a superclass, this same lookup procedure is performed recursively using the direct superclass of C; the method to be invoked is the result of the recursive invocation of this lookup procedure.
> 
> Otherwise, an AbstractMethodError is raised.

즉, 어떠한 메소드를 호출할 때 자기 자신이 호출되어야할 메소드를 갖고있는지 확인하고, 그렇지 않다면 슈퍼 클래스로 이동해서 확인하게 됩니다. 하지만 이는 명세일뿐, 실제 JVM이 구현하는 방식은 다릅니다. 아무래도 매번 이러한 호출과정을 거치게 되면 속도가 느려지겠죠. 따라서 `C++`에서 차용한 개념인 `method table` 을 이용해 위 과정을 최적화했습니다.

각 JVM의 밴더사마다 `method table`의 구현 방식은 다르지만, 대략 아래와 같이 구현되어있을 것이라고 기대할 수 있습니다.

> **Object class**
> |Name|Address|Description|
> |:--:|:--:|:--:|
> |Object.getClass()|0x01|Object.getClass()|
> |Object.toString()|0x02|Object.toString()|
> (기타 다른 메소드는 생략)

ClassLoader(아마 Boostrap)에 의해 `Object` 클래스가 로딩되고 나면, `Object` 클래스가 갖고 있는 여러 메소드들에 대한 `method table`이 초기화됩니다. 그리고, 기타 사용자(개발자)가 정의한 클래스가 런타임중에 로딩되는 순간, 각각의 메소드테이블이 초기화됩니다. 

`Parent` 클래스가 ClassLoader(아마 Application)에 의해 로딩되면, `Object` 클래스로부터 `method table`을 복사한뒤, 자기 자신이 갖고있는 `signature`에 따라 `method table`을 업데이트합니다. 따라서, `Parent`의 `method table`은 아래와 같이 구성될 것입니다.

> **Parent class**
> |Name|Address|Description|
> |:--:|:--:|:--:|
> |Object.getClass()|0x01|Object.getClass()|
> |Object.toString()|0x02|Object.toString()|
> |**Parent.myInstantMethod()**|**0x03**|**Parent.myInstantMethod()**|
> (기타 다른 메소드는 생략)

`Child` 클래스 역시 마찬가지로, 자신의 슈퍼클래스인 `Parent` 클래스로부터 `method table`을 복사한뒤 업데이트 과정을 거칩니다. 그런데 이 때, `Child` 클래스는 `myInstantMethod()`를 overriding 합니다. 따라서, `method table`은 아래와 같이 업데이트됩니다.

> **Child class**
> |Name|Address|Description|
> |:--:|:--:|:--:|
> |Object.getClass()|0x01|Object.getClass()|
> |Object.toString()|0x02|Object.toString()|
> |Parent.myInstantMethod()|0x03|**Child.myInstantMethod()**|
> (기타 다른 메소드는 생략)

자, 이제 정리해보겠습니다. JVM이 명령어를 실행하던 도중, `invokevirtual`을 마주하고 target이 되는 `object`와 `arguments` 들을 `operand stack` 에서 꺼냅니다. 그리고, 해당 object의 `method table`을 뒤져보고, 호출할 대상이 되는 메소드를 찾아냅니다. 그리고, `object`와 `arguments`로 새로운 스택 프레임을 구성하면서 메소드 호출이 이뤄지게 됩니다.

### 나가며

method table 이란 것에 대해서 막연하게만 알고 있었는데, 이번 기회를 통해 좀 깊게 파볼 수 있었던 경험이 된 것 같습니다. 이 글을 읽으시는 여러분들도 JVM이 어떻게 메소드를 호출하는지에 대해 원리를 이해하는데 조금이나마 도움이 되셨으면 좋겠습니다.

참고했던 자료들이 조금 오래된 자료들이라, 현재(Java 8, 11, 17 ...)와는 조금 다를수도 있습니다. 틀린 내용이 있으면 언제든 지적 바랍니다. 감사합니다.

---

### reference

- https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-6.html#jvms-6.5.invokevirtual
- https://www.artima.com/insidejvm/ed2/jvmP.html
- https://en.wikipedia.org/wiki/Virtual_method_table
- https://www.programmingmitra.com/2017/05/how-does-jvm-handle-method-overriding-internally.html
