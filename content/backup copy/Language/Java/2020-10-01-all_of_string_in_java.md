---
# author: cjlee
categories: java
# cover: /assets/covers/coding.png
date: "2020-10-01T16:59:00Z"
tags: ['null']
title: '# (Java) 추석맞이 String 종합선물세트'
---

# 0. 들어가며
: String은 언제 다뤄도 어렵다.  
Java 에서의 String은 조금 특별한 존재이기 때문에, Python처럼 생각없이 다루면 안된다.  

String에 연산이 많이 들어가는 경우, StringBuilder를 쓰는 경우가 많은데,  
도대체 어떤 차이가 있는지, String은 어떻게 변환되는지에 대해  
이번 포스팅을 통해 짚어보고자 한다.

# 1. String

## String Constant Pool
: 기본적으로, String은 Primitive Type이 아닌, Reference Type이다.  
즉, 우리가 바라보고 있는 특정 String은 그 자체로서 값을 갖고 있는 것이 아니고,  
그 값에 대한 **"포인터"** 인 셈이다.  
이 포인터는 **Stack 영역**에서 관리된다.

그렇다면, 그 실제적인 값을 갖고 있는 녀석은 어디에 있을까?  
바로 **Heap 영역의 String Constant Pool**에 있다.

다음을 보자.

```java
public class Main {

    public static void main(String[] args) {
        String s1 = "hello";
        String s2 = "hello";
        String s3 = new String("hello");
        String s4 = new String("hello");
        
        System.out.println(s1 == s2); // true
        System.out.println(s1 == s3); // false
        System.out.println(s3 == s4); // false

    }
}
```

다 같은 "hello" 라는 string을 담고 있는데,  
s1과 s2는 같은 반면, s1과 s3는 다르다. 이에 더해, **s3와 s4 또한 다르다.**  
왜 이런 차이를 보이는 것일까? 이는 다음의 그림을 보면 알 수 있다.

![string_constant_pool](/assets/images/string-constant-pool-latest.png){: .alignCenter}

{: .caption}
[사진 출처](https://www.javaguides.net/2018/07/guide-to-java-string-constant-pool.html)

기본적으로, 우리가 `String str = "hello";` 와 같이 String을 만들면,  
이는 **String literal** 이라고 부르고, 다음과 같이 동작한다.

1. String (constant) Pool 에서, 동일한 String이 있는지 찾는다.
2. 동일한 String을 발견하면, 해당 객체를 가리키도록 한다.
3. 동일한 String을 발견하지 못하면, String Pool에 새로운 String을 만들고 이를 가리킨다.

하지만, `String str = new String("hello");` 와 같이 작성하면,   
우리가 일반적으로 관리하는 클래스처럼, Heap 영역에 해당 String을 만들어버린다.  
**위 코드의 s3와 s4는 같지 않다고 나온 이유가 이 때문이다.**

따라서, 사람의 입장에서 "String이 같은가요?" 라고 물어보려면,
equals() 함수를 사용해야 한다.

equals() 함수를 따라가보면 다음과 같이 작성되어 있다.

```java
@HotSpotIntrinsicCandidate
    public static boolean equals(byte[] value, byte[] other) {
        if (value.length == other.length) {
            for (int i = 0; i < value.length; i++) {
                if (value[i] != other[i]) {
                    return false;
                }
            }
            return true;
        }
        return false;
    }
```

String을 initialize 하면, String은 byte[] 라는 배열로 값이 들어가므로,  
비교하고자 하는 value 라는 byte 배열과, 그 대상인 byte[] other 를 하나씩 비교해가면서  
**하나라도 다를 경우** false를 return한다.

어찌되었든, Heap 영역과, 그 안의 String Pool에 String이 저장된다는 것은 알았다.  

## String as Immutable
: Java에서의 String은 immutable(불변) 이라고 한다. 이게 도대체 무슨 뜻일까?  
바꿔 말해서, String에 값을 변경하면 어떻게 될까?

다음의 코드를 보자.

```java
public class Main {

    public static void main(String[] args) {
        String s1 = new String("hello");
        String s2 = s1;
        String s3 = new String("helloworld");
        String s4 = "helloworld";

        System.out.println(s1 == s2); // true

        s1 += "world";
        System.out.println(s1); // helloworld
        System.out.println(s2); // hello

        System.out.println(s1 == s2); // false
        System.out.println(s2 == s3); // false
        System.out.println(s1 == s4); // false

    }
}

```

만약, String이 mutable, 즉 변경가능한 객체였다면,  
`s1 += "world";` 라는 덧셈을 하고 나면,  
s2 가 s1 의 값을 가리키고 있으므로,  
s2 또한 "helloworld" 가 되었을 것이다.

그러나, 실제로는 기존의 s1이 가리키던 객체를 **놔버리고**,  
world가 추가된 "helloworld" 객체를 새로 만들어서, 그곳을 가리키게 된다.

그렇기 때문에, 두 번째 `s1 == s2` 에서 false를 return 했다.  
또한, s3는 new String으로 String pool 이 아닌 heap 영역에 할당되었기 때문에, 예상한대로 s1과 같지 않았다.  
마지막으로, world가 추가된 s1은 기존의 String pool에 있던 s4와 다르므로,  

**덧셈 연산자가 더해진 s1은 heap 영역(not String pool) 에 객체로서 저장된다는 사실을 알 수 있다.**  

**바꿔 말하면, String은 Immutable 객체이기 때문에, 기존 객체의 값이 변경되지 않고**  
**새로 만들어서 할당된다는 것을 알 수 있다.**


---

> Note. 찾다보니, int 같은 primitive type도 immutable인지에 대해서 찾아보았다.  
> 잠깐 C에서의 예시를 가져와보면, C에서의 primitive data는 mutable 이다.  
> ```c++
> #include <stdio.h>
> int main() {
>    int i = 1;
>    printf("%p\n", &i); // 0x7ffd1111f95c
>
>    i += 1;
>    printf("%p\n", &i); // 0x7ffd1111f95c
> }
>```
> 그런데, Java에서의 primitive는 immutable 이라고 한다.  
> 자세한 내용은 [여기](https://stackoverflow.com/questions/18037082/are-java-primitives-immutable/18037544)를 참고하자.  
> 누군가는 "Primitive는 mutable, immutable 자체를 따질 수 없다"고도 한다.  
> 이 내용은 조금 더 찾아봐야 할 듯 싶다.

---


## Heap ↔ String pool
: 이렇게 보면 String pool은 무슨 신성불가침의 영역처럼 여겨질 수도 있는데,  
다음과 같이 활용하면, 두 영역을 손쉽게 넘나들 수 있다.


```java
public class Main {

    public static void main(String[] args) {
        String s1 = "hello";
        String s2 = new String(s1);
        String s3 = s2.intern();

        System.out.println(s1 == s2); // false
        System.out.println(s2 == s3); // false
        System.out.println(s1 == s3); // true

    }
}
```
s1 은 평소처럼 String pool에 등록해서 사용하는 것이다.  
s2 또한, new String으로 만들어주는 방법으로, heap 영역에 할당할 수 있다.  
s3 에서 나타나는 intern()의 역할은,   
**heap 영역에 있는 String을 String pool로 등록시켜준다.**

이는 필요에 따라, 적절히 사용하면 된다.

# String, StringBuilder, StringBuffer
## History

: 이제 String이 대충 어떻게 노는 녀석인지는 배웠다.  
근데, StringBuilder, StringBuffer는 왜 생겼을까?  

Before JDK 1.5, String에 덧셈을 한다는 것은 굉장히 비효율적인 일이었다.  
생각해보자. 앞서서, "hello"에다가 "world"를 더해주려면,   
"hello"를 놔버리고, 새로 만든 "helloworld"를 가리키도록 하는 일이었다.

근데 만약, for loop 안에서, 100번을 반복하는 일이 생긴다면 어떻게 될까?  
100번을 놓고, 다시 100번을 가리키도록 하는 **불상사**가 생길 것이다.  

따라서, 그 전까지는 **StringBuffer**가 String의 manipulation을 위한 유일한 선택지였다.   
그런데, StringBuffer는 **모든 Public method가 동기화 되어야 한다는 단점**을 안고 있었다.   
즉, multi-thread가 아닐 때에도, 울며겨자먹기로 StringBuffer를 쓸 수 밖에 없었다.

효율적인 String의 연산을 위해서, **StringBuiler**가 JDK 1.5부터 등장하였다.  

이 StringBuilder와 StringBuffer는 Mutable이다.   
역시, 다음의 코드를 보자.

```java
public class Main {

    public static void main(String[] args) {
        StringBuilder sb1 = new StringBuilder();
        StringBuilder sb2 = sb1;

        sb1.append("hello");
        System.out.println(sb1); // hello
        System.out.println(sb2); // hello

        System.out.println(sb1 == sb2); // true
    }
}

```

String이었다면 서로 다른 객체라고 false를 던졌을텐데,  
StringBuilder는 mutable이기 때문에 서로 같다는 결과가 나타나는 것을 볼 수 있다.  

그렇다면, String 연산 시에는 String을 무조건 쓰면 안되는 것일까? 그렇지는 않다.  
JDK 1.5 이후로, String의 + 연산도 StringBuilder를 사용하도록 변경되었다.

## Comparing 
: 백문이 불여일견. 정말 StringBuilder를 사용하는지 직접 코드로 속도 비교 테스트를 해보자.

```java
public class Main {

    public static void main(String[] args) {
        int repeat = 100000;
        long resultString = testString(repeat);
        long resultStringBuilder = testStringBuilder(repeat);
        long resultStringBuffer = testStringBuffer(repeat);

        System.out.println("string : " + resultString);
        System.out.println("stringBuilder : " + resultStringBuilder);
        System.out.println("stringBuffer : " + resultStringBuffer);

    }
    
    public static long testString(int repeat) {
        long start = System.currentTimeMillis();
        String str = "";
        for (int i=0; i<repeat; i++) {
            str += "string";
        }
        long end = System.currentTimeMillis();
        System.out.println(str.length());
        return end-start;
    }
    public static long testStringBuilder(int repeat) {
        long start = System.currentTimeMillis();
        StringBuilder sbd = new StringBuilder();
        for (int i=0; i<repeat; i++) {
            sbd.append("string");
        }
        long end = System.currentTimeMillis();
        System.out.println(sbd.length());
        return end-start;
    }

    public static long testStringBuffer(int repeat) {
        long start = System.currentTimeMillis();
        StringBuffer sbf = new StringBuffer();
        for (int i=0; i<repeat; i++) {
            sbf.append("string");
        }
        long end = System.currentTimeMillis();
        System.out.println(sbf.length());
        return end-start;
    }
}
```

약 10만번의 반복을 테스트해봤는데, 결과는 다음과 같다.

```java
string : 10886
stringBuilder : 6
stringBuffer : 8
```

???. 뭔가 이상하다.   
검색해보니 [StackOverflow](https://stackoverflow.com/questions/14927630/java-string-concat-vs-stringbuilder-optimised-so-what-should-i-do)에 다음과 같은 답변이 있었다.

```java
String one = "abc";
String two = "xyz";
String three = one + two;
// 이렇게 쓰면, 


String three = new StringBuilder().append(one).append(two).toString();
// 이렇게 컴파일 됩니다. 그러나,


String out = "";
for( int i = 0; i < 10000 ; i++ ) {
    out = out + i;
}
return out;
// 이렇게 작성하면,


String out = "";
for( int i = 0; i < 10000; i++ ) {
    out = new StringBuilder().append(out).append(i).toString();
}
return out;
// 이렇게 쓰는거랑 똑같습니다. 그러므로, 그냥 StringBuilder를 쓰세요.


StringBuilder out = new StringBuilder();
for( int i = 0 ; i < 10000; i++ ) {
    out.append(i);
}
return out.toString();
// 이렇게
```

결국 StringBuilder를 계속해서 만드는 꼴이므로,  
꼼수 부리지 말고 그냥 StringBuilder를 쓰자.

대충 정리하면, String, StringBuilder, StringBuffer의 차이점은 다음과 같다.

||String|StringBuilder|StringBuffer|
|:--:|:--:|:--:|:--:|
|Storage|String Pool|Heap|Heap|
|Modifiable|No(immutable)|Yes(mutable)|Yes(mutable)|
|Thread safe|Yes(immutable이므로)|No|Yes|
|Performance|Fast(compiled as StringBuilder)|Fast|(Relatively)Slow|

즉, 
- String : 연산이 많지 않은 경우, thread-safe 를 원하는 경우.
- StringBuilder : 연산이 많은 경우, thread 는 상관 없는 경우.
- StringBuffer : 연산이 많은 경우, thread-safe 를 원하는 경우.

가 되겠다.

### Reference
- [Guide to Java String Constant Pool](https://www.javaguides.net/2018/07/guide-to-java-string-constant-pool.html)
- [java String 풀(Pool)](http://egloos.zum.com/iilii/v/4427484)
- [String, StringBuffer, StringBuilder의 차이점과 장단점은 뭔가요?](https://www.slipp.net/questions/271)
- [Java: String concat vs StringBuilder - optimised, so what should I do?](https://stackoverflow.com/questions/14927630/java-string-concat-vs-stringbuilder-optimised-so-what-should-i-do)