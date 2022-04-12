---
# author: cjlee
categories: jsp
# cover: /assets/covers/coding.png
date: "2020-09-17T11:48:00Z"
tags: ['null']
title: '# 1. JSP 학습기록 - 기본 개념과 Tomcat 실행해보기.'
---

# 1. 기본 개념
: Web server, Web application server, Servlet, Servlet Container...

처음 봤을 때에는, 각각이 무엇을 의미하는지, 무슨 차이가 있는지에 대해서 알지 못했다.

이에 대해서 기본 개념을 정립하면 다음과 같다.

## 1) Web server, Web application server(WAS)

![webserver](/assets/images/2020-09-17-12-33-12_2020-09-17-jsp_learning_01.md.png){: .alignCenter}

___

: 우리가 html 파일을 만들고, 이를 더블클릭해서 열면, (대체로) 브라우저가 이 화면을 보여준다.  
이 때에는 Client와 Server의 개념이 딱히 없다. html이라는 양식을 통해 구성된, 어떤 화면을 Browser가 해석해서 우리에게 보여주는 것이다.

이를 다른 사람에게 보여주려고 하면, 이 때부터 **client와 server의 개념**이 등장한다.

client는 말 그대로, 사용자다.  
우리가 네이버나 구글에 크롬 같은 Web browser로 접속을 하면,  
http 라는 프로토콜(규약)을 통해 해당 server. 즉 네이버 본사에 인터넷으로 요청을 하게 된다.

그리고, web server는 요청을 받고,   
해당 사용자가 어떤 브라우저를 쓰는지 등의 데이터를 바탕으로 우리에게 화면을 보여준다. 

## 2) Static Page, Dynamic Page

: 그리고 이 때, Static(정적) 페이지와, Dynamic(동적) 페이지로 구분이 된다.

Static Page는, 우리가 어떤 요청을 하면, **항상 같은 결과**를 보여준다.  
현재 지금 포스팅을 작성하고 있는 이 페이지도, 정적인 페이지이므로, 우리가 뭔가 특별한 행동을 할 수는 없다.

Dynamic Page는, **우리가 요구하는 사항에 걸맞게 Contents를 반환**한다. 

Static Page는 마치 비유하자면, Java로 `System.out.println("안녕하세요");` 를 작성하는 것과 같다

반대로 Dynamic Page는,  `Scanner sc = new Scanner(System.in);` 과 같이,   
console에서 입력을 받고, 해당 내용을 `System.out.println(sc.next());` 로 출력하는 것과 같다.

똑같이 안녕하세요를 출력할 수 있지만,   
정적인 페이지의 경우 **코드를 수정하지 않는 한 내가 무슨 짓을 하더라도 "안녕하세요" 만 출력될 것**이고,   
동적인 페이지의 경우, 내가 **"반갑습니다" 를 입력하면, "반갑습니다"를 출력할 것**이다.

Static Page를 돌려주는 Server를 Web Server, 그리고 Dynamic Page를 돌려주는 Server를 Web application Server(WAS)라고 한다. 

Web server에는 Apache Server, Nginx, IIS 등이 있고,   
Web application Server는 **Tomcat**, JBoss, Jeus, Web Sphere 등이 있다.

## 3) Web server and Web application are not exclusive
: 또한, Web server와 WAS는 배타적으로 서로 하나를 선택해야 하는 관계는 아니다.

만약, 누가 언제 들어와도 같은 결과를 보여줘야 하는, 즉 Static Page가 있는데,   
이를 DB에서 읽어와서, 이를 코드 레벨에서 html로 잘 바꿔서 돌려주는 작업을 하는 것은 자원의 낭비이다.   
즉, 굳이 WAS로 구성할 필요가 없다.   
(이에 더해, 물리적으로 분리해서 보안을 강화시키거나, 하나의 Web server에 여러개의 WAS를 붙여서 부하를 감당하게 할 수 있다.)

따라서, 기본적으로 Web server를 앞에 하나 두고,   
Static Page 일 경우 Web server가 처리하고,   
Dynamic Page일 경우 Web server가 Web application server로 넘겨줘서 처리하면 된다.

___

> Note. [여기](https://pjh3749.tistory.com/267)를 참고하면,   
> Tomcat은 정확하게는 Web application server보다는 Servlet Container에 가깝다고 한다.   
> 아직 배움이 부족해서, EJB가 무엇인지 몰라 그런갑다 하고 넘겼다..

___

## 4) Servlet, Servlet Container
: Servlet이란, **Server Application + let(조각) 의 약자**라고 한다.  
즉, 어떤 요청이 들어왔을 때, 이에 담당하는 하나의 프로그램(가시적으로는 .java, .class 파일)을 말한다.

내가 Home directory에서 갈 수 있는 페이지가 A page, B page의 두 개의 Dynamic page라고 한다면, **각 페이지를 담당하는 프로그램을 servlet**이라고 한다.

그리고,**Servlet Container**는, 이렇게 여러 개의 Servlet을 갖고 있다가,
A page에 대한 요청이 들어오면, A servlet으로 **Mapping**해서,  
**요청사항(request)을 전달 한 뒤, 결과 값을 반환(response) 해주는 역할**을 한다. 

이 또한, 음식점으로 비유해볼 수 있을 것 같다.  
만약 어떤 중국집에서, 짜장, 짬뽕과 같은 면 요리를 담당하는 요리사,  
그리고 탕수육, 깐풍기와 같은 튀김 요리를 담당하는 요리사가 둘이 있다고 해보자.

이 **각각의 요리사는 하나의 Servlet**이다.  
고객으로부터 짜장이라는 요청이 들어오면, 면 요리사가 짜장을 요리해서 return한다.

그리고, 매니저가 Servlet Container가 될 것이다.   
짜장이라는 Request가 들어왔다면, 짜장 요리사에게 전달해준다.   
그리고, 조리가 완료되면 이를 받아서 다시 고객에게 잘 플레이팅해서 Response 해준다.

# 2. Tomcat 실행해보기
: 우선, Tomcat을 [공식홈페이지](http://tomcat.apache.org/) 에서 다운받았다.   
작성일자(2020.09.17) 기준 최신 버전은 9.0.38 이다.  

![tomcat_download](/assets/images/2020-09-17-12-53-47_2020-09-17-jsp_learning_01.md.png){: .alignCenter}

압축을 푼 뒤, 폴더를 열어서, /bin/startup.bat를 더블클릭해서 실행해보자.  
(실행이 안될 경우, 8080 port가 사용중인지, java 환경변수 설정을 했는지 확인해보자.)  

그리고, localhost:8080 으로 웹브라우저에 접속하면, 다음과 같은 화면을 볼 수 있다.
![tomcat_localhost](/assets/images/2020-09-17-13-16-30_2020-09-17-jsp_learning_01.md.png){: .alignCenter}

다음 포스팅에서, Tomcat 을 직접 수정하면서 다뤄보자.




### Reference
- [Web Server와 WAS의 차이와 웹 서비스 구조](https://gmlwjd9405.github.io/2018/10/27/webserver-vs-was.html)