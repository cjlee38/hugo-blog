---
# author: cjlee
categories: python
# cover: /assets/covers/coding.png
date: "2021-09-16T22:05:00Z"
tags: ['null']
title: '# (Python) @Property까지 가는 길'
---


> Note 1. 개인적으로 공부해가며 작성한 글이기 때문에 틀린 내용이 있을 수 있습니다. 틀린 내용을 발견하시면 언제든 지적 부탁드립니다.  
> Note 2. 이해를 돕기 위해 일부 내용은 Java와 비교해가며 글을 작성했습니다.

# 0. 들어가며
: 한동안 Java를 애용하다가, Python을 다뤄야 하는 일이 생겨서 다시 Python을 붙잡게 되었습니다. Java의 syntax에 대해서 어느정도 이해하고 있다고 자만하던 중이었기 때문에, Python을 사용하면서 "왜 Python에는 Java의 이런 기능이 없지? 역시 Java가 짱이야" 라는 생각도 하곤 했습니다.

그런데 python의 getter와 setter에 대해서 알아보던 중, `@property` 라는 녀석을 이해해보려고 자료를 찾다 보니, Python의 진면목 일부를 엿볼 수 있는 기회가 되었습니다. 

<center>
<iframe src="https://giphy.com/embed/cNO6wb9sEzK8N9rp5E" width="480" height="480" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/twitter-bravo-bravocon-con-cNO6wb9sEzK8N9rp5E"></a></p>
누가 파이썬이 쉽다 했습니까?
</center> 

python의 아주 기본적인 문법만 알고 있던 사람들에게 본 글이 시야를 넓히는데 도움이 되길 바라면서, 시작하겠습니다.

# 1. 그래서 @Property 가 뭔가요 ?
흔히 "객체지향" 에서는, 객체가 갖고 있는 `Field` 를 적절하게 외부로 노출시키거나 숨기기 위해, `getter/setter`를 사용합니다.

Java에서는 `private` 이라는 키워드로 갖고 있는 멤버변수들을 모두 숨기고, 보여줄 내용은 `getter` 메소드를, 외부에 의해 변경되어도 되는 변수는 `setter` 메소드를 활용해 데이터를 관리하죠. 

따라서 직접적으로 변수에 접근하는 것은 원천적으로 금지되어 있고, 메소드 호출을 통해서만 데이터를 조작할 수 있습니다.

```java
class Person {

    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() {
        return this.name;
    }

    public int getAge() {
        return this.age;
    }

    public void setAge(int age) {
        if (age <= 0) 
            throw new IllegalArgumentException("나이는 0 이하가 될 수 없습니다.");
        this.age = age;
    }
}

public class Main {
    public static void main(String[] args) {
        Person person = new Person("Mr.Kim", 26);
        System.out.println(person.getName()); // "Mr.Kim";
        person.setAge(-62); // IllegalArgumentException("나이는 0 이하가 될 수 없습니다.")
    }
}
```

그러나 python 에서는 programming language 수준에서 접근 권한을 제어할 수 없습니다. 따라서 _(언더스코어) 를 활용하는 방법을 제공하지만, 결국 `_{클래스명}__{변수명}` 으로 접근이 가능합니다.(이후 뒤에서 설명할 property도 결국 직접 접근하는 방법은 있습니다.)

따라서 python 에서는 기본적으로 모두 `public` 이라고 생각하면 됩니다. 하지만 그렇다고 해서 `getter/setter`를 쓸 필요가 없다는 것은 아닙니다. 당장 위와 같은 예시만 보더라도, `setAge()` 메소드를 호출할 때, 나이가 음수인지 아닌지 검증하는 로직이 필요하기 때문입니다. 위 코드를 python 으로는 다음과 같이 쓸 수 있습니다.

```python
class Person :
    def __init__(self, name, age) :
        self._name = name
        self._age = age
    
    # 다른 메소드는 편의상 생략합니다.

    def set_age(self, new_age) :
        if new_age <= 0 :
            raise Exception("나이는 0 이하가 될 수 없습니다.")
        self._age = new_age

if __name__ == '__main__' :
    person = Person("Mr.Kim", 26)
    person.set_age(-62) # Exception('나이는 0 이하가 될 수 없습니다.')
```

여기까지는 Java와 똑같습니다. 하지만, 이런 식의 코드 사용은 `explicit` 하지 못하다는 단점이 있습니다. 만약 기존 나이에서 1살을 더해줘야 한다면, 다음과 같이 작성해야 합니다.

```python
person.set_age(person.get_age() + 1)
```

뿐만 아니라, `person._age = -62` 와 같은 코드는 방어할 수가 없어진다는 문제도 여전히 남아있죠. 그러나 `@property` 를 활용하면 다음과 같은 코드가 가능합니다.

```python
person.age += 1
person.age = -62 # Exception('나이는 0 이하가 될 수 없습니다.')
```

??? 어떻게 된 일일까요?   
변수에 값을 직접 대입하는 것 같은데, 로직에 따른 Exception을 던지고 있습니다. 어떻게 이것이 가능한지는 차차 알아보기로 하고, 일단 생김새부터 살펴봅시다. 위 `Person` 클래스에 `@Property`를 적용하면 다음과 같이 작성할 수 있습니다.

```python
class Person :
    def __init__(self, name, age) :
        self._name = name
        self._age = age
    
    @property
    def age(self) :
        return self._age
    
    @age.setter
    def age(self, new_age) :
        if new_age <= 0 :
            raise Exception("나이는 0 이하가 될 수 없습니다.")
        self._age = new_age
```

대충 살펴 봤을때, 눈에 띄는 차이점은 다음과 같습니다.

1. 함수의 이름이 `get_age` 혹은 `set_age` 가 아니라, 그냥 `age` 자체로 바뀌었습니다.
2. 계속 언급했던 `@property` 가 `age` 메소드 위에 붙었습니다. 또한, `@age.setter` 라는 녀석도 두 번째 `age` 메소드 위에 붙었습니다.

이러한 변화가 어떻게 앞서 언급했던 동작에 영향을 미쳤는지 하나씩 알아가봅시다. 

`@property`를 이해하기 위한 배경지식들은 다음과 같습니다.
1. Everything is Object
2. Decorator
3. Descriptor Object


# 1. Everything is Object
Python에서 다루는 모든 값은 객체입니다. Java에서는 일반적으로 값을 `Primitive Type`, `Reference Type` 으로 구분하죠. 반면 Python에서는 `int`, `bool`과 같은 값들도 모두 객체로 다뤄집니다.

```python
a = 1
print(id(a)) # 4341049696
```

`id` 함수는 Built-in 함수로, 객체가 가지고 있는 주소값을 나타냅니다. 따라서 `id(a)`라는 코드는 1 이라는 값이 할당된 메모리 주소를 반환합니다. Java에서는 `Integer` 와 비슷하다고 생각하시면 될 것 같습니다.

이에 더해, Python 에서는 **함수마저도** 객체입니다.

```python
def my_func() :
    return "hello world"

print(my_func.__str__()) # '<function my_func at 0x7ffd70cf1f70>'
```

위 예시에서 보시다시피, `my_func` 이라는 이름의 함수를 만든 뒤, 해당 함수가 갖고 있는 `__str__` 이라는 메소드를 호출할 수 있습니다. 즉, `my_func` 자체는 `Function` 이라는 클래스를 상속하는 클래스입니다.  

간단하게 생각하면, `def my_func()` 라는 키워드는 `my_func = {함수 바디}` 와 같이 변수로 정의되는 것을 상상해볼 수 있겠습니다.

> Note. 함수가 객체라는 이야기를 할 때, `1급 시민` 이라는 키워드가 자주 등장합니다. 1급 시민이란, 변수에 할당할 수 있어야 하고, argument로 전달될 수 있으며, return 으로 반환될 수 있는 값들을 말합니다. Java에서 메소드는 일급 객체가 아니지만, 인터페이스를 활용해 1급 시민의 흉내를 낼 수 있게 되었습니다.

# 2. Decorator
데코레이터를 이야기하려면 Python 함수가 갖는 또 다른 특징을 알아야 합니다. 

## 1) 함수는 중첩이 가능하다.

python은 다음과 같이 함수를 중첩해서 정의할 수 있습니다.
```python
def outer() :
    def inner() :
        print('inner called')
    print('outer called')
    inner()
    return inner

var = outer() # outer called \n inner called (1)
var() # inner called (2)
```

`outer` 함수 안에 `inner` 함수를 정의한 뒤, `inner` 함수를 호출했습니다. 따라서 `var = outer()` 를 통해 `outer`함수를 호출할 때, 내부에 있는 `inner` 함수 (1)도 실행됩니다. 

다음으로 `return inner`를 통해 `inner` 함수 그 자체를 돌려주고, 해당 `함수 객체`는 `var` 라는 변수에 저장됩니다. 따라서 `var()` 를 통해 함수를 실행하는 것은 결국 `inner` 함수를 실행하는 것과 같습니다.

## 2) Enclosed Function Locals
python의 함수가 갖는 두 번째 특징은, 이러한 중첩 함수로 인해 조금 특별한 변수 scope가 생긴다는 것입니다. 

C를 처음 배웠을 무렵, 지역변수 전역변수와 같은 개념에 대해서 배우고, "내가 허용된 범위의 바깥 변수는 알 수 없다", "가까운 scope부터 찾아 나간다" 정도의 개념으로 정리하고 넘어갔었습니다.

python에서는 후자, 즉 "가까운 scope부터 찾아 나간다"는 맞는 말이지만, 전자인 "내가 허용된 범위 바깥 변수는 알 수 없다" 는 조금 다릅니다. 요약하자면, "내가 허용된 범위 바깥 변수에 접근할 수는 있지만, 수정할 수는 없다." 정도가 될 것 같습니다.

```python
x = 1
def outer() :
    y = 2
    def inner() :
        z = 3
        print(z, y, x)
    inner()
    return inner

a = outer() # 3 2 1
a() # 3 2 1
```
아까 예시를 들었던 `outer/inner` 함수를 약간 변형해보았습니다. `outer` 함수가 실행되던 와중에 `inner`가 호출되든, 함수 객체를 return 받은 `a`가 호출되든 상관없이, 모두 3 2 1을 출력합니다.

> Note. python에서는 가까운 곳부터 찾아나가는 접근법을 **LEGB 규칙** 이라고 명명했습니다. 순서대로 Local, Enclosed function locals, Global, Built-in 의 약자이며, `z`, `y`, `x` 변수가 이에 해당합니다. 마지막 Built-in 변수는 __name__과 같이 "내가 정의하지도 않았는데 기존에 정의되어 있던 값들"을 말합니다.

하지만 다음의 예시는 쓰기에 대해서는 조금 다른 결과임을 보여줍니다.

```python
x = 1
def outer() :
    y = 2
    def inner() :
        z = 3
        y += 1 # UnboundLocalError: local variable 'y' referenced before assignment
        x += 1 # UnboundLocalError: local variable 'x' referenced before assignment

        print(z, y, x)
    inner()
    return inner

a = outer()
a()
```

`inner` 내부에서 `y`와 `x`에 대해서 값을 1씩 더하려고 하자 할당되기 전에 참조되었다는 에러를 발생시킵니다. 즉, 자기 자신의 scope 바깥에 있는 변수에 대해서는 "쓰기" 연산에 대해, 즉 "변경"에 대해 제한되어 있습니다.

> Note. 만약 해당 값을 수정하고 싶다면 `nonlocal` 혹은 `global` 키워드를 통해 "내가 외부의 값을 수정하고자 한다" 라는 의도를 드러내야 합니다.

## 3) Closure

Closure는 위키피디아에서 다음과 같이 설명하고 있습니다.

```
컴퓨터 언어에서 클로저(Closure)는 일급 객체 함수(first-class functions)의 개념을 이용하여 스코프(scope)에 묶인 변수를 바인딩 하기 위한 일종의 기술이다.
```

이러한 설명은 처음 Closure를 접한 사람들에게 전혀 도움이 되지 않습니다. 심지어 `Closure`라는 네이밍 그 자체에서도 어떠한 힌트를 얻기는 어렵습니다. 예시를 살펴보면서 이해해보겠습니다.

```python
def outer(val) :
    def inner() :
        print('inner called')
        print(val)
    print('outer called')
    return inner

a = outer(10) # ???
a() # ???
```
첫 번째로 실행한 `a = outer(10)`, 두 번째로 실행한 `a()` 코드가 각각 어떻게 돌아갈지 한번 상상해볼까요?

`a = outer(10)`에 의해 `outer` 함수를 호출했고, argument로 `10`을 넘겨주고, 이를 val 이라는 이름으로 받아냅니다. 그 내부에서 `inner` 함수를 정의했고, `outer called` 를 출력한 뒤, `inner` 함수 객체를 돌려줍니다.

두 번째로 `a()` 를 통해, `a` 변수가 갖고 있는 `inner` 함수 객체를 호출합니다. `inner called`를 출력한 뒤, `val`을 출력하려고 하는데, **여기서 의문점이 하나 생깁니다.** 해당 함수는 자신 바깥에 있는, 즉 `val` 변수를 참조하고 있습니다. 앞서 이야기한 `LEGB 규칙`에 따르면, `L` 에서 `E`를 참조하는 셈이죠. 

여기서 `val`은 살아 있을까요? 바꿔말하면, `val` 변수를 참조해서 아까 넣어준 `10` 이라는 값을 얻어낼 수 있을까요? 정답은 **"YES"** 입니다.

이렇게 보니 마치 `outer` 라는 클래스가 `val` 이라는 멤버변수를 가지고 있는 것처럼 느껴집니다. 여기까지 읽고 나서, 다시 위키피디아의 정의를 살펴보겠습니다. 

```
컴퓨터 언어에서 클로저(Closure)는 일급 객체 함수(first-class functions)의 개념을 이용하여 스코프(scope)에 묶인 변수를 바인딩 하기 위한 일종의 기술이다.
```

이렇게 보니 100% 이해하지는 못하더라도, 어떤 느낌으로 설명하고 있는지는 알 것 같습니다. `Closure`라는 이름 또한 직역 시 "폐쇄" 라는 뜻을 갖고 있는데, 조금 과장해서 "폐쇄망" ... "인트라넷" ... "나만의 구역" .. 과 같은 연결고리로 생각이 들기도 합니다.

> Note. 어떠한 함수 A 가 Closure이기 위해서는 다음의 세 가지 조건이 만족되어야 합니다.  
> 1. A 함수는 다른 함수 B 안에 정의된 함수, 즉 중첩된 함수여야 한다.  
> 2. 자신을 둘러싼 함수 B scope 의 값을 참조해야 한다.
> 3. B 함수는 A 함수를 반환해야 한다.

여기서 한 단계 더 나아가보겠습니다.

```python
def outer(func) :
    def inner() :
        print('inner called')
        func()
    print('outer called')
    return inner

def my_func() :
    print('my_func called')

a = outer(my_func) # outer called
a() # inner called \n my_func called
```

앞서는 `val` 이라는 숫자 값을 넣어줬지만, 이번에는 `my_func` 이라는 함수를 넣어주고 있습니다. 거듭 반복해서 보여드리고 있듯이, 함수 또한 객체이기 떄문에(a.k.a 1급 시민) 다른 함수의 argument로 전달될 수 있습니다.

여기서는 `outer` 라는 함수의 argument로 `my_func`을 전달하고 있고, 좀 전과 마찬가지로 `my_func`는 `outer` 함수의 호출이 끝나더라도 `inner`에 의해 참조되고 있습니다.

## 4) 드디어..
`Decorator`를 알기 위한 사전 지식은 준비되었으니, 이제 본격적으로 알아보겠습니다.

바로 위의 예시에서는 `a = outer(my_func)` 라는 함수를 통해 closure를 만들고, `a()` 를 통해 `inner` 함수, 즉 closure 를 호출했습니다. Decorator를 활용하면 이를 보다 손쉽게 정의할 수 있습니다.

즉, 다음의 두 코드는 같습니다.

기존
```python
def outer(func) :
    ...

def my_func() :
    ...

a = outer(my_func) # outer called
a() # inner called \n my_func called
```

데코레이터
```python
def outer(func) :
    ...

@outer  # outer called
def my_func() :
    ...

a = my_func() # inner called \n my_func called 
```

여기서 한 가지 주의할 점은, 데코레이터 버전에서 `outer called` 는 `@outer`를 붙여서 `my_func`를 정의하는 시점에 호출된다는 점입니다.

이렇게 보니 `Decorator` 라는 이름이 왜 `Decorator` 인지 알 것 같습니다. 또한, Spring의 AOP는 Python에서는 Decorator를 활용해 간단히 구현할 수 있겠다는 생각도 듭니다.

# 3. Descriptor Object

Descriptor 는 객체 A가 갖고있는 속성인 객체 B를 다룰 때 조회, 저장 및 삭제를 사용자 정의할 수 있는 객체 B를 디스크립터라고 이야기합니다.

이 Descriptor라는 녀석은 객체 A의 클래스 변수로서 정의되어야 합니다. (the descriptor must be in either the owner’s class dictionary or in the class dictionary for one of its parents)

역시, 예시를 보면서 진행하겠습니다.

```python
class Age :
    def __init__(self, age) :
        self._age = age
        
    def __get__(self, obj, objtype) :
        print("__get__ method called")
        return self._age
    
    def __set__(self, obj, val) :
        print("__set__ method called")
        self._age = val
        
    def __delete__(self, obj) :
        print("__delete__ method called")
        self._age = -1
        
class Person :
    age = Age(26)

p = Person()
print(p.age) # __get__ method called \n 26
p.age = 20 # __set__ method called
del p.age # __delete__ method called
print(p.age) # __get__method called \n -1
```

Age 라는 클래스를 만들고, `__get__`, `__set__`, `__delete__` 메소드를 정의했습니다. 이 세개의 메소드는 **스페셜 메소드** 라고 불리우며, 객체가 갖는 특성을 지정할 수 있습니다. 가장 대표적으로 `__init__` 메소드는 인스턴스 생성시에 동작하는 행위에 대해서 정의할 수 있습니다.

어떠한 객체에 위에서 언급한 세 개의 `__get__`, `__set__`, `__delete__` 메소드가 정의되었을 경우, 해당 객체를 "디스크립터" 라고 부르게 됩니다.

> Note. `__get__` 메소드만 정의된 경우에는 `Non-data Descriptor`, `__set__` 메소드 혹은 `__delete__` 메소드가 정의된 경우에는 `Data Descriptor` 라고 부르게 됩니다. 이 둘의 차이는 우선순위에 있습니다만, 여기서는 다룰 내용이 아닌지라 스킵하겠습니다.

이렇게 Descriptor를 정의할 경우, 해당 객체로 접근할 때 우리가 생각했던 것처럼 값에 직접 접근하는 것이 아니라, 메소드 호출을 통해 접근하게 됩니다. 그림으로 보면 다음과 같이 상상해 볼 수 있겠습니다.

![](/assets/images/2021-09-18-15-52-24.png)

Descriptor 라는 특별한 개념이 없다면, 코드 상으로는 객체가 위 그림과 비슷한 형태로 구성될 것이라고 예측할 수 있습니다. `Person` 안에 `Age` 객체가, 그리고 `Age` 객체 안에 `_age` 라는 멤버변수, `__getter__`, `__setter__` 라는 메소드를 가질 뿐이죠. 따라서 `p.age` 로 접근하게 되면 `Age()` 객체로 접근할 것처럼 보입니다. 하지만 실질적으로는 다음 그림과 같습니다.

![](/assets/images/2021-09-18-15-56-29.png)

`p.age`를 호출하게 되는 순간, `Person` 객체 내에 있는 `Age()` 객체를 가져오려 하는데, `__get__` 메소드가 오버라이드 되어 실질적으로는 `Age()` 객체가 아닌 그 내부의 `self._age`를 돌려주게 됩니다. 

이러한 동작 방식을 보고나면, 왜 `Data Descriptor` 라는 이름을 갖게 되었는지 이해되기 시작합니다. `Data Descriptor`로 정의된 객체는, 일반적으로 생각하는 것처럼 값을 갖고 메소드라는 행위를 갖지 않고, 대신 내가 원하는 `Data`(여기서는 _age라는 int 변수겠죠)에 대한 톨게이트와 같은 역할을 하게 됩니다.

# 4. Property
장황한 이야기를 끝내고, 드디어 Property에 대한 이야기를 해보겠습니다. 앞서 보여드렸던 Property 사용 예시는 다음과 같습니다.

```python
class Person :
    def __init__(self, name, age) :
        self._name = name
        self._age = age
    
    @property
    def age(self) :
        return self._age
    
    @age.setter
    def age(self, new_age) :
        if new_age <= 0 :
            raise Exception("나이는 0 이하가 될 수 없습니다.")
        self._age = new_age
```

Decorator에 대해서 이해하고 왔으니, `@property` 라는 코드를 통해, `property` 라는 함수(함수는 다시 객체이니 정확히는 `property`라는 객체겠죠)로 `age` 라는 메소드를 전달한다는 사실을 알 수 있습니다.

따라서, 아래와 같은 코드는

```python
@property
def age( ... ) :
    ...
```

실질적으로는 다음과 같다고 볼 수 있겠죠.

```python
age = property(age)
```

아까전에 `def my_func()` 라는 코드는 `my_func = {함수바디}`와 같이 변수로 정의되는 것을 상상해볼 수 있다고 말씀드렸었는데요. 같은 맥락임을 확인할 수 있습니다. 왜냐하면, `@property` 데코레이터를 사용해 `age = property(age)`로, 즉 클래스 변수로 만들었기 때문입니다. (Descriptor는 반드시 클래스 변수에 있어야 한다는 사실을 떠올려보세요.)


[파이썬 공식 문서](https://docs.python.org/3/howto/descriptor.html)에 있는 `property`의 생김새는 다음과 같습니다.

```python
class Property:
    "Emulate PyProperty_Type() in Objects/descrobject.c"

    def __init__(self, fget=None, fset=None, fdel=None, doc=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        if doc is None and fget is not None:
            doc = fget.__doc__
        self.__doc__ = doc

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        if self.fget is None:
            raise AttributeError("unreadable attribute")
        return self.fget(obj)

    def __set__(self, obj, value):
        if self.fset is None:
            raise AttributeError("can't set attribute")
        self.fset(obj, value)

    def __delete__(self, obj):
        if self.fdel is None:
            raise AttributeError("can't delete attribute")
        self.fdel(obj)

    def getter(self, fget):
        return type(self)(fget, self.fset, self.fdel, self.__doc__)

    def setter(self, fset):
        return type(self)(self.fget, fset, self.fdel, self.__doc__)

    def deleter(self, fdel):
        return type(self)(self.fget, self.fset, fdel, self.__doc__)
```

자, 이제 어떻게 코드가 동작하는지에 대해서 이해할 준비가 되었으니, 한 줄씩 따라가보며 총 정리해보겠습니다.

```python
@property
def age( ... ) :
    ...
```

위 코드를 통해 `age = property(age)` 가 실행된다고 말씀드렸었죠. `property`의 생성자 인자로 `age`라는 메소드를 넘겨줬으니, 이 age는 `__init__` 메소드 내부에서 `fget` 이라는 변수에 할당됨을 확인할 수 있습니다. 그리고 `@property`를 적용한 `Person` 클래스의 현재 생김새는 다음과 같습니다.

```python
class Person :
    def __init__(self, name, age) :
        self._name = name
        self._age = age
    
    def age(self) :
        return self._age

    age = property(age)
    
    @age.setter
    def age(self, new_age) :
        if new_age <= 0 :
            raise Exception("나이는 0 이하가 될 수 없습니다.")
        self._age = new_age
```

다음으로 `@age.setter` 부분을 살펴봅니다. 현재 `age`는 `property` 객체인 상태입니다. 그러면 `@age.setter`는 다음과 같이 해석됩니다. age가 계속 나와 헷갈리니 주석을 참고하세요.

```python
age = property(age)
# age(1번) = property(age(2번))
# age 라는 이름의 변수(1번)에 'age 메소드'(2번)를 인자로 넘긴 property 객체를 저장합니다.
age = age.setter(age) 
# age(3) = age(1).setter(age(4))
# age라는 이름의 변수(3번)에, property 객체를 갖고 있는 age 변수(1번)의 setter 메소드에 새로운 age 메소드(4)를 넘겨줍니다.
```

그리고 `setter` 메소드를 살펴볼까요?

```python
def setter(self, fset):
        return type(self)(self.fget, fset, self.fdel, self.__doc__)
```
`type(self)` 부분은 자기 자신에 대한 `type`을 구하고 있습니다. `self`는 `property` 이니 다음과 같이 해석됩니다.

```python
def setter(self, fset):
        return property(self.fget, fset, self.fdel, self.__doc__)
```

자기 자신에 대해서 생성자를 호출하고 있네요. 꼭 마치 자바에서 생성자에 `this()`와 비슷해 보입니다. 또한, 나머지는 `self` 키워드를 붙여서 자기 자신이 갖고 있던 녀석들을 넘겨주고, `fset` 위치에만 `age(self, new_age)` 라는 메소드를 넘겨주고 있습니다. 

이제 `Person` 객체에서 `age`를 접근할 때 무슨일이 일어날까요? `p.age` 로 꺼낸 `age`는 `property` 객체라는 사실은 이 쯤 되면 잘 아실거라 생각합니다. `property` 객체는 연이어 `__get__` 메소드를 호출하겠죠. 

```python
def __get__(self, obj, objtype=None):
    if obj is None:
        return self
    if self.fget is None:
        raise AttributeError("unreadable attribute")
    return self.fget(obj)
```

`__get__` 메소드의 인자 `obj`는 자기 자신을 실행한 인스턴스, `objtype`은 해당 클래스를 의미합니다. `p.age`로 인스턴스를 통해 실행했으니 `obj`는 p일 것이고, `objtype` 은 `Person` 이네요.   
`obj`가 `None`이 아니고, `fget` 또한 아까 할당되었으니, `return self.fget(obj)`가 실행됩니다. 

즉, `fget(obj)` 의 fget은 앞서 정의한 `def age(self)` 이니, 아까 내가 정의한 `getter` 메소드, 즉 다음 메소드가 실행됩니다.

```python
def age(self) :
    return self._age
```

다음으로 `__set__` 메소드를 살펴볼까요? `p.age = -10` 이라고 작성했다면, 역시 `p.age` 변수가 갖고 있는 객체는 `property` 객체이고, 해당 객체의 `__set__` 메소드가 호출됩니다. 

```python
def __set__(self, obj, value):
    if self.fset is None:
        raise AttributeError("can't set attribute")
    self.fset(obj, value)
```

마찬가지로 obj는 자기 자신을 실행한 인스턴스, 즉 `p` 이고, `value`는 `-10` 입니다. `fset` 변수는 앞서 할당한 `def age(self, new_age)` 함수 객체이고, None이 아니니 `self.fset(obj, value)`를 실행합니다. 즉, 다음 메소드가 실행됩니다.

```python
def age(self, new_age) :
    if new_age <= 0 :
        raise Exception("나이는 0 이하가 될 수 없습니다.")
    self._age = new_age
```

마지막으로 `__delete__` 는 `__setter__`와 같으니 추가적으로 다루지는 않겠습니다. 

# 5. 마무리

여러 개념을 융합해서 새로운 개념을 이해하는 것은 언제나 어려운 일이기 때문에 꽤 구구절절 설명하는 경향도 없잖아 있었습니다. 그러다보니 약 600줄에 가까운 글이 되었습니다.

그렇기에 한줄한줄 곱씹어가며 글을 읽으셨다면 크게 어렵지 않으셨을 것이라 생각합니다. 그 동안 프로퍼티를 사용만 하고 그 동작 원리에 대해 잘 모르셨던 분들에게 좋은 설명이 되었기를, 그리고 기존 프로그래밍의 사고를 확장시킬 수 있는 계기가 되었기를 바랍니다.

감사합니다.

---


## Reference

* [Python의 Closure에 대해 알아보자](https://shoark7.github.io/programming/python/closure-in-python)
* [Wikidocs 레벨업 파이썬 - 클로저](https://wikidocs.net/81226)
* [Python 공식문서 - Descriptor](https://docs.python.org/3/howto/descriptor.html)
* [Hardcore in Programming - Descriptor](https://kukuta.tistory.com/339)
* [[Python 지식]- Descriptor 우선순위](https://m.blog.naver.com/r00tdr4g0n/222056023801)
