---
categories: btb
# cover: /assets/covers/coding.png
date: "2020-10-20T12:11:00Z"
tags: ['null']
title: '# (번역) Value Object of Martin Fowler'
---

---

> 본 포스팅은 마틴 파울러가 [Value Object 에 대해 작성한 글](https://martinfowler.com/bliki/ValueObject.html)을 의역했습니다.  
> 정확한 이해를 위해, 가능하시면 원문을 보시는 것을 추천드립니다.

---

프로그래밍을 할 때, 종종 사물(things)을 복합적으로 표현하는게 유용할 때가 있습니다. 2차원의 좌표는 x 값과 y 값을 가집니다. 돈은 숫자와 통화를 가집니다. 기간은 시작 날짜와 끝 날짜를 가지고, 이것들은 연, 월, 일을 가집니다.

이럴 때, 두 복합 객체가 같은지에 대해서 궁금할 때가 있습니다. (2,3)의 직교 좌표계를 갖는 두 개의 Point 객체가 서로 같다는 것은 말이 됩니다. 어떠한 객체가, 갖고 있는 속성으로 인해, (이 경우에는 x와 y 좌표) 같다고 여겨지면, Value Object 라고 불립니다.

하지만 내가 신중하게 프로그래밍 하지 않는다면, 내 프로그램에서 다음과 같은 행위를 이해하지 못할 것입니다.

Javascript에서, 점(Point) 을 표시한다고 해봅시다.

```javascript
const p1 = {x: 2, y: 3};
const p2 = {x: 2, y: 3};
assert(p1 !== p2);  // NOT what I want
```

슬프게도 이 테스트는 통과합니다. 이는 자바스크립트가 객체를 참조에 의해 동일한지 비교하고, 그것들이 갖고 있는 값은 무시하기 때문입니다.

대부분의 상황에서는, 값 보다 참조를 이용하는 것이 타당합니다. 가령 여러 개의 판매 주문을 가져와서 다룰 때에는, 각 주문을 한 곳에 모아야 합니다. 그리고 Alice의 가장 최근 주문이 배송예정 목록에 있는지 확인해야 한다면, 메모리 참조 혹은 Alice의 주문 ID를 이용해서 확인할 수 있습니다. 이러한 테스트를 할 때에는, 주문에 어떤 요소가 있는지 신경 쓸 필요가 없습니다. 비슷하게, 고유한 주문 번호를 이용해서 Alice의 주문 번호가 배송 목록에 있는지 확인할 수 있습니다.

따라서, 내가 객체를 어떻게 구분하느냐에 따라, Value Object와 Reference Object의 두 종류의 객체로 생각하는 것이 유용합니다. 각 객체가 어떻게 동일성을 다룰지에 대해서 본인이 확실히 알아야 하고, 그 기대에 따라 동작할 수 있도록 프로그래밍 해야합니다. 이를 어떻게 구현할 지는, 내가 사용하는 프로그래밍 언어에 따라 달라집니다.

일부 언어는 모든 복합 데이터를 값으로 다룹니다. Clojure에서 간단한 복합물을 만들면, 다음과 같습니다

```clojure
> (= {:x 2, :y 3} {:x 2, :y 3})  
true
```

이는 모든 것들을 immutable(불변) 값으로 다루는 함수형 스타일입니다.

하지만 함수형 언어를 사용하지 않아도, Value Object를 만들 수 있습니다. 예를 들어, Java에서는 기본적인 Point 클래스는 내가 원하는 방식으로 동작합니다.

```java
assertEquals(new Point(2, 3), new Point(2, 3)); // Java
```

Point class 가 원래의 equals 메소드를, 값에 대한 검사로 override 하는 방법입니다.

Javascript에서도 비슷하게 할 수 있습니다.

```javascript
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  equals (other) {
    return this.x === other.x && this.y === other.y;
  }
}
const p1 = new Point(2,3);
const p2 = new Point(2,3);
assert(p1.equals(p2));
```

여기서 Javascript의 문제점은, 내가 정의한 equals 메소드를 다른 Javascript 라이브러리가 모른다는 것 입니다.

```javascript
const somePoints = [new Point(2,3)];
const p = new Point(2,3);
assert.isFalse(somePoints.includes(p)); // not what I want

//so I have to do this
assert(somePoints.some(i => i.equals(p)));
```

이는 Java의 문제가 아닙니다. 왜냐하면, Object 클래스의 equals 메소드는 core 라이브러리에 정의되어 있고, 모든 다른 라이브러리들은 객체를 비교할 때 이 메소드를 사용하기 때문입니다( `==` 연산자는 주로 원시(primitive) 타입에만 사용됩니다.)

Value Object의 좋은 결과중 하나는, 메모리의 같은 값을 갖고 있는 객체에 대해서 하나의 참조를 갖고 있는지, 다른 객체를 갖고 있는지 신경 쓸 필요가 없다는 것입니다. 하지만, 내가 조심하지 않는다면, 행복한 무지(ignorance)가 문제로 이어질 수 있습니다. Java로 예시를 보여드리겠습니다.

```java
Date retirementDate = new Date(Date.parse("Tue 1 Nov 2016"));

// this means we need a retirement party
Date partyDate = retirementDate;

// but that date is a Tuesday, let's party on the weekend
partyDate.setDate(5);

assertEquals(new Date(Date.parse("Sat 5 Nov 2016")), retirementDate);
// oops, now I have to work three more days :-(
```

이것은 Aliasing Bug 의 예시중 하나입니다. 한 곳의 날짜를 바꾸었는데(`partyDate.setDate(5)`), 내가 기대한 것과 다른 결과를 가져옵니다. Aliasing Bug를 피하기 위해, 간단하지만 중요한 규칙을 따라야 합니다 : Value Object는 immutable 이어야 한다.(변하지 않아야 한다.) 만약 내가 파티 날짜를 바꾸고 싶다면, 새로운 객체를 만들어야 합니다.

```java
Date retirementDate = new Date(Date.parse("Tue 1 Nov 2016"));
Date partyDate = retirementDate;

// treat date as immutable
partyDate = new Date(Date.parse("Sat 5 Nov 2016"));

// and I still retire on Tuesday
assertEquals(new Date(Date.parse("Tue 1 Nov 2016")), retirementDate);
```

물론, Value Object가 정말 불변 이라면, 불변으로 Value Object를 다루는 것이 훨씬 쉽습니다. 객체를 가지고, 어떠한 setting method도 제공하지 않음으로써 이를 쉽게 구현할 수 있습니다. 따라서, 예전 나의 Javascript 코드는 다음과 같이 생겼습니다.

```javascript
class Point {
  constructor(x, y) {
    this._data = {x: x, y: y};
  }
  get x() {return this._data.x;}
  get y() {return this._data.y;}
  equals (other) {
    return this.x === other.x && this.y === other.y;
  }
}
```

불변은 Aliasing Bug를 피하기 위한, 제가 가장 좋아하는 테크닉 중 하나이지만, 변수를 할당할 때, 언제나 객체의 복사본을 만들게 함으로써 버그를 피할수도 있습니다. C# 의 구조체와 같은 언어들은 이러한 기능을 제공합니다.

Value Object로 다룰지, Reference Object로 다룰지는 상황에 따라 달라집니다. 대부분의 경우, 우편 주소는 간단한 텍스트 구조를 가진 Value Object로 다룰 수 있습니다. 하지만 좀 더 정교한 지도 시스템은 우편 주소를 정교한 계층 모델로 연결할 수 있고, 이 때는 Reference 가 타당합니다. 대부분의 모델링 문제와 마찬가지로, 다른 상황에는 다른 해결책이 필요합니다.

일반적으로, 문자열과 같은 일반적인 Primitive 요소를 적절한 Value Object로 만드는 것은 좋은 생각입니다. 전화번호는 문자열로 나타낼 수 있지만, "전화번호 객체"로 만드는 것은 변수와 파라미터를 좀 더 명시적이게 하고(언어가 지원한다면, 데이터 타입 확인), 자연스럽게 검증에 초점을 맞출 수 있게 하며, 적용이 불가능한 동작(예를 들면, 정수 ID 숫자에 대한 연산)을 피하게 해줍니다.

좌표, 돈, 기간과 같은 작은 객체들은 Value Object의 좋은 예시입니다. 그러나, 더 큰 객체도, 개념적인 정체성(ID)을 갖고 있지 않거나, 프로그램 내에서 참조를 공유할 필요가 없는 경우, Value Object로 프로그래밍 될 수 있습니다. 이는 불변성이 기본인 함수형 언어에 좀 더 자연스럽습니다.

Value Object는, 특히 작은 경우에, 경시되는 경향이 있습니다 - 너무 사소해서 생각할 가치도 없는 것으로 보이곤 합니다. 하지만, 좋은 Value Object를 발견하고 나면, 객체들에 더욱 다양한 행위를 만들어 낼 수 있습니다. 이를 맛보려면, Range class를 통해, 다양한 행위를 활용해서 start 와 end 속성을 조작하고, 이를 통해 모든 종류의 중복을 어떻게 막는지 확인해보세요. 이렇게 Domain-specific한 Value Object는 리팩토링의 초점 역할을 하고, 시스템을 급격한 간소화로 이끕니다. 이러한 단순화를 몇번 보면, 깜짝 놀랄 것입니다 - 그리고, 그때 쯤이면, Value Object는 좋은 친구가 되어 있을 것입니다.