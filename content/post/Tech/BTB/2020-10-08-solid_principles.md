---
# author: cjlee
categories: btb
# cover: /assets/covers/coding.png
date: "2020-10-08T12:35:00Z"
tags: ['null']
title: '# 객체지향 설계원칙 - SOLID(번역)'
---

SOLID 원칙에 대해서 공부하던 중, 적절한 예시와 함께 작성된 글이 있어,   
짧은 영어 실력으로 번역 및 정리해보았다.  

수능을 마치고나서, 영어를 볼 떄마다 종종 드는 생각은,  
영어가 머릿속에 꽂히는, 맥락의 파악은 쉬워지는데,  
이를 한글로 다시 뱉어내기가 어려워졌다는 점 인것 같다.

사실상 구글번역기를 돌린 것과 큰 차이가 없으니, 읽다가 정 안읽히면  
차라리 영어 원문을 보는것을 추천한다.

[원문](https://www.jrebel.com/blog/solid-principles-in-java)

# SOLID 원칙이란 무엇입니까?

> Robert C.Martin(Uncle Bob)에 의해 만들어진 SOLID 설계 원칙은 5가지 원칙의 조합을 의미합니다. 

- SRP : Single Responsibility Principle. 단일 책임 원칙
- OCP : Open-Closed Principle. 개방 폐쇄 원칙
- LSP : Liskov Substitution Principle. 리스코프 치환(교체) 원칙
- ISP : Interface Segregation Principle. 인터페이스 분리 원칙
- DIP : Dependeny Inversion Principle. 의존관계 역전 원칙

SOLID 원칙은 다양한 테스트 제품군과 함께, 코드가 부패(...) 하는것을 방지하는데 도움을 주는 클래스 레벨의 객체지향 설계 컨셉입니다. 자바에서, SOLID 원칙은 소프트웨어의 주요 가치를 높게 유지하는데 도움을 줍니다.

# 왜 SOLID 설계 원칙을 사용해야 합니까?
![](/assets/images/2020-10-21-12-57-44_2020-10-08-solid_principles.md.png)

개발자들이 SOLID와 같은 구조화된 설계 원칙 없이 설계를 하면, 그들은 이 프로젝트를 진행할 미래의 개발자들에게 오랫동안 지속될 문제를 만들 수 있고, 그들이 개발하는 어플리케이션의 잠재적인 성공을 제한할 수 있습니다. 이러한 이슈는 흔히 **"코드 부패"** 라고 부릅니다.

---

<center> 
<b>Code rot(코드 부패)</b>
<br>
어플리케이션이 점점 개발자들이 유지보수하기 어려워지는 짜증나는 코드 덩어리가 되는 것.
</center>

---

그렇다면, 어떻게 미래의 코드 부패를 알아낼 수 있을까요? 아래와 같은 신호(징후)는 코드가 부패한다는 것을 암시합니다.

- **Rigidity(경직성)** : 작은 변화가 전체 시스템의 rebuild를 야기하는 것
- **Fragility(취약성)** : 한 모듈의 변화가 다른 관련 없는 모듈이 오작동하도록 야기하는 것.  
  (e.g. 라디오 주파수 변경이 창문에 영향을 미치는 자동차 시스템)
- **Immobility(부동성)** : 모듈의 내부 구성요소가 새로운 환경에서 재사용 될 수 없는 것
  (e.g. 한 어플리케이션의 로그인 모듈이 다른 모듈들 간의 coupling과 의존성으로 인해 완전히 다른 시스템에서 재사용 될 수 없는 경우, 부동적이다.)
- **Viscosity(점성)** : 빌드와 테스트가 어렵고 실행하는 데 오래 걸리는 경우. 사소한 변화도 비용이 많이 들고, 여러 위치/수준 에서 변경이 필요한 경우.

사용자들은 사용하는 소프트웨어에서 어떠한 가치를 얻을 것이라고 기대합니다.   
어플리케이션의 가치는, 생산성, 시간, 돈 등을 증대시키고 낭비를 절약함으로써, 사용자들이 무언가를 하는데에 도움을 주는지의 여부에 따라 결정됩니다.  
사람들은 높은 가치를 가친 소프트웨어에 돈을 지불합니다.

하지만, 사용자들이 훌륭한 소프트웨어로부터 얻는 2차 가치가 있는데, 나는 그 가치에 대해서 먼저 이야기하려고 합니다. 왜냐하면, 사람들이 소프트웨어의 가치에 대해 이야기 할 때 이것을 가장 먼저 생각하기 때문입니다 : **Behaviour(행동)**

### 어플리케이션에서 2차 가치를 만드는 것
: 만약 소프트웨어가 버그, 충돌, 딜레이 없이 사용자가 필요로 하는 것을 수행한다면, 소프트웨어의 2차 가치는 높습니다. 2차 가치는 소프트웨어가 사용자의 현재 요구를 만족시킬 때 달성됩니다. 하지만 사용자의 변화는 자주 변합니다; 소프트웨어가 제공하는 행동(behaviour)와, 사용자가 필요로 하는 것은 불일치하기 쉽고, 이는 가치를 낮추게 됩니다. 당신의 소프트웨어는 사용자의 변화하는 요구에 따라갈 수 있어야 합니다. 그래야 우리는 지속적인 변화를 용인하고, 촉진할 수 있는 소프트웨어의 주요 가치에 도달하게 됩니다.

현재 소프트웨어가 사용자의 요구를 만족시키지만, 수정하기에 아주 어렵고 비싸다고 상상해보세요. 여기서, 앱의 inflexibility(경직성)으로 인해, 사용자들은 좋아하지 않고, 수익률도 떨어질 것입니다.

처음에는 2차 가치가 낮지만, 수정하기에 싸고 간단한 소프트웨어가 있다고 상상해보세요. 수익성은 올라가고, 사용자들은 점점 더 좋아할 것입니다.

## SOLID 5원칙에는 무엇이 있습니까?
: 위에서 얘기했듯이, SOLID 원칙은 5가지 객체지향 설계 원칙으로 구성되어 있습니다; 단일책임 원칙, 개방폐쇄 원칙, 리스코프 치환 원칙, 인터페이스 분리 원칙, 의존관계 역전 원칙. 이 원칙들은 개발자들이 "코드 부패"에서 벗어나, 고객들에게 지속적인 가치와 이 프로젝트에 참여할 미래의 개발자들에게 건전성을 제공하는 어플리케이션을 만들 수 있도록 하는 가치있는 기준을 제공해줍니다.

<center> <b>SOLID 설계 원칙</b> </center>

||||
|:--:|:--:|:--:|
SRP|Single Responsibility Principle|클래스를 변경하는 이유는 오직 하나여야 한다.
OCP|Open-Closed Principle|클래스를 수정하지 않으면서, 행동을 확장할 수 있어야 한다.
LSP|Liskov Substitution Principle|상위 클래스는 하위 클래스로 대체할 수 있어야 한다.
ISP|Interface Segregation Principle|클라이언트별로 세분화된 인터페이스를 만들어라.
DIP|Dependeny Inversion Principle|구체 클래스가 아닌 추상 클래스에 의존해라.

## 1.Single Responsibility Principle(SRP)
![](/assets/images/2020-10-21-13-31-19_2020-10-08-solid_principles.md.png)

단일 책임 원칙은 클래스가 변경되어야 하는 이유가 하나보다 많으면 안된다고 명시하고 있습니다. 즉, 코드의 모든 클래스(혹은 유사한 구조)는 하나의 작업만 수행해야 합니다.

클래스의 모든 것은 하나의 목적과 관련이 있어야 합니다.(즉, 응집되어야 합니다.) 이것은 클래스가 하나의 메소드나 속성만을 가져야 한다는 것을 의미하지는 않습니다.

단일책임에 관련되는 한, 많은 구성원이 있을 수 있습니다. 한 가지 변경의 이유가 생길 경우, 클래스의 여러 개의 멤버가 변경되어야 할 수 있기 때문입니다. 또는 많은 클래스에 업데이트가 필요할 수 있습니다.

다음의 코드는 얼마나 많은 책임을 갖고 있을까요?

```java
class Employee {
  public Pay calculatePay() {...}
  public void save() {...}
  public String describeEmployee() {...}
}
```

정답은 세 개 입니다.

하나의 클래스 안에 1) 계산하는 로직, 2) 데이터베이스 로직, 3) 보고 로직이 섞여 있습니다. 만약 당신이 여러개의 책임을 하나의 클래스로 합친다면, 다른 것들을 깨뜨리지 않으면서 하나를 바꾸는 것은 어려울 것입니다. 책임을 섞는것은 또한, 이해하기도, 테스트하기에도 어렵게 만들면서, 응집도를 낮춥니다. 이를 해결하는 가장 간단한 방법은 클래스를 세 개의 서로 다른 클래스로 나누고, 각각이 하나의 책임을 갖도록 하는 것입니다 : 데이터베이스 접근, 계산, 보고

## 2. Open-Closed Principle(OCP)

![](/assets/images/2020-10-21-13-38-48_2020-10-08-solid_principles.md.png)

개방폐쇄원칙은 클래스가 확장에는 개방되어 있으나, 변경에는 닫혀있어야 함을 말합니다."확장에 개방되어 있다"는 것은, 새로운 요구사항이 생겼을 때 새로운 기능을 쉽게 추가할 수 있도록 클래스를 작성해야 함을 의미합니다. "변경에 닫혀 있어야 한다"는 것은, 한번 만들어진 클래스는 버그를 잡는 것이 아니라면 변경해서는 안된다는 것을 의미합니다.

이 원칙의 두가지 부분은 모순되는 것 처럼 보입니다. 하지만, 올바르게 클래스와 의존성을 구조화 하면, 기존의 코드를 변경하지 않고 기능을 추가할 수 있습니다.

일반적으로 concrete(구체) class가 아닌, interface나 abstract class와 같은 추상적인 것들에 의존적이게 참조함으로써 이를 달성할 수 있습니다. 인터페이스를 구현하는 새로운 클래스를 만들어서, 기능을 추가할 수 있습니다.

OCP를 당신의 프로젝트에 적용하는 것은 소스코드가 한번 작성되고, 테스트되고, 디버깅 되었다면, 변경할 필요를 제한합니다. 이는 기존의 코드에 새로운 버그를 발생시킬 위험을 줄임으로써, 더욱 robust(강력한) 한 소프트웨어로 이어집니다.

**Open-Closed Principle Example**
: 의존성을 위해 인터페이스를 사용하는 것의 side-effect는 결합력을 낮추고, 기능성을 증가시키는 것 입니다.

```java
void checkOut(Receipt receipt) {
  Money total = Money.zero;
  for (item : items) {
    total += item.getPrice();
    receipt.addItem(item);
  }
  Payment p = acceptCash(total);
  receipt.addPayment(p);
}
```

신용카드 결제를 어떻게 추가할 수 있을까요? 아래와 같이 if-문을 추가할 수 있지만, 이는 OCP의 위반입니다.

```java
Payment p;
if (credit)
  p = acceptCredit(total);
else
  p = acceptCash(total);
receipt.addPayment(p);
```

다음의 방식이 더 낫습니다.

```java
public interface PaymentMethod {void acceptPayment(Money total);}

void checkOut(Receipt receipt, PaymentMethod pm) {
  Money total = Money.zero;
  for (item : items) {
    total += item.getPrice();
    receipt.addItem(item);
  }
  Payment p = pm.acceptPayment(total);
  receipt.addPayment(p);
}
```

그리고, 여기 더러운(?) 작은 비밀이 하나 있습니다 : OCP는 발생할 것이라고 예측 가능한 경우에만 도움을 줍니다. 따라서, 비슷한 변화가 이미 일어난 경우에만 적용해야 합니다. 따라서, 가장 간단한 것을 먼저 하고, 어떤 변화가 요구되는지 확인해야, 미래의 변화를 좀 더 정확하게 예측할 수 있습니다.

이는 고객이 변화를 만들고 나서야, (미래의 비슷한 변화로부터 당신을 보호하기 위해) 추상적인 것을 개발하는 것을 의미합니다.

## 3. Liskov Substitution Principle (LSP)

![](/assets/images/2020-10-21-13-54-05_2020-10-08-solid_principles.md.png)

LSP는 상속 계층에 적용되며, 클라이언트가 알지 못하게 클라이언트 종속성이 하위 클래스로 대체될 수 있도록 설계해야 함을 말합니다.

따라서, 모든 하위클래스는 상위클래스와 같은 방식으로 동작해야 합니다. 하위 클래스의 특정 기능은 다를 수 있지만, 상위 클래스의 기대되는 동작에 부합해야 합니다. 진정한 행동적 subtype 이 되기 위해서, 하위클래스는 상위 클래스의 메소드와 속성을 구현할 뿐만 아니라, 암시적 행동 또한 부합해야 합니다.

일반적으로, supertype의 subtype이 supertype의 클라이언트가 기대하지 않은 행동을 한다면, 이는 LSP의 위반입니다. 상위 클래스가 던지지 않는 예외를 던지는 하위클래스를 상상해보세요. 기본적으로, 하위클래스의 기능은 상위클래스보다 적어서는 안됩니다.

LSP를 위반하는 전형적인 예로는, 직사각형 클래스에서 파생되는 정사각형 클래스 입니다. 정사각형 클래스는 언제나 높이와 너비가 같습니다. 만약 직사각형이 기대되는 곳에서 정사각형 객체가 사용된다면, 정사각형의 치수는 독립적으로 변할 수 없기 때문에, 예기치 않은 동작이 발생할 수 있습니다.

**Liskov Substitution Principle Example**
: 이 문제는 쉽게 해결할 수 없습니다 : 정사각형이 불변(즉, 치수가 같도록 유지)하기 위해서 정사각형의 setter 메소드를 수정할 수 있다면, 이 method는 직사각형 setter의 치수가 독립적으로 수정될 수 있다는 직사각형의 사후 조건을 위반합니다.

```java
public class Rectangle {
  private double height;
  private double width;

  public double area();

  public void setHeight(double height);
  public void setWidth(double width);
}
```

위 코드는 LSP를 위반합니다.

```java
public class Square extends Rectangle {  
  public void setHeight(double height) {
    super.setHeight(height);
    super.setWidth(height);
  }

  public void setWidth(double width) {
    setHeight(width);
  }
}
```

LSP 위반은 정의되지 않은 행동을 야기합니다.  정의되지 않은 행동이란, 개발과정에서는 잘 동작하지만, 생산 단계에서 blows up(터지거나), 하루에 한번 일어날 수 있는 것을 디버깅 하는데 몇주를 쓰거나, 무엇이 잘못되었는지 알아내기 위해 몇백 MB의 로그파일을 뒤져봐야 하는 것을 의미합니다.

## 4. Interface Segregation Principle (ISP)

![](/assets/images/2020-10-21-14-06-43_2020-10-08-solid_principles.md.png)

ISP는 고객이 사용하지 않는 인터페이스에 의존하도록 강요하지 않아야 하는 것을 의미합니다. 우리가 응집되지 않은 인터페이스를 가지고 있을 때, ISP는 더 작고, 다양하고, 응집된 인터페이스를 만들도록 안내합니다.

ISP를 적용하면, 클래스와 클래스의 의존성은 타이트하게 초점을 맞춘 인터페이스를 이용해 통신함으로써, 사용되지 않는 멤버에 대한 의존성을 최소화하고, 이에 따라 결합력도 낮춥니다. 작아진 인터페이스는 구현하기 쉽고, 유연성과 재사용성을 높여줍니다. 더 적은 클래스가 이러한 인터페이스를 공유함에 따라, 인터페이스 변경에 따라 필요한 변화의 수가 적어지고, robustness(건전성)을 향상시킵니다.

기본적으로, 여기서의 교훈은 "필요하지 않은 것에 의존하지 말라" 입니다. 예를 들자면 다음과 같습니다.

다른 메시지를 표시하고자 하는 화면이 있는 ATM기기를 상상해보세요. 다른 메시지를 표시하는 문제를 어떻게 해결할 수 있을까요? SRP, OCP, LSP를 적용해서 해결책을 제시하지만, 여전히 시스템은 유지보수하기 어렵습니다. 왜일까요?

ATM의 주인이 인출기능만을 위해 나타나는 메시지를 추가하고 싶다고 상상해보세요. 즉, "이 ATM은 인출 시 약간의 수수료를 부과합니다, 동의하십니까?" 라는 메시지를 표현하려고 합니다. 어떻게 해결할까요?

아마, 메신저 인터페이스에 메소드를 추가함으로써 해결할 수 있을 것입니다. 하지만 이로 인해, 인터페이스의 모든 사용자가 다시 컴파일하고, 거의 모든 시스템이 재배포되더야 하는데, 이는 OCP를 직접적으로 위반하고, 코드를 부패시키는 것입니다.

여기서, 인출 기능의 변경이 다른 관련없는 기능들에 변화를 만들었고, 이는 우리가 원하는 것이 아닙니다. 왜 이런 일이 일어났을까요?

**Interface Segregation Principle Example**

사실, 각각의 기능들이 필요로 하지는 않지만, 다른 기능에 의해 필요한 메소드에 의존하는, 거꾸로된 의존성이 있습니다. 이는 우리가 피하고 싶은 것입니다.

```java
public interface Messenger {
  askForCard();
  tellInvalidCard();
  askForPin();
  tellInvalidPin();
  tellCardWasSiezed();
  askForAccount();
  tellNotEnoughMoneyInAccount();
  tellAmountDeposited();
  tellBalance();
}
```

대신, 메신저 인터페이스를 나눔으로써, 다른 ATM 기능들이 분리된 메신저에 의존하도록 해야 합니다.

```java
public interface LoginMessenger {
  askForCard();
  tellInvalidCard();
  askForPin();
  tellInvalidPin();	
}

public interface WithdrawalMessenger {
  tellNotEnoughMoneyInAccount();
  askForFeeConfirmation();
}

public class EnglishMessenger implements LoginMessenger, WithdrawalMessenger {
  ...	
}
```

## 5. Dependency Inversion Principle (DIP)

![](/assets/images/2020-10-21-14-22-09_2020-10-08-solid_principles.md.png)

DIP는, 높은 수준의 모듈은 낮은 수준의 모듈이 아닌 추상화에 의존해야 하는 것을 의미합니다. 둘 째로, 추상화는 세부적인 것에 의존하지 않아야 하고, 세부적인 것들은 추상화에 의존해야 합니다. 핵심은 클래스를 추상화에 의해 형성된 경계선 뒤에 클래스를 고립시키는 것입니다. 추상화 뒤에 있는 모든 세부사항들이 변경되더라도, 클래스는 여전히 안전합니다. 이는 결합력을 낮추고, 설계를 쉽게 바꿀 수 있도록 해줍니다. DIP는 things(아마 코드를 의미하는듯) 독립적으로 테스트할 수 있도록 해줍니다.

**Dependency Inversion Principle Example**

예시는 다음과 같습니다 : 프로그램은 추상화인 Reader와 Writer  인터페이스에 의존하고, 키보드와 프린터는 이 인터페이스를 구현하는, 추상화에 의존하는 세부사항 입니다. 다음의 예에서, CharCopier는 Reader와 Writer의 세부적인 구현을 알지 못하므로, Reader와 Writer 인터페이스를 구현한다면 어느 Device든 넘길 수 있습니다.

```java
public interface Reader { char getchar(); }
public interface Writer { void putchar(char c)}

class CharCopier {

  void copy(Reader reader, Writer writer) {
    int c;
    while ((c = reader.getchar()) != EOF) {
      writer.putchar();
    }
  }
}

public Keyboard implements Reader {...}
public Printer implements Writer {…}

```