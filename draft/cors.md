---
categories: web
date: "2022-06-10T21:31:00Z"
tags: ['CORS', 'SOP']
title: '# CORS 이야기'
---
 
# CORS ?
: CORS란 `Cross-Origin-Resource-Sharing` 의 약자로, 한국말로 "교차 출처 자원 공유"로 직역됩니다. 웹 개발을 하다보면 이 `CORS` 라는 녀석을 필연적으로 마주할 수 밖에 없고, 저 또한 우아한테크코스의 '장바구나' 미션을 진행하면서 이 CORS 문제에 직면하였습니다. Postman으로 테스트할 때도 정상적으로 동작하고, 웹 브라우저에서 백엔드 서버로 직접 요청을 보낼때도 문제가 없었는데, 프론트엔드분이 작업하신 어플리케이션을 띄우고 백엔드로 요청을 보냈더니 문제가 생겼습니다. 요청이 정상적으로 처리되지 않아 개발자 도구를 열어보니, 아래와 같은 에러가 나고 있었던 것이죠.

![](/assets/images/2022-06-11-23-58-31.png)
*보자마자 올게 왔구나 싶었습니다*

심지어는 아래와 같이 뜬금없는 `OPTIONS` 요청을 보내고 있었습니다. 분명 팀 회의 때 이런 메소드를 보낸다는 이야기는 없었는데요. 내가 모르는 프론트분들만의 규약이 있었나 싶었는데, 알고보니 해당 요청은 프론트분들이 작성하신 코드가 아닌, 브라우저 레벨에서 보내는 요청이었습니다.

![](/assets/images/2022-06-11-23-59-25.png)

아래와 같은  프론트엔드의 요청이 `localhost:3000`에서 출발하고, 이를 수신하는 녀석이 스프링의 기본 포트인 `localhost:8080` 
이 CORS를 이해하기 위해선 SOP 를 알아야하고, SOP를 알기 위해선 '출처' 라는 개념과 SOP의 등장 배경에 대해서 이해해야 하는데요. 제가 아는 한 가장 밑바닥부터 하나씩 타고 살펴보도록 하겠습니다.

### 출처
: 출처를 이야기 하기 위해서는 URI의 구조에 대해서 알아야 합니다. URI는 아래와 같은 구조로 이루어져 있는데요. 

![](/assets/images/2022-06-11-22-49-01.png)

이 중에서도 scheme, host, port가 모두 일치하는 경우 같은 출처를 가졌다고 이야기합니다. 아래 예시를 통해 쉽게 비교 & 이해할 수 있습니다.

비교 대상 : `http://store.company.com/dir/page.html`
|URL|Outcome|Reason|
|:--:|:--:|:--:|
|http://store.company.com/dir2/other.html|Same origin|Only the path differs|
|http://store.company.com/dir/inner/another.html|Same origin|Only the path differs|
|https://store.company.com/page.html|Failure|Different protocol|
|http://store.company.com:81/dir/page.html|Failure|Different port (http:// is port 80 by default)|
|http://news.company.com/dir/page.html|Failure|Different host|

세 번째 예시는 `http` 대신 `https` 를 사용했기 때문에 서로 다른 출처로 인식됩니다.  
네 번째 예시의 경우, `http`의 기본 포트가 80 이기 때문에, 비교 대상 URI의 생략된 80 번과, 네 번째 예시에 명시된 81 번 포트가 다르기 때문에 서로 다른 출처로 인식됩니다.  
마지막 예시 또한, `store.company.com` 이 아닌 `news.company.com` 으로 다른 호스트를 갖고 있기 때문에 서로 다른 출처입니다.

## SOP
<!-- SOP란 `Single-Origin-Policy` 의 약자로, 역시 한국말로 '동일 출처 정책' 이라고 직역됩니다. 즉, 브라우저에서 '동일한 출처'에서 얻어낸 자원만 공유할 수 있도록 하는 정책입니다. 즉, 위에서 보았던 서로 다른 출처로부터 요청이 이루어지는 경우에, 브라우저가 올바르지 않은 요청이라고 판단하여 수신한 결과를 block 시켜버립니다. 왜 굳이 이런 불편한 정책을 만들었을까요?  -->

<!-- [XSS](https://developer.mozilla.org/en-US/docs/Glossary/CSRF), [CSRF](https://developer.mozilla.org/en-US/docs/Glossary/CSRF) -->


> 여기서 주의해야 할 점은, Internet Explorer는 독자적으로 위 표준을 지키지 않고 다음의 두가지 예외를 적용합니다.
> 
> 1. 만약 두 도메인이 높은 신뢰 수준을 갖고 있다면, 상호간의 SOP는 적용되지 않습니다.  
> 2. 포트 번호를 출처 기준에 포함하지 않습니다. 즉, 아까 위에서 보았던 표 예시에서, 포트 번호가 달랐던 네 번째 항목은 서로 다른 출처로 인식되지 않습니다.
   

## CORS
: SOP로 인해 서로 다른 출처로부터 자원을 공유할 수 없을 때, 우리가 알아야 할 것이 바로 `CORS` 입니다. 이는, `A` 라는 origin으로부터 `B` 로 요청을 보내고, `B`가 이에 대한 응답을 내려줄 때, "나는 A 서버로부터의 요청을 허락한다" 라는 헤더를 덧붙여줍니다. 

## ref

- [mozilla - 출처](https://developer.mozilla.org/en-US/docs/Glossary/Origin)
- https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy
