---
categories: BTB
date: "2022-10-02T00:00:00Z"
tags: ['encoding', 'mysql']
title: '문자 인코딩과 MySQL varchar'
---

# 1. 문자 인코딩이란
문자 인코딩이란 사용자가 입력한 문자나 기호들을 컴퓨터가 이용할 수 있는 신호로 만드는 것을 말한다. [^문자_인코딩] 우리가 작성한 모든 텍스트는 결국 CPU 상에서 2진수로 처리된다. 사람이 작성한 글을 2진수로 변환하고, 2진수를 다시 사람이 읽을 수 있도록 만드는 작업을 각각 인코딩(encoding)과 디코딩(decoding)이라고 표현한다.

이러한 인코딩을 하기 위해서는 서로 약속한 '문자열 세트'가 필요하다. 가장 대표적인 문자열 세트는 [ASCII](https://ko.wikipedia.org/wiki/ASCII)다. ASCII는 1963년 미국 ANSI에서 표준화한 7 bit 부호 체계로 128 개의 기호(33개의 제어문자와 95개의 출력문자)를 표현할 수 있다. 1 bit 은 [Parity bit](https://en.wikipedia.org/wiki/Parity_bit)으로 7개의 bit 중 1의 개수가 홀수면 1, 짝수면 0으로 값을 할당하여 전송 과정 중 오류를 탐지하는 용도로 사용한다. 

[Extended ASCII](https://en.wikipedia.org/wiki/Extended_ASCII)는 이러한 parity bit 를 없애고 zone bit 를 추가하여, 8개의 bit로 총 256 개의 문자를 사용할 수 있도록 하였다.

# 2. 한글?
ASCII 코드는 영문자와 아라비안 숫자를 자유롭게 표현할 수 있었지만, 한글을 비롯한 여러 국가들의 문자를 표기할 수는 없었다. 1 byte만으로 수많은 언어의 문자들을 표현하기에는 아무래도 무리일 것이다. 

이를 개선하기 위해 마이크로소프트사에서 [EUC-KR](https://ko.wikipedia.org/wiki/EUC-KR) 와 이를 확장한 [CP949](https://ko.wikipedia.org/wiki/%EC%BD%94%EB%93%9C_%ED%8E%98%EC%9D%B4%EC%A7%80_949) 를 도입하였다. 두 문자열 세트는 '완성형' 이라고 하여, 2 byte 로 하나의 완성된 문자를 표현하는 방식이다. 즉, `가`, `힣`과 같은 특정한 문자에 코드를 할당한다. 이의 반대인 조합형은 한글의 제자 원리에 기반하여 초성, 중성, 종성에 각각 코드를 할당하는 방식이다.

EUC-KR 은 현대 한글에서 자주 사용하는 2350자 만을 지원하였다. 이는 반대로 말하면 자주 사용하지 않는 글자는 표현을 못한다는 의미가 된다. CP949 는 이를 개선하기 위해 EUC-KR을 확장하여 약 8000여 자를 추가하였다고 하지..만, 사실상 지금은 표준에 밀려 웬만하면 사용을 지양해야 하는 방식이 되어버렸다.

# 3. 유니코드
: 이렇듯 한글을 포함하여 여러 국가가 제각기 원하는대로 문자를 표현하고자 하다보니 여러 문제가 발생했다. 즉, 네트워크가 발전함에 따라 다른 국가의 홈페이지를 접속했더니 알 수 없는 글자로 표시되는 것이다. 이러한 문제를 해결하고자, 전 세계의 모든 문자를 표현할 수 있도록 [유니코드](https://ko.wikipedia.org/wiki/%EC%9C%A0%EB%8B%88%EC%BD%94%EB%93%9C) 라는 표준을 제정하게 되었다. 

![](2022-10-03-00-54-38.png)

유니코드에서 값을 나타내기 위해서는 **Code point** 를 사용하고, `U+` 를 붙여 표시한다. 가령, `A` 의 유니코드 값은 `U+0041` 혹은 `\u0041` 로 표현한다.
유니코드는 [평면](https://ko.wikipedia.org/wiki/%EC%9C%A0%EB%8B%88%EC%BD%94%EB%93%9C_%ED%8F%89%EB%A9%B4)이라는 논리적 개념을 이용하여 구획을 나누며, 이 중 0번 평면이 기본 다국어 평면(BMP, Basic Multilingual Plain)으로 쓰이다.

# 4. UTF-8

유니코드로 세계의 모든 문자를 표시할 수 있게 되었는데, 이를 결국 "어떻게 저장할 것인가?" 에 대한 문제가 다시 대두되었다. 우리가 오늘날 대부분 사용하는 인코딩 방식인 [UTF-8](https://ko.wikipedia.org/wiki/UTF-8)은 유니코드를 위한 **가변 길이 문자 인코딩** 방식 중 하나이다.

UTF-8 인코딩은 유니코드 한 문자를 나타내기 위해 1byte에서 최대 4byte를 사용한다. 가령, `a` 는 1 byte를 사용하지만, `가` 는 3byte를 사용한다. 이를 조금 더 자세히 살펴보겠다.

우선 UTF-8은 ASCII 와의 호환성을 위해 7 bit 이내의 코드값에는 맨 앞 비트를 0으로 붙이다. 즉, `a` 를 나타내기 위해서 `01000001` 를 사용한다.

7 bit 초과, 11 bit 이내의 코드 값은 (128 ~ 2047)은 총 2 byte로 표현한다. 즉, 
`110XXXXX 10XXXXXX` 로 표현한다. 첫 번째 byte의 `110` 은 순서대로 '이 문자가 1byte를 초과한다', '이 뒤에 1개의 byte가 더 있다', '여기까지가 범위를 나타낸 것이었고, 앞으로는 실제 값에 해당한다'를 의미한다. 두 번째 byte의 `10` 은 '이 byte는 앞선 byte의 후속이다' 를 의미한다.

이렇게 각각의 맨 앞 비트에 몇 byte가 더 추가될것인지에 대한 정보를 공유하고, 남은 11 자리로 문자를 표현하는데 사용한다. 

# 5. MySQL 에서 문자열 다루기

## 5.1. utf8mb3 ? utf8mb4 ?
여기까지 간단하게 컴퓨터에서 어떻게 문자열을 처리하는지에 대해 알아보았다. 그렇다면 본 주제인 MySQL 의 인코딩에 대해 알아보자.

MySQL 5.X 를 기준으로 기본 인코딩은 `utf8mb3` 를 사용했다. 기존에 사용하던 `utf8`은 `utf8mb3`의 alias 이다. MySQL 8.X 로 넘어와서는 `utf8mb4`를 사용한다.(`utf8mb3` 는 deprecated 될 예정이다.[^2]) 그렇다면 이 두 인코딩 방식의 차이는 무엇일까 ?

MySQL 8.0 공식문서에서는 다음과 같이 설명한다.

> `utf8mb3` 는 BMP 문자만 지원하고, 문자당 3 byte만을 할당한다는 점에서 `utf8mb4`와 대조된다.
> - BMP 문자에 대해서, `utf8mb4` 와 `utf8mb3` 는 동일한 특성을 지닌다 : 같은 코드 값, 인코딩, 길이를 가진다.
> - 추가 문자(supplementary character)의 경우, `utf8mb4` 는 저장을 위해 4 byte를 필요로 하지만, `utf8mb3` 는 해당 문자를 저장할 수 없다. `utf8mb3` 칼럼을 `utf8mb4` 로 바꿀 때, 당신은 추가 문자가 없기 때문에(애초에 저장된 적이 없으므로) 이에 대해 걱정할 필요가 없다. 

## 5.2. MySQL 에서 문자열의 길이 세기
위 내용들을 통해, 우리는 MySQL 에서 왜 `length()` 와 `char_length()` 가 따로 존재하는지 알 수 있다. `length()` 함수는 문자열의 byte 길이를 구하는 반면, `char_length()` 는 글자의 길이를 구한다.

```sql
length('한글') -- 6
length('english') -- 7

char_length('한글') -- 2
char_length('english') -- 7
```

## 5.3. MySQL의 varchar, 그리고 인덱스
MySQL에서 주로 사용하는 `varchar` 자료형은 첫 2 byte 에 문자열의 길이에 대한 정보를 저장한다. (정확히는 길이가 255 이하인 경우 1 byte를, 초과인 경우 2 byte를 사용한다.) 그리고, 앞서 MySQL 5.X 버전을 기준으로 default 로 사용되던 `utf8mb3` 는 최대 3 byte를 사용한다는 것을 확인하였다.

MySQL 에서 문자열을 index로 사용할 경우, 최대 767(3 * 255 + 2) bytes를 사용할 수 있다. 이는 바꿔말하면 **`utf8mb3` 를 기준으로, `varchar(255)` 자료형을 사용하면 모든 문자열을 인덱스에 담을 수 있다** 가 된다. 하지만 8.X 버전에 들어서면서 `utf8mb4` 를 사용하게 될 경우에는 성립하지 않는다.

> 좀 더 정확히는, InnoDB를 기준으로, `Dynamic` 혹은 `Compressed` row format 을 사용한다면 3072 byte를, `REDUNDANT` 혹은 `COMPACT` row format을 사용하면 767 bytes를 사용할 수 있다. 자세한 내용은 [공식문서](https://dev.mysql.com/doc/refman/8.0/en/innodb-limits.html)를 확인하자.

[^문자_인코딩]:[위키피디아 : 문자 인코딩](https://ko.wikipedia.org/wiki/%EB%AC%B8%EC%9E%90_%EC%9D%B8%EC%BD%94%EB%94%A9)

[^2]:[MySQL 8.0 Reference Manual, 10.9.8 Converting Between 3-Byte and 4-Byte Unicode Character Sets](https://dev.mysql.com/doc/refman/8.0/en/charset-unicode-utf8mb3.html)

### reference
- [한글 인코딩의 이해 1편: 한글 인코딩의 역사와 유니코드](https://d2.naver.com/helloworld/19187)
- [What is the difference between utf8mb4 and utf8 charsets in MySQL?](https://stackoverflow.com/questions/30074492/what-is-the-difference-between-utf8mb4-and-utf8-charsets-in-mysql)
- [MySQL 8.0 Reference Manual, 10.9.1 The utf8mb4 Character Set (4-Byte UTF-8 Unicode Encoding)](https://dev.mysql.com/doc/refman/8.0/en/charset-unicode-utf8mb4.html)
