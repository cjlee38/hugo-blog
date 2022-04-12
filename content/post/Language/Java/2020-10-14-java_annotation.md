---
# author: cjlee
categories: java
# cover: /assets/covers/coding.png
date: "2020-10-14T21:11:00Z"
tags: ['null']
title: '# (Java) Annotation에 대한 이해'
---

# 0. 들어가며
: 스프링 공부를 하던 중, 자주 사용되는 Annotation에 대해서 살펴보았다.  
다양한 글을 읽었는데도, Java의 Annotation은 뭐라 딱 찝어서 설명하기 어려운 것 같다.  
더욱이, 나는 **'@'** 표시에 대해서, Python의 Decorator로 먼저 이해했기 때문에 더욱 어려웠던 것 같다.

Python의 Decorator는 마치 함수를 감싸는 함수의 역할과 같은 모습을 볼 수 있지만,  
Java의 Annotation은 그냥 **Meta data** 로 퉁쳐버리기 때문에,  
*"정말 그게 다야? 뭐 더 없어? 아닐 것 같은데?"* 싶은 생각이 계속 들어서 이해하기 어려웠다.

# 1. Java Annotation
: Java의 Annotation은, 곧 **메타 데이터**다.   
JDK 5에서 추가된 어노테이션은 소스코드가 동작하는 데에는 영향을 미치지 않는다.   
그저 Class, Interface, method, field 등에 붙어서,   
Java Compiler와 JVM에게 **추가적인 정보**를 제공한다.

그냥 이게 전부다. 여기서 더 설명할 것이 없다.   
Python decorator와 헷갈리는 사람은 스킵하고 쭉 내려가서 4번을  읽어보자.

> Meta data 란, 데이터에 대한 데이터, 혹은 데이터를 설명하기 위한 데이터를 의미한다.

이에 더해, 배포 or 런타임 시에도 사용될 수 있다.

# 2. Built-in Annotations in Java
## 1) Override
: 자주 보는 예시 중 하나가, `@Override` 이다.  
사실, 이러한 Override가 없어도, 실행은 잘 된다(위에서 언급했듯이, 소스코드의 동작에는 영향을 미치지 않는다.)

그런데 이를 사용하는 이유는 무엇일까?  
다음과 같은 상황에서, 컴파일러가 미리 알려준다.

- Super class(interface)에서, 그러한 method가 없는 경우, 즉 이름을 잘못 적은 경우

```java
@Override
public String toStrring() {
    return "something";
}
```
모든 클래스의 최상위 클래스인 Object 클래스에 정의된  
toString method를 Override 하려고 했는데, 실수로 toSt**rr**ing 으로 적었다.  
이러한 경우에 컴파일러가 에러를 뱉어준다.

## 2) Deprecated
: Deprecated 는, 마킹된 요소(클래스, 메소드, 필드)를 더이상 사용하지 않을 것을 권장할 때 사용된다.   
다음의 코드를 보자.

```java
public class Main {
    public static void main(String[] args) {
        MyClass myClass = new MyClass();
        myClass.myMethod();
    }
}

class MyClass {
    @Deprecated
    void myMethod() {
        System.out.println("hello world");
    }
}
```
실행은 정상적으로 이루어지지만, IDE가 myMethod()에 줄을 쫙 그어준다.  
또한, myMethod()는 Deprecated 되었다고 Warning도 띄워준다.

![](/assets/images/2020-10-24-22-06-23_2020-10-14-java_annotation.md.png)

## 3) SupperssWarnings
: 이름에서 나타내듯이, 컴파일러가 특정 warning을 무시하도록 해준다.  
2)에서 사용했던 코드에서, main method에 `@SupressWarnings("deprecation")`을 추가해보자.

![](/assets/images/2020-10-24-22-08-55_2020-10-14-java_annotation.md.png)

경고가 사라진 것을 볼 수 있다.

# 3. 살짝 뜯어보기
: Annotation이 어떻게 정의되어 있는지, 살짝만 뜯어보자.

`@Override` 를 살펴보면, 다음과 같이 정의되어 있다.
```java
/**
 * Indicates that a method declaration is intended to override a
 * method declaration in a supertype. If a method is annotated with
 * this annotation type compilers are required to generate an error
 * message unless at least one of the following conditions hold:
 *
 * <ul><li>
 * The method does override or implement a method declared in a
 * supertype.
 * </li><li>
 * The method has a signature that is override-equivalent to that of
 * any public method declared in {@linkplain Object}.
 * </li></ul>
 *
 * @author  Peter von der Ah&eacute;
 * @author  Joshua Bloch
 * @jls 8.4.8 Inheritance, Overriding, and Hiding
 * @jls 9.4.1 Inheritance and Overriding
 * @jls 9.6.4.4 @Override
 * @since 1.5
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}

```
`@Target` Annotation은 "Annotation이 적용될 수 있는 요소" 를,  
`@Retention` Annotation은, "이 Annotation이 언제까지 살아있는지?" 를,  
`@interface`는 이 인터페이스가 Annotation으로 사용될 것을 의미한다.  

`@Target`의 값에 포함되는, enum ElementType은 다음과 같다.  
(주석이 너무 길어서, 주석을 제외하고 가져왔다.)

```java
public enum ElementType {
    TYPE,
    FIELD,
    METHOD,
    PARAMETER,
    CONSTRUCTOR,
    LOCAL_VARIABLE,
    ANNOTATION_TYPE,
    PACKAGE,
    TYPE_PARAMETER,
    TYPE_USE,
    MODULE,
    @jdk.internal.PreviewFeature(feature=jdk.internal.PreviewFeature.Feature.RECORDS, essentialAPI=true)
    RECORD_COMPONENT;
}
```

몇 가지 눈에 보이는 점은,  
필드, 메소드, 생성자.. 를 넘어서, 파라미터, 패키지 등에 붙일수도 있음을 볼 수 있다.

`@Retention` 의 값에 포함되는, enum RetentionPolicy는 다음과 같다.

```java
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```

- Source : 컴파일러에 의해 버려진다.
- Class : 컴파일러에 의해 클래스파일에 남아있지만, 런타임시에는 유지되지 않는다.
- RUNTIME : 컴파일러에 의해 클래스파일에 남아있고, 런타임 시에도 남아있다.

# 4. 가볍게 Custom Annotation 만들어보기.
: 위에서 본 내용을 바탕으로, Custom Annotation을 만들어보자.

```java
public class Main {
    public static void main(String[] args) throws NoSuchFieldException {
        MyClass myClass = new MyClass("foo");

        MyAnnotation myClassAnnotation = myClass.getClass()
                                                .getAnnotation(MyAnnotation.class);
        System.out.println("myClassAnnotation value = " + myClassAnnotation.value());

        MyAnnotation myFieldAnnotation = myClass.getClass()
                                                .getDeclaredField("myField")
                                                .getAnnotation(MyAnnotation.class);
        System.out.println("myFieldAnnotation value = " + myFieldAnnotation.value());
    }
}

@MyAnnotation(value = 1)
class MyClass {

    @MyAnnotation(value = 10)
    private String myField;

    
    public MyClass(String myField) {
        this.myField = myField;
    }
}

@Target({ElementType.TYPE, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation {
    int value() default 0;
}
```

`MyAnnotation` 이라는 Annotation을 만들었다.  
이 때, **Retention은 RUNTIME**, **Target은 클래스와 Field**에 붙일 수 있도록 했다.  
그리고 `MyClass` 클래스에 1이라는 value를, `MyField`는 10을 붙였다.  
여기서 주의할 것은, value 라는 것은 meta data의 일부이지,  
value를 가져야만 meta data를 가질 수 있는 것은 아니다.  
다시 말해, value가 없어도 된다.

만약, Constructor에 MyAnnotation을 붙이려고 하면   
Target이 Class와 Field에만 붙일 수 있도록 했기 때문에  
`'@MyAnnotation' not applicable to constructor` 라고 에러를 뱉는다.

위와 같이 코드를 작성하고 main method를 실행하면, 결과는 다음과 같다.


```
myClassAnnotation value = 1
myFieldAnnotation value = 10
```

예상한대로 결과가 잘 나타났다.   
만약 `RetentionPolicy.RUNTIME`을 CLASS, 혹은 SOURCE로 변경할 경우,  
run 하면 NullPointerException이 발생한다.

# 4. Python의 decorator와의 차이(번역)

[스택오버플로우의 한 질문글](https://stackoverflow.com/questions/15347136/is-a-python-decorator-the-same-as-java-annotation-or-java-with-aspects)에서, 나의 혼란을 정확히 짚어주는 한 답변을 보았다.  

간략하게 번역하면 다음과 같다.

> 이는 두 언어를 동시에 사용하는 사람이라면 가질 수 있는 아주 타당한 질문입니다. 나는 파이썬을 잠시 사용했고, 최근에는 자바를 사용하고 있습니다. 두 차이에 대한 비교의 제 생각은 다음과 같습니다.
>
> Java의 Annotation은 그저, 말 그대로 Annotation(주석) 입니다. 이는 마커로, 객체에 대한 추가적인 Meta data의 컨테이너입니다. 이것들이 단순히 존재한다고 해서, 실행흐름을 바꾸거나 어떠한 캡슐화를 추가하지 않습니다. 그렇다면 어떤 쓸모가 있을까요? Annotation Processor라는 것이 Annotation을 읽고, 처리합니다. Annotation이 갖고 있는 메타데이터는 사용자가 정의한 Annotation processor에 의해 사용되어, 삶을(코딩을) 좀 더 편하게 하는 보조적인 역할을 합니다. 하지만, 다시 말씀드리겠습니다. 이것들은 기본적인 실행의 흐름을 바꾸거나, 감싸지 않습니다.
>
> Python decorator를 사용해본 사람은, 실행 흐름을 바꾸지 않는 것에 대한 강조를 명확히 이해할 것입니다. Python의 decorator는 Java의 annotation과 비슷하게 보이고 느껴지지만, 그 내면은 상당히 다릅니다. decorator는 사용자가 원하는대로 감쌀 수 있고, 원한다면 실행하는것 자체를 막을수도 있습니다. 내부요소를 가져다 감싸고, 그 감싼 것으로 원래의 것과 교체합니다. decoraor는 효과적으로 요소를 "proxying" 할 수 있습니다.
>
>... 하략

기존에 Annotation에 대해 너무 어렵게 생각했나 싶기도 하다.   
이상으로 포스팅을 마칩니다.