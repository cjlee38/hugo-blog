---
# author: cjlee
categories: java
# cover: /assets/covers/coding.png
date: "2020-10-07T01:03:00Z"
tags: ['null']
title: '# (Java) Stream API 첫걸음'
---

# 0. 들어가며
: 최근, 여러 알고리즘 문제들을 풀면서,  
가끔 심심할때마다 프로그래머스 Lv.1 문제들도 푸는데,  

Lv.1 문제들은 단순하게 제출하는 것이 뭔가 좀 껄끄러웠다.   
개인적으로는, 1차원의 for loop를 쓰더라도 이게 여러 개 있으면 꽤 마음이 불편해진다.   
뭔가, 깔끔하고 멋있게 써야 할 것 같았다.

개인적으로, 이러한 상황에 가장 베스트인 선택지는  
바로 Stream API 라고 생각한다.

아직 익숙치 않은 Stream API를 공부해보자.  
양이 꽤 많기 때문에 아마 모두 적기에는 어려울 것 같다.

---

> Note. Stream API는 일반적으로 컴퓨터 공학에서 사용하는 용어인 Stream 과는 다르다.  
> 본 포스팅은 Stream API(이하, Stream) 에 대해서 다룬다.

---

# 1. Stream 이란?
: Oracle에서 [Stream package의 document](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html#package.description)에서는 다음과 같이 설명하고 있다.

<Center> Classes to support functional-style operations on streams of elements, such as map-reduce transformations on collections. </Center>

요소들의 stream에 대해서, (컬렉션의 map-reduce 변환과 같은) 함수형 연산을 지원하는 클래스 ... ?  
무슨 말인지 이해하기 어렵다.

[이전 HashMap 편](https://cjlee38.github.io/java/journey_to_java_hashmap)에서도 이야기했듯이 나는 **Naming**이 중요하다고 생각하기 때문에,  
가장 먼저 네이버 사전에 stream을 검색한 결과는 다음과 같다.

**stream**

1.	개울, 시내
2.	(액체기체의) 줄기
3.	줄줄 흐르다

이걸 보고나니, 영화 전우치에서 전우치가 화담과 마지막 전투를 할 때에  
벽에서 물을 끌어다가 화담에게 쏘는 장면이 생각났다.

아무튼, **"어떠한 흐름"** 을 다루는 것이 stream 이라고 생각하면 될 것같다.  
(이 부분은 컴퓨터공학의 Stream과 비슷한 맥락을 갖는다.)

이 흐름은 그대로 흘러가는구나.. 하는 것이 아니라,  
우리가 주도적으로 이리저리 다룰 수 있어야 한다.

**수도꼭지를 돌렸을 때** 물이 어떻게 나오는지 상상해보면 이해하기 쉽다.  
수도꼭지는 수도관과 연결되어 있을테고,  
수도관의 물은 단순히 전달되는것이 아니라, 그 과정에서 filtering도 하고, 정화도 하고..  
등등의 여러가지 절차가 있을 것이다. 

<img src="/assets/images/2020-10-10-23-58-33_2020-10-07-java_stream.md.png" style="width: 300px; height: 300px" >
<!-- ![](/assets/images/2020-10-10-23-58-33_2020-10-07-java_stream.md.png) -->

마찬가지로, 우리가 원하는 데이터를 얻기 위해,  
Stream을 이용해서 **요리조리 흐름을 제어하고, 필터링을 하는 등의 작업**을 거치면 된다.


## 간단한 예시 ( 1 )
: Stream을 썼을 때의 장점이라면, **가독성**이 좋아진다는 것이다.   
숫자가 1부터 10까지 있는 array에서, **짝수인 숫자**만 골라내고 싶다면 어떻게 해야할까?  

아마 stream을 쓰지 않는다면, 이렇게 작성할 수 있을 것이다.
```java
int[] array = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// 짝수를 구하는 부분
int[] evens = new int[array.length/2];
for (int i=0; i<array.length; i++) {
    if (array[i] % 2 == 0)  {
        evens[i/2] = array[i];
    }
}
// 짝수를 구하는 부분

for(int i=0; i<evens.length; i++) {
    System.out.print(evens[i] + " ");
} // 2 4 6 8 10 
```
그러나, 이 방법은 꽤 우아해보이진 않는다.  
만약, array가 **랜덤**인 정수라면 어떨까?  
조금 더 극단적으로, array가 **짝수로만** 이루어져있다면 어떨까?  

물론, ArrayList 나 LinkedList 같이 편리한 Collection을 사용하는 방법도 있지만,  
**근본적인 해결책**을 제시했다고 보기는 어렵다.

**stream을 이용한다면 어떨까?**

```java
int[] evens = Arrays.stream(array)
        .filter(x -> x % 2 == 0)
        .toArray();
```

...???  
이게 전부다.

array를 stream으로 감싸서 Instream 이라는 객체로 만든뒤에,  
x % 2 == 0, 즉 짝수인 애들만 남기고,  
이를 다시 Array로 만들었다.

이렇게 작성하면, array에 무슨 값이 들어와도 두렵지 않다.

```java
int[] array = {2, 4, 1, 3, 4, 4, 4};
int[] evens = Arrays.stream(array)
                .filter(x -> x % 2 == 0)
                .toArray();
// 2 4 4 4 4
```

하나만 더 보자.

## 간단한 예시 ( 2 )
: 프로그래머스에서 풀었던 문제 중 하나다.  
[자연수 뒤집어 배열로 만들기](https://programmers.co.kr/learn/courses/30/lessons/12932) 라는 문제인데,  
숫자형 값이 들어왔을 때, 이를 뒤집은 뒤, 배열로 만들면 된다.

가령, 12345가 들어오면, [5, 4, 3, 2, 1] 로 만들어서 돌려주면 된다.

stream을 쓰지 않는다면, 이렇게 썼을 것 같다.  
(일부러 조금 귀찮게 썼다.)
```java
StringBuilder sb = new StringBuilder(String.valueOf(n)).reverse();
String[] strArray = sb.toString().split("");

int[] answer = new int[strArray.length];
for (int i=0; i<strArray.length; i++) {
    answer[i] = Integer.parseInt(strArray[i]);
}
return answer;
```

StringBuilder로 만들어서 뒤집고, 이를 빈 문자열로 split한다.  
--> ["5", "4", "3", "2", "1"]

그리고, int형의 빈 배열을 만든 뒤 하나씩 int형으로 parsing해서 집어넣고, 돌려준다.

꽤 괜찮아보인다.  
stream을 쓰면 어떨까?
```java
return new StringBuilder(String.valueOf(n))
        .reverse()
        .toString()
        .chars() // 여기서 Intstream이 생성된다
        .map(x -> Character.getNumericValue(x)) 
        .toArray();
// chars()를 만들면 각 char 값이 아스키코드값이 되기 때문에 
// 이를 다시 원래 숫자값으로 해석해줘야 한다.
```

같은 기능을 하는 코드지만, 좀 더 간결해졌다.  
이런 코드를 보고있으면 왠지모르게 심적으로 편안해진다.

# 2. 사용법
: stream은 **대개** 다음과 같은 절차를 거친다.

1. 생성
2. 가공
3. 소비

위 3단계 절차에 대해서, 아주 **"간략하게"** 다룰 것이며,    
자세한 사용법을 모두 서술하는 것은 포스팅이 너무 길어지므로   
다른 분들의 포스팅을 참고하길 바란다.

## 1) 생성
### 배열
: 배열은 클래스가 아니므로 Arrays라는 클래스를 통해 스트림으로 만들어줘야 한다.

```java
int[] intArr = new int[10];
IntStream stream = Arrays.stream(array);

String[] strArr = {"hello", "world"};
Stream<String> strStream = Arrays.stream(strArr);
```

### Collection
: Collection (e.g. List, stream)의 경우,   
Collection interface에 추가된 method인 .stream()을 통해 만들 수 있다.

```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3));
Stream<Integer> stream = list.stream();
```

### 자체 생성
: Stream.of() method를 이용하는 방법도 있다.  
이는 입력받은 값을 stream으로 변환해주는 기능을 한다.

```java
Stream<Integer> stream = Stream.of(1, 2, 3);
```

## 2) 가공
: 다음과 같은 String의 stream을 기준으로 가공하는 예시를 들어보겠다.

```java
List<String> list = Arrays.asList("hello", "world", "java", "stream");
```

이 떄 주의해야할 것은,  
첫 째로, **"각각의 요소"**에 접근해서 사용한다는 것을,  
둘 째로, 가공하는 method가 **return**하는 객체는 **또 다시 stream**이라는 것을   
기억해야 한다.

### .filter()
: 이름 그대로, 조건에 부합하는 요소들만 남기는 작업을 한다.

```java
Stream<String> stream = list.stream()
                            .filter(x -> x.contains("o"));
// {"hello", "world"}
```

"o" 라는 문자열이 포함된 요소만 남기겠다는 의미이므로,  
o를 포함하고 있는 hello, world 가 남게 된다.

이때, x 라는 녀석은 임의로 붙이는 **"임시변수"** 이다.  
파이썬의 for loop, java의 for-each 에서 각 요소를 접근하는 법을 생각하면 된다.

### .map()
: map 또한 이름과 마찬가지로, 각각의 요소를 mapping 하는 역할을 한다.

```java
Stream<String> stream = list.stream()
                            .map(x -> x.substring(0, 2));
// {"he", "wo", "ja", "st"}
```

각각의 요소에 접근하면서, 
x 라고 붙인 임시변수를 substring, 즉 index 기준으로 잘라내겠다는 의미가 된다.

## .sorted()
: 정렬을 하는 기능을 해준다.  
내부 객체(요소)가 스스로 Compartor를 갖고 있다면 이에 기반해서 default 정렬을 할 것이고,  
없거나, 커스텀 정렬을 할 경우 Compartor를 넘겨줄 수도 있다.

```java
Stream<String> stream = list.stream().sorted();
// {"hello", "java", "stream", "world"}
Stream<String> stream = list.stream().sorted(Collections.reverseOrder());
// {"world", "stream", "java", "hello"}
```
## .peek()
: 자료구조에서 사용하는 peek()과 비슷하다.  
stream을 소비하지 않고, 각각의 요소를 한번 "확인" 하는 정도의 역할만 하고,  
return 되는 **결과에는 영향을 미치지 않는다.**

```java
Stream<String> stream = list.stream().peek(x -> x.substring(0,2));
// 아무런 영향을 미치지 않으므로, {"hello", "world", "java", "stream"}을 유지

Stream<String> stream = list.stream().peek(x -> System.out.println(x));
// hello, world, java, steram 순서대로 출력
```

## 3) 소비
### .min(), .max(), .sum()
숫자형일 때 최소, 최대, 합의 값을 구해주는 연산이다.  

*String은 계산이 불가능하므로 잠깐 다른 예시를 활용한다.*

```java
int[] arr = {1, 2, 3, 4};
OptionalInt oMax = Arrays.stream(arr).max(); // OptionalInt[4]
OptionalInt oMin = Arrays.stream(arr).min(); // OptionalInt[1]
int sum = Arrays.stream(arr).sum(); // 10

```

이 때, max()와 min()으로 나온 값은 Optional 객체이므로,  
이를 getAsInt()를 뒤에 붙여줘야 한다.

```java
int max = oMax.getAsInt();
int min = oMin.getAsInt();
```

### .toArray()
: stream을 배열로 변환해주는 작업이다.

```java
String[] strArr = list.stream()
                    .map(x -> x.substring(0,2))
                    .toArray(x -> new String[x]);
// {"he", "wo", "ja", "st"}
```

parameter를 넘기지 않으면 Object 객체의 배열로 넘어가게 된다.

### .collect()
: Collection 객체로 모으는 역할을 한다.  
이 때, Collector 객체를 파라미터로 넘겨줘야 한다.

```java
Set<String> set = list.stream()
                        .map(x -> x.substring(0,2))
                        .collect(Collectors.toSet());
// Set = {"he", "wo", "ja", "st"}
```



### .forEach()
: 각 요소에 대해서, "어떤 작업"을 하고 그대로 실행하는 Consumer를 받는다.  
대표적으로, System.out.println() 이 있다.

```java
list.stream()
    .map(x -> x.substring(0,2))
    .forEach(x -> System.out.println(x));
// 순서대로, he, wo, ja, st 가 출력
```

# 3. 주의점
: 이렇듯 막강한 stream에도, 주의해야할 점이 있다.

1. 너무 **복잡한 경우**, stream 보다 그냥 **일반적인 for-loop를** 쓰는 것이 낫다.
2. stream은 한번 **"소비"**하면, 그대로 사라진다.

2번의 경우, [여기](https://hamait.tistory.com/547)를 참고하자.

앞서도 언급했듯이 위 예시 이외에도 여러 메소드들이 있고,   
응용에 따라 사용법은 무궁무진해지므로,  
이를 **적절히** 활용하면 정말 편리함을 느낄 수 있을 것이다.

## Reference
- [JAVA 8 스트림 튜토리얼](https://wraithkim.wordpress.com/2017/04/13/java-8-%EC%8A%A4%ED%8A%B8%EB%A6%BC-%ED%8A%9C%ED%86%A0%EB%A6%AC%EC%96%BC/)
- [Oracle - Package java.util.stream](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html)
- [for-loop 를 Stream.forEach() 로 바꾸지 말아야 할 3가지 이유](https://homoefficio.github.io/2016/06/26/for-loop-%EB%A5%BC-Stream-forEach-%EB%A1%9C-%EB%B0%94%EA%BE%B8%EC%A7%80-%EB%A7%90%EC%95%84%EC%95%BC-%ED%95%A0-3%EA%B0%80%EC%A7%80-%EC%9D%B4%EC%9C%A0/)
- [자바8 Streams API 를 다룰때 실수하기 쉬운것 10가지](https://hamait.tistory.com/547)