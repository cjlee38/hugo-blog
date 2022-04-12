---
# author: cjlee
categories: java
# cover: /assets/covers/coding.png
date: "2020-10-18T14:22:00Z"
tags: ['null']
title: '# (Java) 효율적인 반복문 탈출을 위한 Label'
---

# 0. 들어가며
: 혹시, 여러 개로 겹쳐있는 for문을 반복하다가, 어느 조건을 만족했을 때,  
**전체 반복문을 중단시키려면 어떻게 해야 할까?**에 대해서 고민해본 적이 있는가?

아마 지금까지는 두가지 방법 중 하나를 사용했을 것 같다.

A 방법
```java
boolean flag = false;
for (int i = 0; i < 10; i++) {
    for (int j = 0; j < 10; j++) {
        if ( /*some_condition*/ ) {
            flag = true;
            break;
        } 
    }
    if (flag) {
        break;
    }
}
```

B 방법
```java
for (int i = 0; i < 10; i++) {
    for (int j = 0; j < 10; j++) {
        if ( /*some_condition*/ ) return;
    }
}
```

아마 후자의 방식을 많이 사용했을 것 같은데,  
만약 return을 하기 전에 추가적으로 해줘야 하는 어떤 작업이 있었다면,  
코드가 약간 더러워보이더라도 전자의 방식을 어쩔 수 없이 사용했을 것이다.

그러나, label을 이용하면, 아주 간단하게 해결할 수 있다.

# 1. 형태
: for-loop, 혹은 while-loop 앞에, 이름 그대로 **"Label"** 을 붙이는 것을 의미한다.

다음과 같이 사용할 수 있다.

```java
foo: 
for(int i = 0; i < 10; i++) {
    // do something
    if (i == 5) break foo;
}
```

결과
```
0
1
2
3
4
```

별 거 없어보이고, 기존 for-loop 랑 별반 차이도 없어보이지만,  
label의 힘은 아까와 같이, 반복문이 **겹쳐 있을 때** 비로소 발휘된다.

```java
foo:
for (int i = 0; i < 10; i++) {
    bar:
    for (int j = 0; j < 10; j++) {
        System.out.println(i + " " + j);
        if (j == 6) break foo;
    }
}
```

결과
```
0 0
0 1
0 2
0 3
0 4
0 5
0 6
```
break는 한번만 사용되었는데, 가장 바깥쪽 `foo` 라고 labeling 된 loop가 중단되었다.

처음에는 보고 '이거 코틀린 코드인가?' 싶어 눈을 비벼보아도, 자바코드였다.   
나중에 공부를 하고 나서야 label 이라는 것을 알게 되었다.

# 2. 써도 될까?
: 언뜻 보면, C 계열 언어에서 거의 **죄악시** 하는 goto 문이랑 비슷해서,  
쓰면 안될것 같기도 한데, 찾아보니 써도 괜찮다고 한다.

말인 즉슨, "Control Flow를 **전이 시키는 것**이 아니므로, 괜찮다" 고 한다.
자세한 내용은 [여기](https://stackoverflow.com/questions/14960419/is-using-a-labeled-break-a-good-practice-in-java) 를 참고하자.

# 3. How about Python ?

여담으로, Python에서는 이러한 기능이 없는데,  
파이썬의 창시자인 귀도 반 로섬이 [이러한 이유](https://mail.python.org/pipermail/python-3000/2007-July/008663.html)로 거절했다고 한다.

대신, 예외처리를 발생시키는 `raise StopIteration` 을 응용해 활용할 수 있다.


# 4. 의문점
: label **statement** 라면, if-statement에도 붙을 수 있다는 뜻이고,  
실제로 코드를 작성해보아도 어떠한 에러도 나지 않고, 결과에도 영향이 없다.

그런데, 이를 어떻게 응용할 수 있을지에 대해서는 도저히 떠오르지 않는다.  
이 부분에 대해서는 조금 더 공부를 해봐야 알 것 같다.

## Reference
- [Labeled Statements in Java](https://howtodoinjava.com/java/flow-control/labeled-statements-in-java/)
- [Naming Loops in Python - StackOverflow](https://stackoverflow.com/questions/8419796/naming-loops-in-python)
- [Is using a labeled break a good practice in Java?](https://stackoverflow.com/questions/14960419/is-using-a-labeled-break-a-good-practice-in-java)