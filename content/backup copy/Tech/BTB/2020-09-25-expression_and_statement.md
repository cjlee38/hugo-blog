---
# author: cjlee
categories: btb
# cover: /assets/covers/coding.png
date: "2020-09-25T00:12:00Z"
tags: ['null']
title: '# Expression과 Statement의 차이 ( feat. Python, Java )'
---

# 0. 들어가며
: 코드를 작성하다보면 한번쯤은, 그런 생각을 해본다.  

<center> "변수명도 동적으로 선언할 수는 없을까?" </center>

물론, Key-Value 를 갖는 Java의 Hashmap, Python의 Dictionary가 있지만,  
때로는 편의성 때문에 변수명 자체가 그 값에 의해서 자동으로 선언될 수 있으면 좋겠다는 생각을 해본다.

이럴때, Python에서는 eval() 혹은 exec() 이라는 내장함수를 이용해볼 수 있는데,  
쓰다보면 expression과 statement 라는 용어가 등장한다.

생각해보니, expression 이라는 단어도, statement라는 단어도, 둘 다 한 번쯤은  
어디선가 들어본 용어인데, 갑자기 두 개의 차이가 있다고 하니 당혹스러울 수 밖에 없다.

오늘은 이 두 용어의 차이에 대해서 알아보고자 한다.

# 1. eval()과 exec()
## eval()

: 먼저, 다음의 코드를 보자.

```python
a = 1
print("eval")
print(eval("a"))
# -> eval
# -> 1
```
eval() 함수 안에, "a"라는 string을 넣었더니 변수 a에 해당하는 값이 튀어나오고,  
print() 함수는 이를 받아서 1이라는 값을 실행하는 모습을 볼 수 있다.

그렇다면, a=2로 바꿀수도 있을까?
```python
a = 1
print("eval")
print(eval("a=2"))
# -> eval
# -> SyntaxError : invalid synatx
```

문법이 잘못 작성되었다고, SyntaxError가 튀어나오는 모습을 볼 수 있다.  
즉, 우리는 print(a=1) 이라고 작성한 셈이다.

## exec()
: 그렇다면, exec()은 어떨까?

```python
a = 1
print("exec")
print(exec("a"))
# -> exec
# -> None
```

보는바와 같이, exec("a") 의 결과로는 아무것도 반환되지 않는다.  
그렇다면, 마찬가지로 a=2 는 어떨까?

```python
a = 1
print("exec")
print(exec("a=2"))
print(a)
# -> exec
# -> None
# -> 2
```

이번에는 다르다. 역시 exec()의 return값은 존재하지 않았지만,  
a가 2로 변경된 것을 볼 수 있다.

## What's the difference?
: 왜 이런 차이가 발생하는 것일까? help()함수를 통해 두 함수를 확인해보자.

---

> **eval()**  
> Help on built-in function eval in module builtins:  
> 
> eval(source, globals=None, locals=None, /)  
>     *Evaluate* the given source in the context of globals and locals.  
>
> The source may be a string representing a Python *expression*  
> or a code object as returned by compile().  
> The globals must be a dictionary and locals can be any mapping,  
> defaulting to the current globals and locals.  
> If only globals is given, locals defaults to it.  

---

> **exec()**
> Help on built-in function exec in module builtins:  
> 
> exec(source, globals=None, locals=None, /)  
> *Execute* the given source in the context of globals and locals.  
> 
> The source may be a string representing one or more Python *statements*  
> or a code object as returned by compile().  
> The globals must be a dictionary and locals can be any mapping,  
> defaulting to the current globals and locals.  
> If only globals is given, locals defaults to it.  

---

두 함수에 대한 설명은 거의 유사하다.  
그런데, eval() 함수는 "Expression"을, exec() 함수는 "statements"를 받는다는 것을 알 수 있다.

다시 위 코드로 돌아가서,   
eval()과 exec()이 의도한 대로 실행된 경우는,   
eval()은 `eval("a")` 를 작성했던 경우고,  
exec()은 `exec("a=2")` 를 작성했던 경우다.

만약 Python에서 단순하게 `a` 라고 작성한다면, print()가 아닌 output에 변수 a의 값이 튀어나오고,  

`a=2` 라고 작성한다면, output은 없지만 a가 2로 변경된다.

따라서, 우리는 "아, `a`와 같이 **output이 있으면 expression**이고,  
`a=2`와 같이 **output이 없는, 값의 변경과 같은 경우라면 statement 구나**" 라고 유추해볼 수 있다.

# 2. Expression & Statement
## Difference between expression and statement
: 이 가설을 검증하기 위해, 구선생님께 여쭤본 결과는 다음과 같다.

> In programming language terminology, an “expression” is a combination of values and functions that are combined and interpreted by the compiler to create a new value, as opposed to a “statement” which is just a standalone unit of execution and doesn’t return anything. One way to think of this is that the purpose of an expression is to create a value (with some possible side-effects), while the sole purpose of a statement is to have side-effects.

짧은 번역 실력으로 해석해보면, 다음과 같다.

> 프로그래밍 언어의 용어 중에서, epxression은 값 또는 함수의 조합으로서, 새로운 값을 만들기 위해 컴파일러에 의해 조합되거나 해석된다. 반면, statemnet는 독립적인 실행의 단위이며, 아무것도 return 하지 않는다. expression은 새로운 값을 만드는 것이 목적이고(side-effect가 있을 수 있다), statement는 side-effect를 갖는 것 자체가 목적이라고 생각해볼 수 있다.

즉, 우리가 변수 a를 그대로 파이썬에 작성해서 넣으면, output으로 변수 a에 해당하는 값이 튀어나온다.  
혹은, `1+2` 라고 작성하면, **3**이라는 값이 튀어나온다.
이런식으로, **"새로운 값"을 만드는 것**이 expression이다.

반대로, a=1 과 같이 작성하면, 아무것도 return되는 것이 없지만, a에는 1이라는 값이 할당된다.  
다시 말해, side-effect가 생긴다.

## Are they exclusive?
: 이렇게보면 expression과 statement는 서로 배타적인 관계라고 생각할 수 있지만,  
사실은 statement가 expression의 superset이다.

아까의 코드로 다시 돌아가서 보면,

`print(eval("a=2"))` 는 에러를 내뱉은 반면,  
`print(exec("a"))` 는 아무것도 return되지 않을지언정, 에러가 발생하지는 않았다.  

즉, 다시 말해서,  
**모든 expression은 statement지만, 모든 statement가 expression은 아니다.** 라는 것을 알 수 있다.

# 3. Usage
: 이렇게까지 알았다면, 우리는 이제 동적으로 변수명을 정할수도 있다.

```python
some_json = {
    "0" : {
        "title" : "some_title",
        "content" : "some_content"
    }, 
    "1" : {
        "title" : "some_second_title",
        "content": "some_second_content"
    }
}

for v in some_json:
    title = some_json[v]['title']
    content = some_json[v]['content']
    exec(f"{title}=content")

print(some_title)
# -> some_content
print(some_second_title)
# -> some_second_content
```

위의 코드에서, exec() 함수 내부를 보면,  
title이라는 변수 안에는 some_title 혹은 some_second_title 이 들어있으므로,  
최종적으로는 **exec("some_title=some_content")** 혹은 **exec("some_second_title=some_second_content")**  가 실행된다.

# **4. 의문점**
: if문을 영어로 검색하게되면, "if statement" 라고 하는데,  
역시 생각해보면, if문의 결과는 어떤 return 값이 주어지지 않으므로, statement 라고 하는 것은 타당하다.

그런데, 지난 [Java Hasmap으로의 여행](https://cjlee38.github.io/java/journey_to_java_hashmap) 편을 읽어보면,  
이러한 작성법을 볼 수 있다. (혹은, C 계열 언어를 자주 쓰는 사람은 이런 작성법도 종종 볼 수 있을 것이다.)

```java
if ((tab = table) == null || (n = tab.length) == 0)
```

tab=table 을 실행하면, 이는 expression이 아닌 statement이므로, null일텐데,  
이를 어떻게 비교할 수 있는 것일까?

Python에서 또한, 다음과 같이 작성하는 것은 허용되지 않는다

```python
if (a=someFunc()) == 1:
    # do something
```

[PEP 572](https://www.python.org/dev/peps/pep-0572/) 에서, python 3.8 이후로,  
**Named Expression** 이라는 이름으로 `:=` 를 이용해서 이런 작성 방법이 허용되지만,  

java에서 위와 같이 사용하는게 허락되는 것이 어떻게 가능한건지 잘 이해가 되지 않는다.  
나중에 여유가 되면 disassemble 이라도 해봐야 하나..



## Reference
- [Expressions vs. statements](https://fsharpforfunandprofit.com/posts/expressions-vs-statements/)
- [코드 단위인 Expression과 Statement의 차이를 알아보자](https://shoark7.github.io/programming/knowledge/expression-vs-statement)