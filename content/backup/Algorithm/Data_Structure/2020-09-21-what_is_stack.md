---
# author: cjlee
categories: data-structure
# cover: /assets/covers/data-structure.jpg
date: "2020-09-21T18:13:00Z"
tags: ['null']
title: '# 마음으로 이해하는 자료구조 : Stack ( feat. C )'
---

# 0. 들어가며
: Stack과 Queue는 아주 기본 중의 기본인 자료구조이다.  
아주 간단하므로, 사실 Stack과 Queue가 무엇인지에 대해서 이야기할 부분은 많지 않다.

따라서, 이번 포스팅은 코드 구현 위주로 작성할 것이다.  
쓰다보니, 생각보다 포스팅이 또 길어져서, Stack과 Queue를 나누어서 작성했다.

# 1. 이론 편
: Stack 이란 말 그대로, "쌓는 것"이다. **접시쌓기**를 생각해보면 된다.  
접시를 쌓을 때에는, 처음 혹은 중간에 끼워 넣을 수 없으며, **가장 상위에만 쌓아야 한다**.  
반대로, 꺼낼 때에도, 처음 혹은 중간에서 꺼낼 수 없으며, **가장 상위에서만 꺼내야 한다.**

이를,**LIFO(Last in, First out)(후입선출)** 라고 한다.


# 2. 구현 편 - ADT

___

> Note. ADT란, Abstract Data Type의 약자로, 프로그램의 대상이 되는 사물이나 현상을 **추상화** 하여 정의 한 것을 의미한다.
> 이는, 해당 기능의 완성과정이 아닌 **기능의 종류를 나열**한 것이다.
> 예를 들면, 지갑의 ADT는 "현금 넣기, 현금 꺼내기, 카드 넣기, 카드 꺼내기 ..." 등이 된다.

___

: Stack의 ADT는 다음과 같다.

- init()
  * Stack의 초기화를 진행
  * Stack 생성 후, 가장 먼저 호출되어야 하는 함수.
- size()
  * 해당 Stack에 데이터가 몇 개 쌓여 있는지 확인.
- isEmpty()
  * 해당 Stack이 비어있는가를 확인.
  * 비었으면 true, 아니면 false 를 반환
- push(data)
  * Stack에 데이터를 저장.
  * 당연히, 가장 맨 위에 저장한다.
- pop()
  * Stack에 있는 데이터를 반환.
  * 꺼낸 데이터는, **Stack에서 삭제**한다.
  * 역시 당연히, 가장 맨 위에서 꺼낸다. 
  * **Stack에 데이터가 최소 하나는 있어야 한다.**
- peek()
  * Stack에 있는 데이터를 반환
  * 꺼낸 데이터는 **삭제하지 않는다.**
  * 물론 역시 당연히, 가장 맨 위에서 꺼낸다.
  * **Stack에 데이터가 최소 하나는 있어야 한다.**


# 3. 구현 편 - Code
: 지난 편에, LinkedList를 배웠으므로, 배열이 아닌 LinkedList 를 이용해 구현할 것이다.  
먼저, Stack을 위한 구조체를 만들어야 한다.

```c++
typedef struct Stack {
    int size;
    struct Node* head;
}Stack;

typedef struct Node {
    int data;
    struct Node* prev;
}Node;
```

Stack 구조체는 Stack의 크기, 그리고 가장 상위에 있는 Node를 가리키는 구조체이다.  
Node 구조체는, data를 담고 있고, Pop이 되었을 경우, 이전 데이터를 가리켜야 하기 때문에, struct Node* prev; 를 가진다.

먼저, 초기화를 시켜주는 init() 함수를 만들어보자.

```c++
void SInit(Stack* pstack) {
    pstack->size = 0;
    pstack->head = NULL;
}
```
처음에는 아무런 데이터도 들어있지 않으므로, size는 0, head는 NULL로 초기화 시킨다.

그리고, 가장 단순한, size() 함수와 IsEmpty() 함수를 구현해보자.
```c++
int SSize(Stack* pstack) {
    return pstack->size;
}

int SIsEmpty(Stack *pstack) {
    if (pstack->size == 0) { // if (SSize(pstack) == 0) 으로 대체해도 무방.
        return 1;
    } else {
        return 0;
    }
}

```
이정도는 코드에 대한 주석이 없어도 이해가 될 것이라 생각한다.

다음으로, 핵심이 되는 push()와 pop() 함수를 구현해보자.

```c++
void SPush(Stack *pstack, int data) {
    Node* newNode = (Node *)malloc(sizeof(Node));
    
    newNode->data = data;
    newNode->prev = pstack->head;
    
    pstack->head = newNode;
    pstack->size++;
}
```

![stack_push](/assets/images/2020-09-21-21-28-52_2020-09-21-what_is_stack.md.png){: .alignCenter}

push() 함수는, stack 이외에 data라는 int형 변수를 하나 받는다.  
그리고 **Node 만큼의 공간을 새로이 할당**하고, 이를 **newNode가 가리키도록 한다.** 

newNode의 data는 인자로 전달받은 data를,  
그리고 **newNode의 prev**는 기존에 가장 상위에 있던 **pstack->head** 를 가리키도록 한다. **(그림의 1번)**

마지막으로,**pstack->head 가 newNode를 가리키도록 하고**, **(그림의 2번)**
pstack의 size를 하나 증가시킨다.

```c++
int SPop(Stack *pstack) {
    int data = pstack->head->data;
    
    Node* delNode = pstack->head;
    pstack->head = pstack->head->prev;
    free(delNode);
    
    pstack->size--;
    
    return data;
}
```
![stack_pop](/assets/images/2020-09-21-21-31-49_2020-09-21-what_is_stack.md.png){: .alignCenter}
pop() 함수는 삭제하는 작업이 필요하다.  
가장 먼저, pstack->head 의 Node가 갖고 있는 data를 int data에 저장한다.

그리고, **delNode가 pstack의 head를 가리키게 하고**(그림의 1번)  
**pstack의 head는 하나 이전의 요소를 가리키게 한다.**(그림의 2번)  
마지막으로, free(delNode)를 통해, delNode가 가리키는 요소를 삭제하게 하면 된다.

이후, pstack의 data를 하나 감소시키고, data를 return 해주면 된다.

마지막으로, peek()은 아주 간단하다.

```c++
int SPeek(Stack *pstack) {
    return pstack->head->data;
}
```

head가 가리키고 있는 data를 return 하면 된다.

이를 테스트 하는 코드는 다음과 같다.
```c++
int main() {
    Stack stack;
    SInit(&stack);
    
    printf("is stack Empty ? : %d\n", SIsEmpty(&stack));
    // is stack Empty ? : 1
    // stack이 비어있으므로, true인 1을 반환한다.
    
    SPush(&stack, 1);
    SPush(&stack, 4);
    SPush(&stack, 3);
    SPush(&stack, 15);
    // 순서대로 1,4,3,15를 Push한다.
    
    printf("stack size : %d\n", SSize(&stack)); 
    // stack size : 4
    // push 이후 size는 4
    
    int data = SPop(&stack);
    printf("data : %d, current stack size : %d\n", data, SSize(&stack));
    // data : 15, current stack size : 3
    // 데이터를 하나 꺼내보면, 가장 마지막에 넣은 15가 return 되고, size는 3으로 감소하였음을 볼 수 있다.
    
    printf("peek : %d\n", SPeek(&stack));
    printf("peek : %d\n", SPeek(&stack)); 
    // peek : 3
    // peek : 3
    // peek 함수는 여러 번 실행해도, 같은 값을 보여준다.
    // 현재 Stack에는 1, 4, 3 이 들어있다.
    
    while(!SIsEmpty(&stack)) {
        printf("pop data : %d\n", SPop(&stack));
    }
    // pop data : 3
    // pop data : 4
    // pop data : 1
    // stack이 빌 때까지, pop 하면 역순으로 3,4,1이 출력된다.
    
    printf("After all, Stack Size : %d\n", SSize(&stack));
    // After all, Stack Size : 0
    // Stack이 비어있으므로, size는 0이 된다.

    return 0;
}
```

전체 코드는 포스팅 가장 하단에 작성하였다.

# 4. 응용 편 - 괄호 여닫기
: 우리가 프로그래밍을 할 때, () 의 소괄호가 되었든, {} 의 중괄호가 되었든, []의 대괄호가 되었든,  
**괄호는 항상 열렸으면 닫아줘야 한다.**

이 소괄호가 올바르게 열고 닫혔는지 확인하는 방법은,  
Stack을 이용하면 아주 간단하게 해결할 수 있다.

프로그래머스에도 이러한 문제가 있는데,   
[지난 번에 푼 여기](https://cjlee38.github.io/problem_solving/problem_solving_4) 를 참고하자.

___

> **주의 : 단순히 괄호의 개수를 세는 것만으로는 이 문제를 해결할 수 없다!**  
> 가령, 다음과 같이 괄호가 열고 닫혔다고 해보자.  
> 
> **())(**
> 
> 열린건 한번인데, 두번을 닫아버리면, 이는 잘못된 괄호이다.  
> 이럴 경우에는, 에러로 처리해야 한다.
 
___

Stack으로 어떻게 해결할 수 있는가, 하면 
* '(' 의 여는 괄호가 들어오면, stack에 Push 한다.
* ')' 의 닫는 괄호가 들어오면, stack에서 Pop 한다.
* 정상적으로 끝났다면, 이는 올바른 괄호다.
* 만약 정상적으로 처리가 되지 않는다면, 이는 잘못된 괄호다.

기존의 코드를 살짝 수정해보자.

int형 data 대신, char 형 data를 받도록 하고, 로직에 맞게 코드만 작성해주면 된다.
```c++
typedef struct Node {
    char data;
    struct Node* prev;
}Node;
```
당연히, data와 관련된 다른 녀석들도 int에서 char로 바꿔주면 된다(e.g. SPop())

```c++
char brackets[] = "((()))";
int answer = isCorrectBrackets(&stack, brackets, strlen(brackets));

if (answer) {
    printf("It's correct !\n");
} else {
    printf("It's wrong !\n");
}
```
먼저, brackets 라는 char 형 data의 변수를 만들고,  
isCorrectBrackets() 라는 함수를 통해, 이것이 올바른 괄호인지 아닌지 확인할 것이다.

```c++
int isCorrectBrackets(Stack *pstack, char brackets[]) {
    int answer = 1;
    for (int i=0; i<strlen(brackets); i++) {
        char current = brackets[i];
        if (current == '(') { // '(' 이 들어오면 무조건 push
            SPush(pstack, current);
        } else { // ')'이 들어왔다면
            if (SIsEmpty(pstack)) { // ')'이 들어왔는데, stack이 비어있다면, 즉 열린 괄호가 없는데 닫으려 한다면
                answer = 0; // 잘못된 괄호이므로 false
                break;
            } else {
                SPop(pstack); // stack이 비어 있지 않다면, pop
            }
        }
    }
    
    
    // 다 돌았는데, Stack이 남아있다면, 열린 괄호가 남아 있다는 뜻.
    if (!SIsEmpty(pstack)) {
        answer = 0;
    }
    
    return answer;
}
```

strlen() 함수를 사용하기 위해, `#include <string.h>` 또한 작성하였다.  
또한, 코드에 대한 설명은 주석으로 달아두었다.

# 구현 코드들.

**stackImpl.c**
```c++
#include <stdio.h>
#include <stdlib.h>

typedef struct Stack {
    int size;
    struct Node* head;
}Stack;

typedef struct Node {
    int data;
    struct Node* prev;
}Node;

void SInit(Stack *pstack);
int SSize(Stack *pstack);
int SIsEmpty(Stack *pstack);
void SPush(Stack *pstack, int data);
int SPop(Stack *pstack);
int SPeek(Stack *pstack);


int main() {
    Stack stack;
    SInit(&stack);
    
    printf("is stack Empty ? : %d\n", SIsEmpty(&stack));
    
    SPush(&stack, 1);
    SPush(&stack, 4);
    SPush(&stack, 3);
    SPush(&stack, 15);
    
    printf("stack size : %d\n", SSize(&stack));
    
    int data = SPop(&stack);
    printf("data : %d, current stack size : %d\n", data, SSize(&stack));
    
    printf("peek : %d\n", SPeek(&stack));
    printf("peek : %d\n", SPeek(&stack));
    
    while(!SIsEmpty(&stack)) {
        printf("pop data : %d\n", SPop(&stack));
    }
    
    printf("After all, Stack Size : %d\n", SSize(&stack));

    return 0;
}

void SInit(Stack* pstack) {
    pstack->size = 0;
    pstack->head = NULL;
}


int SSize(Stack* pstack) {
    return pstack->size;
}

int SIsEmpty(Stack *pstack) {
    if (pstack->size == 0) {
        return 1;
    } else {
        return 0;
    }
}


void SPush(Stack *pstack, int data) {
    Node* newNode = (Node *)malloc(sizeof(Node));
    
    newNode->data = data;
    newNode->prev = pstack->head;
    
    pstack->head = newNode;
    pstack->size++;
}

int SPop(Stack *pstack) {
    int data = pstack->head->data;
    
    Node* delNode = pstack->head;
    pstack->head = pstack->head->prev;
    free(delNode);
    
    pstack->size--;
    
    return data;
}

int SPeek(Stack *pstack) {
    return pstack->head->data;
}
```

**Brackets.c**
```c++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct Stack {
    int size;
    struct Node* head;
}Stack;

typedef struct Node {
    char data;
    struct Node* prev;
}Node;

void SInit(Stack *pstack);
int SSize(Stack *pstack);
int SIsEmpty(Stack *pstack);
void SPush(Stack *pstack, char data);
char SPop(Stack *pstack);
char SPeek(Stack *pstack);
int isCorrectBrackets(Stack *pstack, char brackets[], int length);


int main() {
    Stack stack;
    SInit(&stack);
    
    char brackets[] = "((()))";
    int answer = isCorrectBrackets(&stack, brackets);
    
    if (answer) {
        printf("It's correct !\n");
    } else {
        printf("It's wrong !\n");
    }
    
    

    return 0;
}

void SInit(Stack* pstack) {
    pstack->size = 0;
    pstack->head = NULL;
}


int SSize(Stack* pstack) {
    return pstack->size;
}

int SIsEmpty(Stack *pstack) {
    if (pstack->size == 0) {
        return 1;
    } else {
        return 0;
    }
}

void SPush(Stack *pstack, char data) {
    Node* newNode = (Node *)malloc(sizeof(Node));
    
    newNode->data = data;
    newNode->prev = pstack->head;
    
    pstack->head = newNode;
    pstack->size++;
}

char SPop(Stack *pstack) {
    char data = pstack->head->data;
    
    Node* delNode = pstack->head;
    pstack->head = pstack->head->prev;
    free(delNode);
    
    pstack->size--;
    
    return data;
}

char SPeek(Stack *pstack) {
    return pstack->head->data;
}

int isCorrectBrackets(Stack *pstack, char brackets[]) {
    int answer = 1;
    for (int i=0; i<strlen(brackets); i++) {
        char current = brackets[i];
        if (current == '(') {
            SPush(pstack, current);
        } else {
            if (SIsEmpty(pstack)) {
                answer = 0;
                break;
            } else {
                SPop(pstack);
            }
        }
    }
    
    
    
    if (!SIsEmpty(pstack)) {
        answer = 0;
    }
    
    return answer;
}

```