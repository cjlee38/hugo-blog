---
# author: cjlee
categories: python
# cover: /assets/covers/coding.png
date: "2020-09-05T00:35:00Z"
tags: ['null']
title: '# Python list 다뤄보기 ( a.k.a. 잡기술 )'
---

# 0. 서론
: 파이썬 사용자에게, 파이썬에서 가장 자주 사용하는 자료형이 무엇이냐고 물어보면, 원시 타입(Primtive type)을 제외하고는, list라고 답할 것이다.

---

> Note. 사실, 파이썬에서는 원시타입이 존재하지 않는다. 우리가 사용하는 int, float .. 등등은 사실 객체이다. 이는 당장 python에서 help(int) 라고만 입력해보면, 다음과 같이 나타난다.

> Help on class int in module builtins:

> class int(object)  
> |  int([x]) -> integer  
> |  int(x, base=10) -> integer  
> |    
> |  Convert a number or string to an integer, or return 0 if no arguments  
> |  are given.  If x is a number, return x.__int__().  For floating point  
> |  numbers, this truncates towards zero.  

> ... 이하 생략

---

이러한 list는 다양한 **잡기술**이 존재하는데, 이를 하나하나 살펴보자.

# 1. list indexing
: 리스트를 indexing 하는 법에 대해서는 많은 사람들이 알고 있다.   
그런데, 이렇게 사용하는 것은 흔하게는 보지 못하였을 것이라 생각한다.

```python
_list = [1,2,3,4]
print(_list[::2])

-> ???
```
기존에는 colon(:) 을 한번만 사용했는데, 두개를 사용한다. 어떤 결과가 나타날까? 정답은 다음과 같다.

```python
print(_list[::2])
-> [1, 3]
```

indexing 없이 그냥 print 했다면 [1,2,3,4] 가 나와야 할텐데, 첫 번째와 세 번째가 나타났다. 즉, colon을 두 번 쓰고 숫자를 입력하면, 이는 **step**이 된다.

즉, _list[start : end : step] 인 셈이고, start와 end가 생략되었으니 전체 list에 대해 순회하면서, step만큼 움직인다.

다시 말해, 기존에 계단을 **한 칸씩** 오르던 것을, 이제는 **두 칸씩** 오르겠다는 말이다.

이를 조금 응용하면, 다음과 같이 사용할 수도 있다.

```python
print(_list[::-1])
-> [4,3,2,1]
```
기존에 계단을 한 칸씩 **오르던 것을**, 이제는 한 칸씩 **내려가겠다**는 의미가 된다.

# 2. list 합치기

: list 를 합치는 방법에는, 다음의 세 가지 방법이 사용된다.
1. \+ operator
2. append()
3. extend()

### 1. \+ operator
: \+ 연산자는 말 그대로, list끼리 더하는 역할을 해준다.  
다음과 같은 list가 존재한다고 해보자.

```python
_list = [1,2,3,'a']
```

그리고, 여기에 새로이 4.5 라는 요소를 추가하고 싶다면, 다음과 같이 사용하면 된다

```python
_list += [3.5]
```

아주 명료하다. 

### 2. append()
: append는 영어로 덧붙이다, 추가하다.. 이런 뜻을 가지고 있다.   
개인적으로는 덧붙이다 라는 표현이 좀 더 걸맞는 듯 하다. 위 list에서, 3.5를 추가하는 방법은 다음과 같다.

```python
_list.append(3.5)
```

이 또한, 아주 간단명료하다.

그러나, 눈썰미가 좋은 분은, 한 가지 차이점을 발견하였을 것이다.

" 1번에서는 list를 더해줬는데, 2번에서는 3.5라는 float을 더해주네요?"
{: .caption}

이는, list 자료형의 append 함수를 help(list.append)를 통해 살펴보면 알 수 있다.

```
append(self, object, /)
: Append object to the end of the list.
```

즉, list의 끝에 float이라는 object를 붙였기 때문에 이와 같이 나타나는 것이다.

따라서, 다음과 같이 사용할 경우, 의도한 대로 결과가 나타나지 않는다.

```python
_list.append([3.5]) 
print(_list)
-> [1,2,3,'a',[3.5]]
```

### 3. extend()
extend() 함수는 결과적으로 1번의 덧셈 연산자와 같은 결과를 내놓는다. 그러나, 다음과 같은 차이점이 있다.

위와 같은 list에서, 역시 3.5를 추가하고 싶다고 해보자. 
```python
_list.extend(3.5)
-> TypeError: 'float' object is not iterable
```

무슨 에러인고 하니, float은 iterable하지 않다는 뜻이다. 즉, 반복할 수 없다는 말이 된다.

이 또한 help(list.extend)를 통해 설명을 보면 알 수 있는데, append와 달리 extend는 **object가 아닌 iterable한 객체를 받기 때문에** 발생하는 일이다.

```
extend(iterable, /) method of builtins.list instance
    Extend list by appending elements from the iterable.
```

즉, 다음과 같이 사용해야 한다.

```python
_list.extend([3.5])
```

---

Note. extend()와 append()는 객체 내에 있는 요소를 변경시키기 때문에, return 값을 반환하지 않는다.(사실 null을 반환한다) 따라서, _list2 = _list.append(3.5) 혹은 _list2 = _list.extend([3.5]) 이와 같이 사용해서는 안되고, 마찬가지로 _list + [3.5]라고 사용한다면, _list에는 3.5가 들어가지 않을 것이다. 

---

### 4.1 extend() + String

그렇다면, **extend()에 string을 넣는다면 어떻게 될까?**  
String은 결국 char의 sequential이므로, iterable한 객체이다. 따라서, 다음과 같이  사용할 수 있다.

```python
_list.extend("Hello")
print(_list)
-> [1, 2, 3, 'a', 'H', 'e', 'l', 'l', 'o']
```

보는 바와 같이, Hello가 각각의 character로 나누어져 하나씩 들어간 모습을 볼 수 있다.

# 3. list comprehension
: list comprehension은 정말 유용하면서도, 몇 번을 사용해도 손가락이 순차적으로 움직이지 않고, 커서가 Home과 End를 들락거리는 마성을 가진 녀석이다.

### 1) 기본
: 일반적인 형태는 다음과 같다.

```python
_list2 = [e for e in _list] # e는 element의 약자이다.
print(_list2)
-> [1, 2, 3, 'a']
```

단순히 이렇게 생긴 것만 봐서는, 얘가 뭘 의미하는 건지 모른다.

일반적인 for문과 차이점이라면, for문 앞에 e라는 녀석이 있고, 그 for문 전체를 []로 감싸고 있다는 것만 보인다.

### 2) 조건문
: 이 list comprehension의 파워는 if/else의 조건문이 붙었을 때 힘을 발휘한다.  예시를 위해, 우리가 _list에서 int형인 녀석만 뽑아오고 싶다고 해보자. 그러면 다음과 같이 작성했을 것이다.

```python
_list2 = []
for e in _list:
    if isinstance(e, int) == True:
        _list2.append(e)
        # 혹은 _list2 += e
print(_list2)
-> [1, 2, 3]
```

---

> Note. isinstance()는 내장함수로써, "이 instance가 해당 class 소속인가요?" 하고 물어보는 것이다. 첫 번째 인자로 instance를, 두 번째 인자로 class를 넣어주면, True/False의 boolean으로 return된다. java의 instanceof와 비슷하다.

---

위 코드를 list comprehension으로 바꾸면, 다음과 같이 작성할 수 있다.

```python
_list2 = [e for e in _list if isinstance(e, int)]
print(_list2)
-> [1, 2, 3]
```

print문을 제외하고, **4줄인 코드가 1줄로 감소**하였다. 또한, 이 문법을 알고 있다면, **가독성**도 훨씬 늘어난다.

단순 if문 뿐만 아니라, else문도 다음과 같이 추가할 수 있다. 다만, **if 하나만 넣는 것과 달리, if else를 같이 쓰면 위치가 다르다는 것**을 명심하자.

```python
_list2 = [e if isinstance(e, int) else 0 for e in _list]
print(_list2)
-> [1, 2, 3, 0]
```

if문의 조건에 맞으면 e를 그대로 사용하고, 아니라면 0을 집어넣겠다는 의미가 된다.

---

> Note 1. 사실, list comprehension에서 사용되는 if else문은 list comprehension에 종속된 문법이 아니라, 변수를 할당할 때에도 사용될 수 있는 문법이다. 다음의 예시를 참고하자.
> ```python
> a = 1
> b = a if a > 10 else 0
> print(b)
> -> 0
> ```

---

> Note 2. Note 1의 문법을 참고하면, 다음과 같이 사용할 수도 있다. 
> 파이썬의 함수를 정의할 때, default 값을 줄 수 있다는 것을 알고 있을 것이다.
> ```python
> def myFunc(a = 1, b = 2):
>   # do something
> ```
> 헌데, default 값에 조건문을 주고 싶다면, 이를 조금 응용하면 된다..
> ```python
> def myFunc(a = 1, b = 2):
>   a = a if b == 2 else 0
>   # do something
> ```
> 이렇게 작성하면, b가 2일 경우에는 a를 그대로 사용하고, 2가 아닐 경우에는 a에는 어떤 값이 들어왔었더라도, 0이 할당될 것이다.

---

### 3) 중첩
: list comprehension은 중첩해서 쓸 수도 있다. 다음과 같은 list가 있다고 해보자.

```python
_list = [[1,2,3], [4,5,6], [7,8,9]]
```

이를 일반적인 list comprehension을 사용해 접근한다면, 첫 번째 요소는 [1,2,3] 이라는 list고, 두 번째 요소는 [4,5,6], 세 번째 요소는 [7,8,9]가 된다.

각각의 1, 2, 3 ... 9 의 int data를 접근하고 싶다면, 다음과 같이 사용하면 된다.
```python
[e for l in _list for e in l]
```
첫 번째 for문에서, 접근하는 요소인 l에 대해서, 다시 한번 for문을 통해 e라는 객체에 접근하겠다는 뜻이다.


이 또한, if else문을 섞어줄 수 있다.
```python
print([e for l in _list for e in l if e < 5])
-> [1, 2, 3, 4]
print([e if e < 5 else 0 for l in _list for e in l ])
-> [1, 2, 3, 4, 0, 0, 0, 0, 0]

_list2 = [[1,2,3], [4,5], [7,8,9]] # 두 번째 list element의 6 제거
print([e if e < 5 else 0 for l in _list2 if len(l) <= 2 for e in l ])
-> [4, 0]
```

그런데, 이렇게 쓰면 쓰면 쓸수록 복잡해지고, 가독성도 떨어진다. 따라서, 이렇게 nested list comprehension을 사용하는 경우는 **중첩 list를 flatten 하는 상황**이 대부분이다.

```python
print([e for l in _list for e in l])
-> [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

---

> Note 1. 그러나, 사실 이 flatten 하는 과정 마저도,  sum() 함수를 통해 쉽게 해결할 수 있다.
> ```python
> _list2 = sum(_list, [])
> ```
> 이 또한 help(sum)을 입력해보자.  
> 
> Help on built-in function sum in module builtins:
>  sum(iterable, start=0, /)  
>    Return the sum of a 'start' value (default: 0) plus an iterable of numbers  
>      
>    When the iterable is empty, return the start value.  
>    This function is intended specifically for use with numeric values and may  
>    reject non-numeric types.  
> 
> sum 또한 iterable 객체를 첫 번째 인자로 받고, 두 번째 인자는 start = 0 으로 default는 숫자이다.
> 따라서, start를 빈 객체로 바꿔준다면, 위 2-1 번에서 설명한 덧셈 연산자와 같은 효과를 발휘하게 된다.

---

> Note 2. 다음과 같이 사용하는 것을 **"최대한 지양"** 해야 한다.
> ```python
> [_list2.append(e) for e in _list]
> ```
> 위 코드를 실행하면, append는 null을 return하므로, list comprehension을 받는 변수가 있다면, 이는 null로 이루어진 list가 된다.
> 이렇게 사용하는 것을, 파이썬 사용자들은 최대한 피하기를 권하고 있는데, 그 이유는  
> 1. 위 코드를 실행하면, null 값이 들어있는 list object가 생성되었다가 소멸되므로, 자원의 낭비가 생기며
> 2. Explicit 하게 작성하는 것을 목표로 하는, 파이썬의 이념(혹은 convention)에 어긋나기 때문이다.  
> 
> 한마디로, 저렇게 쓰는 것은 **"쓸데없는 겉멋"** 이라는 것이다.

---

결론적으로, list comprehenson은 정말 유용한 문법임은 맞지만, 이를 남용하는 일이 없도록 주의해야 한다.

끝.