---
# author: cjlee
categories: java
# cover: /assets/covers/coding.png
date: "2020-10-23T17:48:00Z"
tags: ['null']
title: '# (Java) Junit은 어떻게 Main method 없이 실행될까? '
---

# 0. 들어가며
: 첫 처음 자바를 배웠을 당시가 떠오른다.  
아무것도 모른채, "약속"이라는 말과 함께   
`public static void main(String][] args)` 를 기계적으로 입력했었는데,  
이게 무슨 뜻인지, 왜 이렇게 썼는지에 대해서 그닥 신경을 안썼던 것 같다.

그런데, 최근 스프링을 공부하면서, 난생 처음으로 내손으로 테스트를 작성해보다,  
main method가 없는데 어떻게 실행되지? 라는 의문에 봉착했다.  
생각의 꼬리를 물고 올라가다 보니, main method 자체에 대한 의문이 생겼다.  

파이썬을 생각해보면, 이러한 main method 하나 없이도,  
`print("hello world")` 라는 명령어 하나만으로 실행이 가능했다.  
그런데 Java는 다르다. 그 근본을 하나씩 파헤쳐보자.

# 1. public static void main(Strign[] args)
### main
: 가장 먼저 의문이 들었던 부분은, **"메소드 이름이 정말 main이어야 하는가?"** 이다.

백문이 불여일견. main 의 이름을 init으로 바꿔보고, 실행해보았다.

**코드**
```java
public class Main {

    public static void init(String[] args) {
        System.out.println("hello world");
    }

}
```

**실행결과**
```bash
java Main.java
error: can't find main(String[]) method in class: com.company.Main
```

보다시피, 반드시 이름은 main 이어야 하는 것 같다.  
그렇다면, 각각의 keyword가 필요한 이유는 무엇일까?

### public
: public은 알다시피, 접근 제어자다.  
public으로 선언하게 되면, **외부로 공개**되어 이 메소드는 다른 클래스에서 사용될 수 있다.  
따라서 main method가 public으로 선언되어야 하는 이유는 당연해보이지만,  
그 좀만 더 자세하게 살펴보자.

Java 프로그램이 실행되는 절차는 다음과 같다.  
1. Java classloader가 Class들을 JVM으로 load한다.*(≒ 등록한다.)*
2. class 내에 작성된 static block을 실행한다.
3. main method를 찾은 뒤, 이를 invoke 한다.

---

> Note. 위 흐름과 같이, 우리가 작성한 코드가 그 자체로써 동작하는 것이 아니라, JRE 라는 환경 안에 편입(?) 되므로, 파이썬 처럼 곧바로 메소드를 수행할 수 없는 것이다.

---

테스트를 위해, 다음과 같은 코드를 작성해서 실행해보자.  

**코드**
```java
public class Main {

    static {
        System.out.println("hello world");
    }

}
```

**실행결과**
```bash
java Main.java
hello world
error: can't find main(String[]) method in class: com.company.Main
```

보는 바와 같이, static block을 실행했으나,  
main method를 찾지 못해서 에러가 발생한다.  

--- 

> Note. 위와 같은 에러를 발생시키지 않으려면, `System.exit(0);` 를 통해  
> main method() invokation을 일으키지 않고 프로그램을 종료할 수 있다.

---

다음으로, static block을 실행 한 **이후에**, main method가 실행 되는지 확인해보자.

**코드**
```java
public class Main {
    static int value = 10;
    static {
        System.out.println("1 : " + value);
        value++;
        System.out.println("2 : " + value);
    }

    public static void main(String[] args) {
        System.out.println("3 : " + value);
        value++;
        System.out.println("4 : " + value);
    }

}
```

**실행결과**
```bash
1 : 10
2 : 11
3 : 11
4 : 12
```

예상한 순서대로 프로그램이 진행된 것을 확인할 수 있다.  

즉 정리하자면, 이 JVM이 위와 같은 흐름을 진행하면서  
main method를 호출하는데, JVM은 `Main` 이라는 class가 아니므로,  
public으로 선언해줘야 한다.

### static
: static 또한 마찬가지다.  
`Main` 이라는 클래스의 instance를 만들지 않고 호출할 수 있도록, static keyword 를 붙여야 한다.  
다시 말해, non-static 이라면, main method를 호출하기 위해서 instance를 만들어야 한다.

또한, main method는 어찌 되었든, 프로그램의 **가장 첫 스택**에 쌓이는 메소드다.  
말인 즉슨, 프로그램의 시작과 끝(Stack이므로)을 함께하는 메소드이고,  
따라서 **프로그램이 종료될 때 까지 유지되어야 하기 때문에**, static으로 선언된다.

### void
: void는 딱히 특별할 것이 없는 것 같다.   
void가 아니라면, 우리가 프로그램을 종료할 때에, 어떤 값을 반환하겠다는 의미인데,  
이 값을 받는 JVM이 무슨 역할을 할까?

Java의 개발자가 아닌 사용자는, 이에 대해서 쉽게 답변할 수 없을 것 같다.

--- 

> Note. 사실 의문점이 남는 부분이 하나 있는데, 가장 처음 배웠던 C언어를 생각해보면, main() 함수를 void main() 이 아닌 int main()으로 사용했고, void main()을 사용하더라도 컴파일러에 의해 int main()으로 변환된다는 이야기를 들었다. 그리고, 이 때 return 되는 값은 기본적으로 0 인데, 이는 "정상적으로 종료되었다" 를 의미한다. 그리고 이 결과값을 받는 것은 Shell 프로그램이고, 이 Shell 이 받는 결과값에 따라서 또 행동이 달라질 수 있다.  
> 그러나, Java에서는 그저 void로, 아무것도 return하지 않는다. 즉, 결과에 관계 없이, JVM은 언제나 같은 동작만을 수행한다고 예측할 수 있다. 이러한 차이에 대해서, Java 개발자인 제임스 고슬링의 철학을 알 수 있는 도리가 없으니, 아쉽지만 여기서 멈춰야한다.

---

### String[] args
: args는 arguments의 약자로, 프로그램 외부에서 전달하는 인자를 받아내는 역할을 한다.  
역시, 코드로 살펴보자.

**코드**
```java
public class Main {
    public static void main(String[] args) {
        for (String arg : args) {
            System.out.println(arg);
        }
    }

}
```

**실행결과**
```bash
java Main.java hello world
hello
world
```

두 개의 숫자를 사칙연산을 하는 계산기 프로그램을 만들었다고 생각해보자.  
그렇다면, 숫자를 어디서 구할 것인가?  
코드 안에서 input을 받을 수도 있지만, 위와 같은 방식으로 곧바로 넣어줄 수도 있는 것이다.

# 2. Where is main method in Junit test?
: Junit의 main method 를 이야기 하는데,  
위와 같이 main method의 근본에 대한 잡설을 한 이유는,  
우리가 작성한 Java program이 어떻게 동작하는가? 에 대해서 이해하기 위함이었다.

즉, `3. main method를 찾은 뒤, 이를 invoke 한다.` 라는 것이 핵심이다.  
그렇다면, main method가 없다면 프로그램이 실행되지 않을까?  
*static block을 사용하지 않는다면,* 그렇다.

결국, main method는 있어야 한다.  
다시 돌아와서, Intellij와 같은 개발환경에서 테스트 코드를 보면,  
**main method는 전혀 존재하지 않는다.** 

그런데도, 옆에 ▶ 의 run 버튼이 생긴다. 어떻게..?  

Main 클래스와 테스트 클래스를 각각 돌려서,  
터미널을 확인해보면 그 이유를 알 수 있다. 

**Main**
```bash
"C:\Program Files\Java\jdk1.8.0_261\bin\java.exe" "-javaagent:C:\Program Files\JetBrains\IntelliJ IDEA 2020.2.1\lib\idea_rt.jar=7495:C:\Program Files\JetBrains\IntelliJ IDEA 2020.2.1\bin" -Dfile.encoding=UTF-8 -classpath "어어엄청나게 많은 라이브러리들" com.company.Main
```

**Test**
```bash
"C:\Program Files\Java\jdk-11.0.8\bin\java.exe" -ea -Didea.test.cyclic.buffer.size=1048576 "-javaagent:C:\Program Files\JetBrains\IntelliJ IDEA 2020.2.1\lib\idea_rt.jar=2288:C:\Program Files\JetBrains\IntelliJ IDEA 2020.2.1\bin" -Dfile.encoding=UTF-8 -classpath "어어엄청나게 많은 라이브러리들" com.intellij.rt.junit.JUnitStarter -ideVersion5 -junit5 com.company.myTest
```

*(테스트를 사용하는 프로젝트와, Java 연습용으로 사용하는 프로젝트의 JDK 버전을 다르게 설정해놓았는데, 여기서는 특별한 의미는 없다.)*

우리가 cmd 창을 열어서, `java Main.java`라는 명령어를 입력하듯이,  
IDE의 run 버튼은 위와 같은 정말 **길고도 긴 명령어**를 한방에 딱 입력해준다.

이 중에서 주목해야할 부분은, `com.intellij.rt.junit.JUnitStarter` 라는 부분이다.  
저녀석과 JDK 버전, 그리고 실행하는 java 파일 이름을 제외하면 **거의 비슷하다는 것**을 확인할 수 있다.

즉, 우리가 테스트를 실행하면,   
본질적으로는 `com.intellij.rt.junit.JUnitStarter` 이 녀석이 실행되는 것이다.

`C:\Program Files\JetBrains\IntelliJ IDEA 2020.2.1\plugins\junit\lib\junit-rt.jar`   
를 까보면, (경로는 사용자마다 다를 수 있다.)  
`com.intellij.rt.junit.JUnitStarter.class` 가 있는 것을 확인할 수 있다.

엥? 위에서 테스를 실행했던 명령어에서,   
`com.intellij.rt.junit.JUnitStarter` 랑 같은 위치다.  
와우, 그렇다면 우리가 이 녀석을 실행한다는 것을 알 수 있다.  
Intellij를 이용해서, 디컴파일 해보면

```java
public static void main(String[] args) {
    List<String> argList = new ArrayList(Arrays.asList(args));
    ArrayList<String> listeners = new ArrayList();
    String[] name = new String[1];
    String agentName = processParameters(argList, listeners, name);
    if (!"com.intellij.junit5.JUnit5IdeaTestRunner".equals(agentName) && !canWorkWithJUnitVersion(System.err, agentName)) {
        System.exit(-3);
    }

    if (!checkVersion(args, System.err)) {
        System.exit(-3);
    }

    String[] array = (String[])argList.toArray(new String[0]);
    int exitCode = prepareStreamsAndStart(array, agentName, listeners, name[0]);
    System.exit(exitCode);
}
```
드디어 메인 메소드를 찾았다.  
그리고, 이 뒤에 붙은 `-ideVersion5 -junit5 com.company.myTest` 녀석들을 인자로 넘겨주면서,  
적절한 프로세싱과, 테스트의 실행이 동작한다.

# 3. 마치며
Junit이 동작하는 방식에 대해서도 코드로 살짝 훑어봤는데,  
어렵기도 하고, 여기에 또 쓰기에는 양이 정말 많아질 것 같아서, 이쯤 해야 할 것 같다.

이상으로 포스팅을 마칩니다. 틀린 부분이 있으면 지적 바랍니다.


## Reference
- [Stackoverflow - Can we execute a program without main() method?](https://stackoverflow.com/questions/26881076/can-we-execute-a-program-without-main-method)
- [main 함수의 반환 값 활용하기](https://blog.naver.com/tipsware/221255071329)
- [Stackoverflow - java junit has no main function](https://stackoverflow.com/questions/14011981/java-junit-has-no-main-function)
- [Stackoverflow - Why is main() in java void?](https://stackoverflow.com/questions/540396/why-is-main-in-java-void)
- [wikipedia - 자바 클래스로더](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%B0%94_%ED%81%B4%EB%9E%98%EC%8A%A4%EB%A1%9C%EB%8D%94)