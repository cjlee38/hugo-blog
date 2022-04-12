---
# author: cjlee
categories: java
# cover: /assets/covers/coding.png
date: "2020-09-10T14:54:36Z"
tags: ['null']
title: '# (Java) HashMap 으로의 여행'
---

# 0. 들어가며
: 나는 프로그래밍에서 Naming이 정말 **Critical한 중요성**을 갖는다고 생각한다.  
새로운 무언가에 대해서 배우고 나서, 그 이름을 다시 떠올려보면,  
왜 이걸 만든 사람들이 이 이름을 지었는지에 대해서 가닥이 잡힌다.  

Python에서의 dictionary, 그리고 Java에서의 HashMap은 Key-Value의 구조를 갖는  
데이터를 저장하는 가장 대표적인 클래스다.

Python의 dictionary 라는 이름을 보면,   
"음, 사전이니까, 마치 단어사전 혹은 백과사전처럼, 어떤 이름과 그 안에 내용물을 갖춘 무언가겠군!"  
이라고 유추해 볼 수 있지만,

Java의 HashMap을 보면 잘 이해가 되지 않는다.  
Map이야 Key랑 Value를 Mapping한다는 의미로 쓰인것 같은데,  
"Hash? 내가 아는 그 Hash의 의미로 쓴게 맞나? 왜 Hash라는 이름을 쓴거지?"   
라는 의구심이 든다.  

또한, 흔히 "Key-Value의 시간복잡도는 O(1) 이다" 라고 한다. 왜일까?  
Key라고 하는 것은 결국, Index나 Memory의 위치가 아니라, Object일텐데,  
어떻게 O(1)이 나올 수 있을까?

궁금증을 해결하기 위해,  
세계 모든 지식을 총망라하신 구선생님께 여쭤보았다.  
(공부하면서 작성한 것이니, 틀린 부분이 있다면 말씀해주세요.)

# 1. Hash
: Hash 자체에 대한 내용은 이 포스팅에서 다루지 않는다.   
Hash에 대한 이해가 없으면, 이 글을 읽기가 조금 어려울 수 있다.

(2020.09.25 수정)  
[Hash 맛보기](https://cjlee38.github.io/btb/what_is_hash)   
Hash에 관해서 포스팅했습니다.

# 2. HashMap의 구조
: HashMap에 왜 Hash라는 이름이 붙었는지 이해하기 위해선,  
먼저 그 구조를 파악하는 것이 우선이다. 

![Structure of Hashmap](/assets/images/2020-09-10-15-07-31_journey_to_java_hashmap.md.png){: .alignCenter}

[사진 출처](https://www.javatpoint.com/working-of-hashmap-in-java)

먼저, bucket 이라는 **Array**가 있고,  
그 Array 내부에는 **Node로 구성된 "LinkedList"** 가 존재한다.   

즉, 이렇게 생긴 것을 **"Hash table"** 이라고 하고, Hash table을 통해 Key-Value를 관리하게 된다.  
*기수정렬(Radix Sort) 할 때의 모습을 상상해보면 된다.*  

# 3. HashMap의 동작 (이론편)
: 대충 어떻게 생겨 먹었는지는 알겠으니, 어떻게 동작하는지에 대해서 한번 간략하게 익혀보자.

데이터를 집어 넣는 과정을 이해하면, 꺼내는 것을 이해하는 것은 당연하게 여겨진다.  
HashMap에 put() 함수를 사용해서, 데이터를 넣고자 하면, 다음과 같은 과정이 일어난다.

1. Key로 들어온 Object를 Hash화를 한다
2. Hash화한 값을 **"배열의 인덱스"** 로 지정한다.
3. 해당 배열에 값이 존재하지 않으면, 그 위치에 Node라는 클래스로 Key-Value Object를 저장한다.
4. 해당 배열에 값이 존재하면, 그 **다음** 위치에 새로운 Node를 연결하고 Key-Value Object를 저장한다.(즉, LinkedList로 관리한다.)
5. List가 너무 길게 이어진다면, Tree로 변환한다.
6. Hash충돌(Collision)이 너무 많이 발생한다면, Bucket. 즉 배열의 크기를 늘린다.

이해력이 좋은 사람이라면, 여기까지만 봐도 아하 할 수 있다.  
나같은 멍청이는 뭔소리야? 싶어서 결국 코드를 뜯어보고야 만다.  

# 4. HashMap의 동작 (코드편)
: HashMap 에 데이터를 넣고자 하는, put() 함수로 이동해보자.

```java
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
}
```

보는 바와 같이,   
1) hash()라는 함수를 실행한 key  
2) 원래 key  
3) value  
4) 뭔지 모를 false  
5) 뭔지 모를 true  

가 있다. 

hash() 함수로 슬쩍 발 하나 담궈보자.

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

key가 null이라면 0을,
null이 아니라면 **key의 hashCode() 라는 함수를 실행한 값**과,
**key의 Hash코드를 unsigned로 쉬프트 연산한 값**을 **XOR 연산**한 것을 확인할 수 있다.

이렇게 하면 어떤 값이 나온다는 것을 예측할 수 있다.  
이번 포스팅에서의 목적은 Hash가 어떻게 구현되는지가 아니므로,   
뭐 대충 어떤 숫자가 나왔다고 이해하고 넘어가자.

다시 돌아와서, 이제 putVal() 이라는 곳으로 구경 가보자.

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

꽤 길다. 그리고 복잡하다. 코드를 머릿속으로 Deserialize 하는데 좀 시간이 걸렸다.  
Section을 조금 나눠서 살펴보자.  
```java
if ((tab = table) == null || (n = tab.length) == 0)
    n = (tab = resize()).length;    
```
첫 if문에서 table, 즉 Hash테이블이 null 인지를 물어본다.
즉, **"데이터를 처음 넣는거냐?"** 라고 물어보는 것이다.  
참이라면, resize() 함수를 실행하고, 그 결과를 tab에, 그리고 그 길이를 n에 넣는다.


```java
if ((p = tab[i = (n - 1) & hash]) == null)
    tab[i] = newNode(hash, key, value, null);
```
다음으로, table의 ((n-1) & hash) 위치. 
즉 배열의 특정 index에 위치한 값이 null 이라면,   
다시 말해, 자리가 비어있다면,     
해당 위치에는 새로운 Node를 만들어서 집어넣는다.

```java
else {
    Node<K,V> e; K k;
    if (p.hash == hash &&
        ((k = p.key) == key || (key != null && key.equals(k))))
        e = p;
```
아니라면, 즉 해당 위치에 무언가가(Node) 이미 존재한다면, else문으로 진입하게 된다.

그 이후 등장하는 첫번째 if문은 "hash만 같은게 아니라, key 자체도 같은거야?' 라고 물어본다.  
맞다면, 해당 key를 교체하게 된다.  

```java
else if (p instanceof TreeNode)
    e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
```

그게 아니라면, 우선 해당 LinkedList가 Tree인지를 물어본다.  
Tree라면, Tree에 맞게 값을 추가한다.

```java
else {
    for (int binCount = 0; ; ++binCount) {
        if ((e = p.next) == null) {
            p.next = newNode(hash, key, value, null);
            if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                treeifyBin(tab, hash);
            break;
        }
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            break;
        p = e;
    }
}
```

트리가 아니라면, LinkedList란 뜻이다. 그렇다면 해당 LinkedList를 돌면서,  
"Key까지 같으면, replace" 해주고, 처음 들어오는 놈이라면, 맨 끝에 이어붙이게 된다.

그 과정속에서, bin. 즉 LinkedList의 길이가 너무 길다면, 이를 Tree로 바꿔준다.

```java
if (e != null) { // existing mapping for key
    V oldValue = e.value;
    if (!onlyIfAbsent || oldValue == null)
        e.value = value;
    afterNodeAccess(e);
    return oldValue;
}
```
여기서 e는 우리가 새로이 추가하고자 하는 Node를 가리킨다.  
else문으로 분기한 이후, 우리가 지금까지 한 것은, Node를 추가할 위치를 찾는 것이었다.  
이제 값을 넣어야 한다. 해당 코드가 Key 안에 Value를 넣는것이다.  

onltIfAbsent는 "키가 이미 있으면, 안넣겠다"를 의미한다.
즉, 우리는 false를 넣어줬고, ! 가 붙었으므로 true이므로, 참이 되니 값을 교체할 수 있다.

```java
++modCount;
if (++size > threshold)
    resize();
afterNodeInsertion(evict);
return null;
```
마지막으로, 수정된 횟수를 ++시키고,  
HashTable의 길이 또한 ++ 시킨 뒤, 일정 수준을 넘어서면 resize() 하게 된다.

afterNodeAccess와 afterNodeInsertion은 LinkedHashMap에서 Override 되는 내용이므로 Pass한다.

resize() 코드 또한 꽤 복잡하므로, 주석만 따왔다. 여기서는 다음과 같이 설명한다.

> Initializes or doubles table size.  If null, allocates in
> accord with initial capacity target held in field threshold.
> Otherwise, because we are using power-of-two expansion, the
> elements from each bin must either stay at same index, or move
> with a power of two offset in the new table.

즉, bucket을 초기화 하던지, 충돌이 너무 많이 일어나면 두배로 늘리겠다는 얘기다.

이렇게 흐름을 읽고 나면, 왜 HashMap이 O(1)에 가까운 접근속도를 보이는지 알 수 있다.  

기본적으로, 데이터의 구조가 배열이므로 O(1)의 접근속도를 가지고,  "해쉬"라는 것은 결국 일정 길이의 숫자를 return 하므로,  
"해쉬값을 통해 index 접근"을 진행하면, 어떤 Object 이던 O(1)으로 접근할 수 있다.

그런데, 충돌이 아예 안일어난다면 좋겠지만, 이를 위해 배열의 길이를 너무 길게 만들면 공간의 낭비고,
그렇다고 적게 만들자니, 충돌이 일어날 때 문제가 생긴다. 따라서 충돌이 일어나면 LinkedList로 연결지어준다.

이 LinkedList는 특정 길이를 넘어서면 Tree 형태를 갖게 되고,  
또한 충돌 자체도 많이 일어나면 배열을 늘리므로,  
이 LinkedList 혹은 Tree는 제한된 크기를 갖게 된다.

# 5. 마치며
: 기존에 가졌던 궁금증이 모두 해소되었다.  
Hash를 왜 사용했는지, 그리고 왜 시간복잡도가 이렇게 빠른 것인지 알 수 있었다.   
그런데, 또 한가지 궁금증이 발생한다.  
Python은 3.7 이후 Dictionary는 Ordered, 즉 순서를 갖는 Dictionary를 Default로 사용한다고 한다.  

얘는 또 어떻게 구현되었을지에 대해 알아보고 싶지만, 일단 여기에서 멈추자.


**Reference**
- [Java HashMap은 어떻게 동작하는가?](https://d2.naver.com/helloworld/831311)  
- [Hash Table은 프로그래머의 기본기](https://www.youtube.com/watch?v=S7vni1hdsZE&t=297s&ab_channel=%ED%8F%AC%ED%94%84TV)  
- [Java HashMap동작 원리](https://backdoosaan.tistory.com/13)  