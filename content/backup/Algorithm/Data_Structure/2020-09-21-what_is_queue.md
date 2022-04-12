---
# author: cjlee
categories: data-structure
# cover: /assets/covers/data-structure.jpg
date: "2020-09-21T19:38:00Z"
tags: ['null']
title: '# 마음으로 이해하는 자료구조 : Queue ( feat. C )'
---

# 0. 들어가며
: Stack과 Queue는 아주 기본 중의 기본인 자료구조이다.  
아주 간단하므로, 사실 Stack과 Queue가 무엇인지에 대해서 이야기할 부분은 많지 않다.

따라서, 이번 포스팅은 코드 구현 위주로 작성할 것이다.  
쓰다보니, 생각보다 포스팅이 또 길어져서, Stack과 Queue를 나누어서 작성했다.  
[지난 Stack편](http://cjlee38.github.io/data_structure/what_is_stack) 에 이어집니다.

# 1. 이론 편
: Queue는, 편의점의 냉장고를 생각하면 된다.  
편의점 알바를 해보면, 우리가 보고 있는 냉장고 뒤편은 냉장창고라는 것을 알 수 있다.  
창고 뒤편에서, 냉장고에 음료수를 넣으면, 반대편에선 가장 먼저 넣은 것을 우선으로 꺼낼 수 있다.

즉, A,B,C,D 를 넣으면, 편의점 손님이 이것을 꺼낼 때도 A,B,C,D 순서로 꺼낸다.

이를, **FIFO(First in, First out)(선입선출)** 라고 한다.

# 2. 구현 편 - ADT
: Queue의 ADT는 다음과 같다.

- init()
  * Queue의 초기화를 진행
  * Queue 생성 후, 가장 먼저 호출되어야 하는 함수
- size()
  * 해당 Queue에 데이터가 몇 개 쌓여 있는지 확인.
- isEmpty()
  * 해당 Queue가 비어 있는가를 확인.
  * 비었으면 true, 아니면 false를 반환.
- enqueue(data)
  * Queue에 데이터를 저장.
  * 당연히, 가장 마지막에 저장한다.
- dequeue()
  * Queue에 있는 데이터를 반환
  * 꺼낸 데이터는 **Queue에서 삭제**한다.
  * 역시 당연히, **가장 오래된 데이터**를 꺼낸다.
  * **Queue에 데이터가 최소 하나는 있어야 한다.**
- peek()
  * Queue에 있는 데이터를 반환.
  * 꺼낸 데이터는 **삭제하지 않는다.**
  * 물론 역시 당연히, **가장 오래된 데이터**를 꺼낸다.
  * **Queue에 데이터가 최소 하나는 있어야 한다.**

# 3. 구현 편 - Code
: 역시, Queue 또한, LinkedList로 구현한다.

___

> Note. 구현에 앞서  
> Stack과 달리, Queue에서 앞, 뒤를 구분하는 것은 헷갈린다.   
> Front와 Rear로 표현하는 사람도 있고, Head와 Tail로 표현하는 사람도 있다.  
> 우리가 구현하는 큐는, 다음과 같은 그림을 갖고 있다고 생각하고 보자.  

![Queue](/assets/images/2020-09-21-21-16-35_2020-09-21-what_is_Queue.md.png){: .alignCenter}

___

Stack과 마찬가지로, 구조체부터 만들자.  

```c++
typedef struct Queue {
    int size;
    struct Node* head;
    struct Node* tail;
}Queue;

typedef struct Node {
    int data;
    struct Node* prev;
}Node;
```
지난 Stack 구조체에 비해서, 추가 된 것은 tail 하나다.  
Stack에서는 최상단의 대상만 삭제하고, 추가했지만,  
Queue에서는 추가할 때는 Tail, 삭제할 때는 Head를 이용할 것이므로,  
Head와 Tail이 둘다 필요하다.

다음으로, Init() 함수는 다음과 같다.

```c++
void QInit(Queue *pqueue) {
    pqueue->size = 0;
    pqueue->head = NULL;
    pqueue->tail = NULL;
}
```

역시, size는 0으로, head와 tail은 모두 NULL로 우선 초기화 해준다.

```c++
int QSize(Queue *pqueue) {
    return pqueue->size;
}

int QIsEmpty(Queue *pqueue) {
    if (pqueue->size == 0) { // if (Qsize(pqueue) == 0) 으로 대체해도 무방.
        return 1;
    } else {
        return 0;
    }
}

```

QSize()와 QIsEmpty() 또한 간단하다.  
IsEmpty() 그대로, 비어 있는지를 묻고 있으므로,  
비어 있다면, 즉 size가 0 이라면 참인 1을,  
비어 있지 않다면 거짓인 0을 return 하면 된다.

다음으로, 핵심이 되는 Enqueue()와 Dequeue() 작업이다.

```c++
void QEnqueue(Queue *pqueue, int data) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->data = data;
    newNode->prev = NULL;
    
    if(QIsEmpty(pqueue)) {
        pqueue->head = newNode;
    } else {
        pqueue->tail->prev = newNode; // 그림 1번
    }
    pqueue->tail = newNode; // 그림 2번
    pqueue->size++;
}
```
![queue_enqueue](/assets/images/2020-09-21-21-42-21_2020-09-21-what_is_Queue.md.png){: .alignCenter}

만약, 처음으로 데이터를 넣는 것이라면, pqueue의 head와 tail은 둘다 newNode를 가리킨다.  
그러나, 두 번째부터는, 조금 다르다. 

**원래 tail이 가리키고 있던 Node가 갖고있는 prev 포인터**를, 새로운 Node인 **newNode를 가리키게 한다.**(그림의 1번)  
그리고, **Tail은 새로운 Node인 newNode를 가리킨다.** (그림의 2번)  

그리고, pqueue가 갖고 있는 size라는 속성을 하나 증가시킨다.

```c++
int QDequeue(Queue *pqueue) {
    int data = pqueue->head->data;
    
    Node* delNode = pqueue->head;
    pqueue->head = pqueue->head->prev;
    free(delNode);
    
    pqueue->size--;
    
    return data;
}
```

![queue_dequeue](/assets/images/2020-09-21-21-44-13_2020-09-21-what_is_Queue.md.png){: .alignCenter}

삭제를 하는 경우는, stack과 같다.

가장 먼저, pqueue->head 의 Node가 갖고 있는 data를 int data에 저장한다.

그리고, **delNode가 pqueue의 head를 가리키게 하고**(그림의 1번)  
**pqueue head는 하나 이전의 요소(prev)를 가리키게 한다.**(그림의 2번)  
마지막으로, free(delNode)를 통해, delNode가 가리키는 요소를 삭제하게 하면 된다.

이후, pstack의 data를 하나 감소시키고, data를 return 해주면 된다.

마지막으로, peek() 또한 stack과 같다
```c++
int QPeek(Queue *pqueue) {
    return pqueue->head->data;
}
```

역시, 잘 돌아가는지 한번 테스트 해보자.  

```c++
int main() {
    Queue queue;
    QInit(&queue);
    
    QEnqueue(&queue, 3);
    QEnqueue(&queue, 5);
    QEnqueue(&queue, 4);
    QEnqueue(&queue, 1);
    
    printf("Q size : %d\n", QSize(&queue));
    // Q size : 4
    printf("Q peek : %d\n", QPeek(&queue));
    // Q peek : 3
    
    while(!QIsEmpty(&queue)) {
        printf("Dequeue : %d\n", QDequeue(&queue));
    }
    // Dequeue : 3
    // Dequeue : 5
    // Dequeue : 4
    // Dequeue : 1

    return 0;
}

```

잘 동작하는 모습을 볼 수 있다. 
역시, 전체 코드는 최하단에 작성하였다.

# 4. 응용 편 - 다리를 지나는 트럭
: [다리를 지나는 트럭 문제](https://programmers.co.kr/learn/courses/30/lessons/42583)는 java 로 한번 풀었으나,  
C로도 한번 풀어보자.(프로그래머스에서 C를 지원하지 않으므로, 제시된 테스트 케이스만 통과하면 맞았다고 보자.)

이를 위해선, **트럭이 다리에 얼마나 올라와 있었는가?** 를 구분하기 위해,  
Truck 구조체를 하나 만들어야 한다.

```c++
typedef struct Truck {
    int weight;
    int time;
}Truck;
```
또한, 이 Truck 을 Queue에 담을 것이므로,  
int 대신 Truck 구조체를 담을 수 있도록 적절히 바꿔주자.
```c++
typedef struct Node {
    Truck truck;
    struct Node* prev;
}Node;
```

당연히, 다른 관련 함수들 또한 Truck으로 적절히 바꿔주어야 한다(e.g. QDequeue())

main() 함수는 다음과 같다. 

```c++
int main() {
    Queue waitQ;
    QInit(&waitQ);
    
    Queue bridgeQ;
    QInit(&bridgeQ);
    
    int bridge_length = 100;
    int weight = 100;
    
    int truck_weights[] = {10, 10, 10, 10, 10, 10, 10, 10, 10, 10};
    int array_length = sizeof(truck_weights) / sizeof(int);
    
    int answer = solution(&waitQ, &bridgeQ, bridge_length, weight, truck_weights, array_length);
    printf("answer : %d\n", answer);

    return 0;
}
```

**"진입을 대기하는 waitQ"**와, **"다리에 해당하는 bridgeQ"** 를 만들어주고,  
문제에서 주어지는 값들과 함께 solution() 함수로 넘겨주었다.

```c++
int solution(Queue *waitQ, Queue *bridgeQ, int bridge_length, int weight, int truck_weights[], int array_length) {
    int flag = 0; // 다리를 나가야 하는 트럭이 존재하는지 확인하는 flag
    int bridge = 0; // 현재 다리에 올라가 있는 트럭 무게의 합
    int answer = 0;
    
    // 처음엔 모두 Truck 구조체로 만들어서 waitQ에 넣어준다.
    for(int i=0; i<array_length; i++) {
        Truck truck;
        truck.weight = truck_weights[i];
        truck.time = bridge_length;
        QEnqueue(waitQ, truck);
    }
    
    // waitQ와 bridgeQ가 모두 비어있다면, 모든 트럭이 통과한 것이므로 중단한다.
    while(!QIsEmpty(waitQ) || !QIsEmpty(bridgeQ)) {
        // bridgeQ를 돌면서, 다리 위에 올라가 있는 시간인 time을 1씩 감소시킨다.
        // time이 0 이하 인 경우, 즉 다리를 나가야 하는 트럭이 존재하는 경우, flag는 1을 추가시킨다.
        Node* p = bridgeQ->head;
        while(p != NULL) {
            p->truck.time--;
            if (p->truck.time <= 0) {
                flag = 1;
            }
            p = p->prev;
        }
        
        // flag가 존재하는 경우, 무조건 가장 앞에 있는 트럭이 나가야 하므로,
        // 트럭을 하나 빼내고, 현재 다리에 올라가 있는 트럭 무게의 합인 bridge를, 빠져나가는 트럭 무게 만큼 감소시킨다.
        if (flag) {
            Truck outTruck = QDequeue(bridgeQ);
            bridge -= outTruck.weight;
            flag = 0;
        }
        
        // waitQ가 비어있지 않다면, 들어갈 수 있는지를 확인해보고, 
        // 들어갈 수 있다면, waitQ에서 빼서 bridgeQ에 추가한다.
        // 역시, 현재 다리에 올라가 있는 트럭 무게의 합인 bridge를, 들어가는 트럭 무게 만큼 증가시킨다.
        if (!QIsEmpty(waitQ)) {
            Truck inTruck = QPeek(waitQ);
            if (bridge + inTruck.weight <= weight) {
                QEnqueue(bridgeQ, QDequeue(waitQ));
                bridge += inTruck.weight;
            }
        }
        
        answer++;
    } // end of while
    
    
    return answer;
}
```

역시, 코드에 대한 설명은 주석으로 달아놓았다.

# 마치며.
: C 자체로는, 생산성이 많이 떨어짐을 느꼈다. 한 번 풀었던 문제인데도,  
이렇게 시간이 오래 걸릴줄은 몰랐다 ㅠ.  
역시 나는 Java가 좋다.


# 구현 코드들.
**queueImpl.c**
```c++
#include <stdio.h>
#include <stdlib.h>

typedef struct Queue {
    int size;
    struct Node* head;
    struct Node* tail;
}Queue;

typedef struct Node {
    int data;
    struct Node* prev;
}Node;

void QInit(Queue *pqueue);
int QSize(Queue *pqueue);
int QIsEmpty(Queue *pqueue);
void QEnqueue(Queue *pqueue, int data);
int QDequeue(Queue *pqueue);
int QPeek(Queue *pqueue);


int main() {
    Queue queue;
    QInit(&queue);
    
    QEnqueue(&queue, 3);
    QEnqueue(&queue, 5);
    QEnqueue(&queue, 4);
    QEnqueue(&queue, 1);
    
    printf("Q size : %d\n", QSize(&queue));
    printf("Q peek : %d\n", QPeek(&queue));
    
    while(!QIsEmpty(&queue)) {
        printf("Dequeue : %d\n", QDequeue(&queue));
    }

    return 0;
}



void QInit(Queue *pqueue) {
    pqueue->size = 0;
    pqueue->head = NULL;
    pqueue->tail = NULL;
}

int QSize(Queue *pqueue) {
    return pqueue->size;
}

int QIsEmpty(Queue *pqueue) {
    if (pqueue->size == 0) {
        return 1;
    } else {
        return 0;
    }
}

void QEnqueue(Queue *pqueue, int data) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->data = data;
    newNode->prev = NULL;
    
    
    if(QIsEmpty(pqueue)) {
        pqueue->head = newNode;
    } else {
        pqueue->tail->prev = newNode;
    }
    pqueue->tail = newNode;
    pqueue->size++;
}

int QDequeue(Queue *pqueue) {
    int data = pqueue->head->data;
    
    Node* delNode = pqueue->head;
    pqueue->head = pqueue->head->prev;
    free(delNode);
    
    pqueue->size--;
    
    return data;
}

int QPeek(Queue *pqueue) {
    return pqueue->head->data;
}
```

**Trucks.c**
```c++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct Truck {
    int weight;
    int time;
}Truck;

typedef struct Queue {
    int size;
    struct Node* head;
    struct Node* tail;
}Queue;

typedef struct Node {
    Truck truck;
    struct Node* prev;
}Node;



void QInit(Queue *pqueue);
int QSize(Queue *pqueue);
int QIsEmpty(Queue *pqueue);
void QEnqueue(Queue *pqueue, Truck truck);
Truck QDequeue(Queue *pqueue);
Truck QPeek(Queue *pqueue);
int solution(Queue *waitQ, Queue *bridgeQ, int bridge_length, int weight, int truck_weights[], int array_length);

int main() {
    Queue waitQ;
    QInit(&waitQ);
    
    Queue bridgeQ;
    QInit(&bridgeQ);
    
    
    int bridge_length = 2;
    int weight = 10;
    
    int truck_weights[] = {7, 4, 5, 6};
    int array_length = sizeof(truck_weights) / sizeof(int);
    
    int answer = solution(&waitQ, &bridgeQ, bridge_length, weight, truck_weights, array_length);
    printf("answer : %d\n", answer);

    return 0;
}



void QInit(Queue *pqueue) {
    pqueue->size = 0;
    pqueue->head = NULL;
    pqueue->tail = NULL;
}

int QSize(Queue *pqueue) {
    return pqueue->size;
}

int QIsEmpty(Queue *pqueue) {
    if (pqueue->size == 0) {
        return 1;
    } else {
        return 0;
    }
}

void QEnqueue(Queue *pqueue, Truck truck) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->truck = truck;
    newNode->prev = NULL;
    
    
    if(QIsEmpty(pqueue)) {
        pqueue->head = newNode;
    } else {
        pqueue->tail->prev = newNode;
    }
    pqueue->tail = newNode;
    pqueue->size++;
}

Truck QDequeue(Queue *pqueue) {
    Truck truck = pqueue->head->truck;
    
    Node* delNode = pqueue->head;
    pqueue->head = pqueue->head->prev;
    free(delNode);
    
    pqueue->size--;
    
    return truck;
}

Truck QPeek(Queue *pqueue) {
    return pqueue->head->truck;
}

int solution(Queue *waitQ, Queue *bridgeQ, int bridge_length, int weight, int truck_weights[], int array_length) {
    int flag = 0;
    int bridge = 0;
    int answer = 0;
    
    for(int i=0; i<array_length; i++) {
        Truck truck;
        truck.weight = truck_weights[i];
        truck.time = bridge_length;
        QEnqueue(waitQ, truck);
    }
    
    
    
    while(!QIsEmpty(waitQ) || !QIsEmpty(bridgeQ)) {
        Node* p = bridgeQ->head;
        
        while(p != NULL) {
            p->truck.time--;
            if (p->truck.time <= 0) {
                flag = 1;
            }
            p = p->prev;
        }
        
        if (flag) {
            Truck outTruck = QDequeue(bridgeQ);
            bridge -= outTruck.weight;
            flag = 0;
        }
        
        if (!QIsEmpty(waitQ)) {
            Truck inTruck = QPeek(waitQ);
            if (bridge + inTruck.weight <= weight) {
                QEnqueue(bridgeQ, QDequeue(waitQ));
                bridge += inTruck.weight;
            }
        }
        
        answer++;
    }
    
    return answer;
}

```