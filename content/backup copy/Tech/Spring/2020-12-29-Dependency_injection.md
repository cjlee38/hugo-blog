---
# author: cjlee
categories: spring
# cover: /assets/covers/spring.png
date: "2020-12-29T09:58:00Z"
tags: ['null']
title: '# [Spring 학습 정리] Dependency Injection 의 필요성'
---


# 0. Dependency Injection 이란?
: Dependency Injection(이하, DI)는 Spring에서만 사용되는 용어가 아니라,  
객체지향프로그래밍(이하, OOP)에서 통용되는 개념이다.
이를 이해하기 위해서는, OOP에서 중요시 여기는 SOLID의 5원칙에 대해서 알아야 한다.

이번 포스팅에서는 SOLID 5원칙에 무엇이 있는지만 심플하게 짚고 넘어가자.   
자세한 내용은 [이전의 포스팅](https://cjlee38.github.io/btb/solid_principles) 혹은 다른 참고자료를 이용해 공부하자.

||||
|:--:|:--:|:--:|
|SRP|Single Responsibility Principle|클래스는 하나의 책임만을 가져야 한다.
|OCP|Open-Closed Principle|클래스를 수정하지 않으면서, 행동을 확장할 수 있어야 한다.
|LSP|Liskov Substitution Principle|상위 클래스는 하위 클래스로 대체할 수 있어야 한다.
|ISP|Interface Segregation Principle|클라이언트별로 세분화된 인터페이스를 만들어라.
|DIP|Dependeny Inversion Principle|구체 클래스가 아닌 추상 클래스에 의존해라.

---

그런데 아직 공부가 부족해서 그런지, 의존성이 뭐냐? 라고 물어보면 한마디로 "이거다" 라고 짚어내기는 어려운 것 같다. 역시 예시를 보자. 

```java
class Computer {
    private CableMouse mouse;

    public Computer() {
        this.mouse = new CableMouse();
    }

    public void moveMouse() {
        mouse.move();
    }
}
```

Computer 클래스에서 CableMouse 라는 객체를 가지고 있다. 뭐 컴퓨터가 마우스를 갖고 있는거야 당연해보이지만, 당연하지 않다. 왜일까? **Computer 객체가 생성 될 때 반드시 CableMouse가 생기기 때문이다.** 이게 정상적인 상황일까? 절대 그렇지 않다. 컴퓨터는, 조금 불편하겠지만 마우스가 없더라도 컴퓨터로서의 기능을 여전히 할 수 있기 때문이다. 

이를 코드 관점으로 바라보면, **Computer 객체가 CableMouse에 의존하고 있다.** 라고 표현한다. 혹은, Computer 와 CableMouse 사이엔 **강한 결합**이 존재한다고 말할 수 있다.



# 1. Dependency Injection 이 필요한 이유?
: 그렇다면 DI 를 이용해서, SOLID 원칙을 지킬 수 있다는 뜻일까? 대답은 **그렇다.** 이긴 한데, 어떻게 그게 가능한 것인지, 위 Computer 예시를 조금씩 수정해나가면서 알아가보자.
## Example.1
: 우선 위 예제를 그대로 갖고 오자. 그리고, CableMouse와 WirelessMouse 또한 만들어보자.


```java
class Computer {
    private CableMouse mouse;

    public Computer() {
        this.mouse = new CableMouse();
    }

    public void moveMouse() {
        mouse.move();
    }
}

class CableMouse {
    public void move() {
        System.out.println("CableMouse moved");
    }
}

class WirelessMouse {
    public void move() {
        System.out.println("WirelessMouse moved");
    }
}
```
아까 위 예시에서 봤듯이, 현재 Computer 클래스는 CableMouse에 대한 의존성을 갖고 있다. 여기서, 컴퓨터가 사용하는 유선마우스를 무선마우스로 바꾼다면 어떻게 해야할까?

```java
class Computer {
    WirelessMouse mouse; // 변경된 부분

    public Computer() {
        this.mouse = new WirelessMouse(); // 변경된 부분
    }

    public void moveMouse() {
        mouse.move();
    }
}
```

위와 같이, CableMouse를 쓰는 부분을 모두 WirelessMouse로 고쳐야 한다. SOLID 원칙에 입각해서 보면, 다음과 같다.

|SOLID|Compliance|Reason|
|:--:|:--:|:--:|
|Single Responsibility Principle|X|Computer 클래스는, 하나의 역할만 하고 있지 않다. 마우스 객체를 생성하고, 사용하는 역할까지 하고 있다.|
|Open Closed Principle|X|유선 마우스를 무선 마우스로 변경하기 위해서, Computer 클래스를 수정해야 했다.|
|Liskov Substitution Principle|X|아직 유선 마우스와 무선 마우스는 서로 독립적인 클래스이므로, 아직 super/sub의 개념이 없다.|
|Interface Segregation Principle|X|마찬가지로 아직 Interface가 존재하지 않는다.
|Dependency Inversion Principle|X|"무선"마우스, "유선" 마우스 라는 구체적인 클래스에 의존하고 있다.|

아무래도 보아하니, 우선 interface 를 만드는 것이 급선무인 것 처럼 보인다. 우선 Mouse interface를 만들어보자. 그리고, CableMouse와 WirelessMouse가 이를 구현할 수 있도록 해보자.

## Example.2

```java
interface Mouse {
    void move();
}

class CableMouse implements Mouse {
    @Override
    public void move() {
        System.out.println("CableMouse moved");
    }
}

class WirelessMouse implements Mouse {
    @Override
    public void move() {
        System.out.println("WirelessMouse moved");
    }
}
```

마지막으로, Mouse 객체의 변경이 용이하도록(즉, 다형성을 이용하도록), Data type을 CableMouse인 구체 클래스가 아닌, Mouse 라는 interface를 사용하도록 하자.

```java
class Computer {
    private Mouse mouse;

    public Computer() {
        this.mouse = new CableMouse(); // 여기는..?
    }

    public void moveMouse() {
        mouse.move();
    }
}
```

그런데 이렇게 interface 를 만든다고 해서 모두 해결된 것이 아니다. 여전히, 생성자에서 `new CableMouse()` 라는 구체 클래스를 사용하고 있고, WirelessMouse로 바꾸기 위해서는 코드를 수정해야 하기 때문이다. 

SOLID 5원칙을 기준으로 다시 하나씩 점검해보자.

|SOLID|Compliance|Reason|
|:--:|:--:|:--:|
|Single Responsibility Principle|X|여전히, 마우스 객체를 클래스에서 생성하고 있다.|
|Open Closed Principle|X|여전히, 유선 마우스를 무선 마우스로 변경하기 위해서, Computer 클래스가 수정되어야 한다.|
|Liskov Substitution Principle|O|마우스 interface가 기대한 대로 유/무선 마우스 클래스가 동작하고 있으므로, LSP를 지키고 있다.|
|Interface Segregation Principle|O|인터페이스가 작은(응집된) 역할만 하고 있으므로, ISP를 준수하고 있다.
|Dependency Inversion Principle|X|다형성을 이용했음에도 불구하고, `CableMouse` 라는 여전히 구체 클래스에 의존하고 있다.|


결국 문제가 되는 부분은, Computer 클래스에서 Mouse를 직접 만든다는 것이다. Mouse 객체를 **외부에서 받을 수 있도록**, 즉 의존성을 주입받을 수 있도록 수정해보자.

## Example.3

```java
class Computer {
    private Mouse mouse;

    public Computer(Mouse mouse) {
        this.mouse = mouse;
    }

    public void moveMouse() {
        mouse.move();
    }
}
```

|SOLID|Compliance|Reason|
|:--:|:--:|:--:|
|Single Responsibility Principle|O|Computer 클래스에서 마우스를 직접 생성하지 않고, 외부에서 입력받는다.|
|Open Closed Principle|O|유선 마우스에서 무선 마우스로 바꾸더라도, Computer 클래스는 변하지 않는다.|
|Liskov Substitution Principle|O|마우스 interface가 기대한 대로 유/무선 마우스 클래스가 동작하고 있으므로, LSP를 지키고 있다.|
|Interface Segregation Principle|O|인터페이스가 작은(응집된) 역할만 하고 있으므로, ISP를 준수하고 있다.
|Dependency Inversion Principle|O|Computer 클래스는 CableMouse, WirelessMouse 라는 구체 클래스에 전혀 의존하지 않는다.|

드디어 SOLID 원칙을 모두 준수하는(우리가 당연스럽게 작성해왔던) 설계의 코드를 만들 수 있었다. 이제 행복하게 사용하기만 하면 될...까? Computer 객체를 사용하는 Person 이라는 객체가 있으면 어떨까?

```java
class Person {
    public static void main(String[] args) {
        Mouse mouse = new CableMouse(); // 여기
        Computer computer = new Computer(mouse);
        computer.moveMouse();
    }
}
```

역시 또 CableMouse를 WirelessMouse로 바꾸기 위해 주석으로 달아놓은 코드를 수정해야 하는 문제가 생긴다. 결국 이렇게 보면 반쪽짜리 SOLID 가 되는 것이다. 그런데 여기까지 오면, 이런 의문이 생긴다.

<br>
<center> CableMouse 와 WirelessMouse 라는 구체 클래스를 결국 만들기는 해야되잖아? </center>
<br>

맞는 말이다. 결국 어딘가에서는 내가 사용하고자하는 마우스가 유선인지, 무선인지에 대한 지정은 해줘야한다. 그러나 여기서 잠깐 짚고 넘어갈 것이 있다.

### 1) SOLID 원칙을 준수함으로써 얻는 이득

우리가 SOLID 원칙을 지킴으로써 얻고자 하는것은 한 쪽의 코드를 변경했을 때, 다른 쪽의 코드를 변경하는 상황을 막는 것. 즉, 약한 결합력(light coupling) 이다. 이를 통해, 코드 설계를 이해하기에, 유지보수하기에, 확장하기에 쉽도록 만들 수 있다. 우리가 Computer 클래스를 사용하는 Person 클래스의 코드를 변경하더라도, Computer 클래스의 코드는 전혀 변화하지 않는다.

### 2) 역할과 구현
: 가령, 어떤 흥행하는 뮤지컬이 있다고 해보자. 주연 A의 역할을 배우 a가 맡았고, 주연 B의 역할을 배우 b가 맡았다. 그런데 b가 갑자기 아파서, c 라는 사람이 b의 역할을 대신해야 한다. 이게 문제가 될까? 문제가 되지 않는다. 왜냐하면, B 라는 역할을 수행하던 b가 하던대로(즉, **각본대로**), c 또한 잘 해내면 되기 때문이다.

위 예시로 바꿔서 생각해보자. Computer는 뮤지컬이고, Mouse는 주연이다. Mouse 의 역할을 기존 CableMouse 라는 배우가 담당했는데, 사정이 생겨서 WirelessMouse 가 대체해야 한다. 문제가 될까? 그렇지 않다. 똑같이 마우스를 움직이고, 클릭하는 기능만 제대로 수행할 수 있으면 되기 때문이다. 잘 생각해보면, 결국 1번과 비슷한 이야기다.

즉, 위 두 이야기를 고려했을 때, Computer 클래스에는 전혀 문제가 없다는 이야기다. 그렇다면 문제가 됐던 Person 클래스의 코드를 수정해야 하는데, 관건은 **도대체 어디서 실제 객체를 생성할 것이냐?** 가 문제가 되는 것이다. 

뮤지컬 주연은 배우를 선택하지 않는다. 배우를 선택하는 건 뮤지컬 감독이 한다. 마찬가지다. Mouse 라는 주연, 즉 인터페이스는 CableMouse 라는 배우, 즉 객체를 선택하지 않는다. **우리는 뮤지컬 감독의 역할이 필요하다.** 즉, 감독의 역할을 맡는  config 설정파일을 생성할 수도 있고, 혹은 Config 라는 클래스를 만들어서 사용할 수도 있다. 그리고 이렇게 Config 에서 실 객체를 넣어주는 작업을 바로 Dependency Injection이라 한다.

```java
class Config {
    public static Mouse getMouse() {
        return new CableMouse();
    }
}

class Person {
    public static void main(String[] args) {
        Mouse mouse = Config.getMouse();
        Computer computer = new Computer(mouse);
        computer.moveMouse();
    }
}
```

이렇게 하면,  유선 마우스를 무선 마우스로 바꾼다면, Config 라는 클래스의 `new CableMouse()` 를 `new WirelessMouse()` 로 수정하기만 하면 된다.



# 2. 아직 끝이 아니다.
: 그런데, 마우스와 같은 간단한 경우가 아니라, 사용자가 많은 주문 서비스라면 어떨까?  가령, 사용자가 초당 100 명이라고 한다면, 매 초마다 100개의 객체가 생성되었다가, 삭제되었다가를 반복할 것이다.

객체들이 **특정한 상태**가 요구되지 않는다면, 싱글톤 패턴을 이용하는 것이 해결책이 될 수 있다.

```java
class Computer {
    private static final Computer instance = new Computer();

    private Computer() {}
    
    public static Computer getInstance() {
        return instance;
    }
}
```

---

> Note. 위 Computer 클래스는 간단하게 싱글톤 패턴의 예시를 보여줄 뿐, 위에서 했던 내용과는 연관이 없다.

---

싱글톤 패턴은, 클래스의 instance가 1개만 생성되는 것을 보장하는 디자인 패턴을 의미한다. 따라서, instance가 여러 개 생성되는 것을 막기 위해서, private constructor를 사용했다.  
그리고 이 instance를 얻기 위해서는, 반드시 미리 생성된 `getInstance()` 메소드를 호출해야 한다.

그러나, 싱글톤 패턴을 이용할 경우, 다음과 같은 문제점들이 있다.

- 싱글톤 패턴을 구현하는 코드 자체가 많아진다.(e.g. multi-thread 환경의 동기화)
- 구체 클래스의 instance를 가져오는 메소드를 호출하므로, 구체 클래스에 의존하게 된다.  
  따라서, DIP를 위반하게 된다.
- 테스트가 어렵다.
- private 생성자를 사용하므로, 자식 클래스를 만들기 어렵다.
- 즉, 유연성이 떨어진다.

이렇게 되면 골치가 아프다. 우리는 Singleton 패턴의 단점을 최소화하면서, 동시에 Dependency injection 을 사용하고 싶다. 그리고 여기서, 드디어 **Spring이 등장한다.**

Spring 은 이 두마리 토끼를 다 잡을 수 있도록 지원해준다. 그것이 어떻게 가능한 것인지, 그 동작 방식은 무엇인지에 대해 다루기는 양이 꽤 되므로 다음 포스팅에서 다루기로 하고, 마틴 파울러의 저서, '리팩토링' 에서 남긴 말로 포스팅을 마무리한다.


> 컴퓨터가 이해할 수 있는 코드는 어느 바보나 다 짤 수 있다.  
좋은 프로그래머는 사람이 이해할 수 있는 코드를 짠다. - Martin Fowler


## Reference
- [Spring프레임워크를 사용하여 DI(Dependency Injection) 해보기](https://taesan94.tistory.com/87)
- 인프런, 김영한 님 스프링 강좌
- 유튜브, 뉴렉처 님 스프링 강좌
