---
# author: cjlee
categories: python
# cover: /assets/covers/coding.png
date: "2020-08-27T01:47:00Z"
tags: ['null']
title: '# Python string prefix에 관하여'
---
# 0. 서론
: 데이터 수집 모듈을 작성했던 인턴쉽 과정 동안, 파이썬을 많이 애용했다. 초기 작성은 그럭저럭 잘 되가나 싶었는데, 가장 해결하기 힘들었던 문제 두가지를 꼽자면, 비동기와 인코딩이 되겠다.

비동기란 개념을 이번에 처음 알게되었는데, 각고의 구글링과, 테스트를 통해 어찌어찌 기워넣는 수준으로 비동기를 적용하였다. 여담이지만 체감상, 구글링해서 나오는 자료의 대부분은 자바스크립트에서 개념을 갖고와 설명해줬고, 나머지 대부분은 코드만 보여주고 "이게 비동기 요청입니다" 하는데, 제각기 사용법도 다르고, 실제 몇번 테스트를 해봤을 때도 성능이 다른 것을 확인할 수 있었다. 일부 아주 훌륭하신 분들이 작성한 게시글에 큰 도움을 받았다.

인코딩은 추후에 다시 공부해서 작성하겠지만, 직접적으로 해결한 것이 아닌, 간접적으로 우회해서 해결하는 방식을 채택했기 때문에 해결했다고 보기도 어려울 것 같다. 와중에, 계속해서 눈에 밟히던 string prefix에 대해서 가볍게 다뤄보고자 한다.
# 1. What is the string prefix ?
: 별 거창한 건 아니고, 문자열 앞에 붙는 알파벳을 말한다.

가령, 근래에 파이썬을 배우신 분들은 이런 코드를 본 적이 있을것이라 생각한다.

```python
>>> name = "Jack"
>>> string = f"Hi, my name is {name}"
>>> print(string) 

Hi, my name is Jack
```
string에 문자열을 할당할 때 앞에 보이는 f 가 바로 prefix의 일종이다.

이러한 prefix의 종류는 파이썬 3.7 기준 4가지로, r, b, u, f가 있다. 이에 대해서 살펴보자.

편의상 f, b, u, r 순으로 설명한다.



# 2. String prefix
###    1) f - format
: 바로 위 예시에서 보이듯, 따옴표 앞에 f를 붙여주니 { } 중괄호가 그대로 쓰이지 않고, 변수를 받는데 사용된 것을 볼 수 있다. 이는 PEP 498, 파이썬 버전으로는 3.6부터 새로 도입된 기능이다.

이렇게 작성하는 주된 이유는 가독성인데, 위 예제를 좀 더 복잡하게 만들어보자.
```python
>>> name = "Jack"
>>> age = "25"
>>> sex = "male"
>>> major = "Computer Science"
>>> favorite_food = "kimchi"
```
이름에 더해서, 나이, 성별, 전공, 좋아하는 음식까지 추가했다.

이를 출력하려면, 다음과 같은 방법들이 사용될 수 있다.


```python
# A
>>> print("Hi, my name is " + name + ", I'm " + age + " years old, and my gender is " + sex + ". I'm majoring in " + major + ". I like " + favorite_food)
# B
>>> print("Hi, my name is {}, I'm {} years old, and my gender is {}. I'm majoring in {}. I like {}".format(name, age, sex, major, favorite_food))
# C
>>> print(f"Hi, my name is {name}, I'm {age} years old, and my gender is {sex}. I'm majoring in {major}. I like {favorite_food}")

Hi, my name is Jack, I'm 25 years old, and my gender is male. I'm majoring in Computer Science. I like kimchi
```

A 방법은 가장 일반적으로 string을 concatenate 하는 방법이지만, 중간중간 쉼표, 띄어쓰기 등을 작성하기가 굉장히 까다롭다. + 대신 , 를 붙이는 방법도 있겠지만, 그래도 쉽지 않다.

B 방법은 그나마 손이 좀 덜가지만, 각 순서를 따지기가 어렵다. 실수로 age가 들어가야 할 자리에 sex 변수를 할당할 실수의 여지가 있다.

C 방법은, 손도 그나마 덜가고, 변수 할당의 실수도 적다. 따라서, C 방법을 가장 애용하는 편이다. 결국 작성하는 사람이 편한대로 쓰겠지만, 좋은거 쓰라고 줬는데 안 쓸 이유가 없지 않다.

여담으로, 요새 자바스크립트를 배우고 있는데, 자바스크립트에서 다음과 같은 템플릿 리터럴과 흡사하다.

```javascript
const value = "20";
console.log(`value is ${value}`)
```

###   2) b - bytes
: b는 bytes를 의미한다. 우리가 보는 데이터들은 모두 바이트로 저장되어야 한다. 추후에 다른 게시글로 자세히 서술하겠지만, 간단히 이야기하자면, 우리가 다루는 모든 데이터들은 특정 "인코딩"을 통하여 바이트로 변환이 되고, 이 바이트를 저장했다가, 필요할 때 다시 "디코딩"을 통하여 사람이 알아볼 수 있게 해석해내는 것이다.


```python
>>> _str = "this is a string"
>>> _bytes = b"this is a string"

>>> print(_str)
>>> print(_bytes)

this is a string
b'this is a string'
```

위 코드를 보면, "어, 바이트인데 앞에 b가 붙은거 말고는 차이가 없는데요?"라고 할 수 있다. 그러나, 실상은 바이트코드이고, 사람에게 보여주기 위해서 다시 컴퓨터가 임의로 디코딩한 결과물이다. 이를 명시하기 위에 앞에 b가 붙었다.

###   3) u - unicode
: unicode를 의미하는 u는 파이썬2에서 사용되던 syntax이다. python3는 default encoding이 unicode이므로, 굳이 붙이지 않아도 상관 없다.
```python
>>> u"this is a string" == "this is a string"
True
```
헷갈리는 사람은 다음의 두 글을 살펴보는 것도 좋겠다.



- [What's the u prefix in a Python string?](https://stackoverflow.com/questions/2464959/whats-the-u-prefix-in-a-python-string/2464968#2464968)

- [What encoding do normal python strings use?](https://stackoverflow.com/questions/3547534/what-encoding-do-normal-python-strings-use)

###  4) r - raw
: 마지막으로, r 은 raw를 의미한다. 이는 escape 문자를 특별하게 다루지 않는다. 

다음의 예제를 보자.
```python
>>> print("hi.\nhello.")
hi.
hello.

>>> print(r"hi.\nhello")
hi.\nhello
```

줄바꿈이 일어나지 않고, 이스케이프 문자가 그대로 표현된 것을 볼 수 있다.

# 3. 비교해보자
: 마지막으로, 각 문자열들을 돌아가면서 간단히 비교해보자.

```python
from itertools import combinations

def compare(dct, x, y):
    return dct[x] == dct[y]
    
string_dict = {
    "_default" : "string",
    "_format" : f"string",
    "_bytes" : b"string",
    "_unicode" : u"string",
    "_raw" : r"string"
}

for c in combinations(string_dict.keys(), 2):
    print(f"compare {c[0]} & {c[1]} -> {compare(string_dict, c[0], c[1])}")
```

결과는 다음과 같다.

```python
compare _default & _format -> True
compare _default & _bytes -> False
compare _default & _unicode -> True
compare _default & _raw -> True
compare _format & _bytes -> False
compare _format & _unicode -> True
compare _format & _raw -> True
compare _bytes & _unicode -> False
compare _bytes & _raw -> False
compare _unicode & _raw -> True
```

bytes를 제외한 나머지는 전부 같은 것을 확인할 수 있다.



raw와 다른 문자열들이 왜 같은지 의구심을 가질 수 있는데, 이는 차후에 str() 과 repr()의 차이를 다루면서 설명하겠다.



\- Reference  
[https://docs.python.org/ko/3/reference/lexical_analysis.html#literals](https://docs.python.org/ko/3/reference/lexical_analysis.html#literals)  
[https://stackoverflow.com/questions/2464959/whats-the-u-prefix-in-a-python-string/2464968#2464968](https://stackoverflow.com/questions/2464959/whats-the-u-prefix-in-a-python-string/2464968#2464968)  
[https://stackoverflow.com/questions/54533637/rstring-bstring-ustring-python-2-3-comparison](https://stackoverflow.com/questions/54533637/rstring-bstring-ustring-python-2-3-comparison)   
[https://stackoverflow.com/questions/2592764/what-does-a-b-prefix-before-a-python-string-mean](https://stackoverflow.com/questions/2592764/what-does-a-b-prefix-before-a-python-string-mean)  
[https://stackoverflow.com/questions/3547534/what-encoding-do-normal-python-strings-use](https://stackoverflow.com/questions/3547534/what-encoding-do-normal-python-strings-use)





