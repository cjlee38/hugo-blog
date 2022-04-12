---
# author: cjlee
categories: python
# cover: /assets/covers/coding.png
date: "2020-09-17T09:54:00Z"
tags: ['null']
title: '# Python Asterisk(*) 다루기'
---

# 들어가며
: Python을 다루다 보면, 어떤 함수에서 여러개의 인자를 받고 싶을때가 있다.  
이럴 때는 보통, list로 묶어서 보내는 경우가 대부분이다.

그러나, Python을 계속해서 쓰다가 보면, 일종의 debugging을 위해 print()함수를 쓸 때가 종종 있는데,  
print() 함수에 다음과 같이 여러개의 인자를 넘겨줄 때에는 list로 묶어서 보내지 않는 것을 볼 수 있다.

```python
print("안녕하세요, 제 이름은", myName, "입니다.")
# "안녕하세요, 제 이름은 홍길동 입니다"
```

위 코드는, "안녕하세요, 제 이름은" 이라는 문자열, myName이라는 변수, 그리고 "입니다." 라는 문자열의  
총 세개의 변수를 넘겨줬는데, 정상적으로 처리가 되는 모습을 볼 수 있다.

이것이 어떻게 가능한 것일까? 이에 대해서 알아보자.

# What is Asterisk ?
: Asterisk 라는 거창한 이름이 붙었지만, 한글로는 결국 "별표"다.  

`2*3`과 같은 곱하기나, `2**3`과 같은 제곱을 제외하고,  
이 asterisk를 가장 대표적으로 많이 쓰는 케이스는, 함수다.  
다음의 예시를 보자.
```python
def myFunction(*args):
    print(type(args))
    print(args)
    print(*args)

myFunction('alice', 'tom', 'john')
# <class 'tuple'>
# ('alice', 'tom', 'john')
# alice tom john
```
위와 같이, 여러개의 인자를 넘기더라도, 이를 tuple로 받을 수 있고,   
`print(*args)`를 통해 unpacking이 이루어 지는 모습을 볼 수 있다.  
(다시 말해, `print(*args)` 는 `print('alice', 'tom', 'john')` 으로 변환된 것이다)

즉, "내가 인자를 몇 개 받아야 할 지 모를 때" 사용할 수 있다.  

그러나, 다음의 코드를 보면, "그냥 리스트를 써도 되는거 아닌가?" 싶을 수 있다.

```python
def sayHello(*nameList):
    for name in nameList:
        print("Hello", name)
sayHello('alice', 'tom', 'john')
# Hello alice
# Hello tom
# Hello john
```
___
```python
def sayHello(nameList):
    for name in nameList:
        print("Hello", name)
# Hello alice
# Hello tom
# Hello john
```

그러나, 만약 Hello 뒤에 `Hello alice tom john`과 같이 여러 개의 이름을 붙이고 싶다면 어떻게 해야 할까?

아마 asterisk를 쓰지 않는다면, 이렇게 무식한 방법을 취할 수도 있다.

```python
def sayHello(nameList):
    print("Hello", end=' ')
    for name in nameList:
        print(name, end = ' ')
# Hello alice tom alice
```

이렇게 하는 것보단, 다음과 같이 쓰는 것이 깔끔하게 쓸 수 있다.

```python
def sayHello(nameList):
    print("Hello", *nameList)
# Hello alice tom alice
```

또 다른 사용법으로는, 여러 개의 vector를 받아서, 이를 덧셈 연산하는 것에 활용할 수도 있다.
```python
def sumVector(vectorList):
    ret = []
    for pos in range(len(vectorList[0])):
        sum_ = 0
        for vector in vectorList:
            sum_+= vector[pos]
        ret.append(sum_)
    return ret
sumVector([[1, 3], [2, 4], [6, 5]])
# [9, 12]
```

별로 보기 좋은 코드는 아니다. zip() 함수를 응용하자.

```python
def sumVector(vectorList):
    return [sum(i) for i in zip(*vectorList)]
sumVector([[1, 3], [2, 4], [6, 5]])
# [9, 12]
```

아주 간단명료하게 해결할 수 있다.

또한, asterisk를 두 개 써서, dictionary와 같은 key-value의 형태로도 받을 수 있다.  
kwargs 는 keyword arguments의 약자다.

```python
def introduce(**kwargs):
    print("제 이름은", kwargs['name'], "입니다.")
    print("제 나이는", kwargs['age'], "입니다.")
introduce(name="홍길동", age=25)
# 제 이름은 홍길동 입니다.
# 제 나이는 25 입니다.
```

이렇게 알고 나면, print() 함수가 어떻게 구성되어 있는지에 대한 감이 잡힐 것이다.
