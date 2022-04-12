---
# author: cjlee
categories: jsp
# cover: /assets/covers/coding.png
date: "2020-09-17T11:48:00Z"
tags: ['null']
title: '# 2. JSP 학습기록 - Tomcat 구조 파악부터 Servlet 만들기까지'
---

# Directory
: Tomcat을 직접 사용하면서, 우리가 쳐다봐야 할 부분은 크게 세 곳으로 나뉜다.
* bin
* conf
* webapps

bin 폴더는, tomcat을 실행하기 위한 여러 batch file 들이 저장된 곳이다.   
startup.bat을 통해 tomcat을 실행시킬 수 있다.  

conf 폴더는, configuration의 약자로, 말 그대로 어떤 설정하는 파일들이 구성된 곳이다.  

webapps 폴더는, 우리가 페이지를 구성하기 위한 파일들을 모아놓는 곳이다. 주로 이곳을 다루게 된다.  

# webapps
: 먼저, webapps 폴더, 그리고 그 안에 있는 ROOT 폴더에 들어가보자.  
눈치가 빠르면 알겠지만, 우리가 localhost:8080 으로 접속했을 때 보이는 문서는  
여기에 있는 파일들을 통해서 이루어진 것이다.

**localhost:8080/index.jsp** 를 입력해보면, localhost:8080 으로 접속했을 때와 같은 화면이 보인다는 것을 알 수 있다.

그렇다면, **demo.txt** 라는 text 파일을 하나 만든 뒤, **"This is demo.txt"** 라고 적어보자.  
그리고, **localhost:8080/demo.tx**t로 접속하면, This is demo.txt 라는 문서가 보일 것이다.

![demo.txt](/assets/images/2020-09-17-13-22-01_2020-09-17-jsp_learning_02.md.png){: .alignCenter}

# Context
: Context란, 일종의 카테고리라고 생각하면 될 것 같다.  
혹은, 영어 이름 그대로의 "맥락" 이라고 이해해도 무관하다.

네이버의 홈페이지를 보면, 메일, 카페, 블로그 등등의 **여러 개의 sub 요소들이 존재**한다.  
이를 하나의 프로젝트, 하나의 디렉토리에서 관리하고자 한다면,  
이를 개발하는 개발 팀들이 관리하기가 어려울 것이다.

따라서, 이를 나누어서, 여러 개의 프로젝트, 여러 개의 디렉토리로 나눠서 개발하게 된다.  

이를 위해, 우리는 **ROOT 폴더 하위에, myFolder 라는 이름의 폴더**를 만들고, demo.txt를 해당 폴더로 옮겨보자.

![myFolder/demo.txt](/assets/images/2020-09-17-13-24-33_2020-09-17-jsp_learning_02.md.png){: .alignCenter}

잘 동작하는 모습을 볼 수 있다.

그러나, 이것만으로는 만족할 수 없다. 프로젝트를 여러 개로 나눈다는 것은,  
**완전히 다른 directory에서도 작업**을 할 수 있어야 한다.  

따라서, 디렉토리를 다른 곳에서 만든 뒤에,  
Tomcat에서는, **"이게 내가 관리해야 하는 폴더 중 하나구나"** 라는 것을 알 수 있도록  
**conf 폴더의 파일에서 설정**해줘야 한다.

테스트를 위해, C:/ 바로 하위에 myFolder를 이동시켜보자.  
당연하게도, 이 상태에서는 localhost:8080/myFolder/demo.txt 를 입력하면   
**404 Error**가 나게 된다. Tomcat에서 "야 니가 말한거 없는데?" 라고 에러를 보내주는 것이다.

conf 폴더로 이동해서, server.xml을 열어보자.  
밑으로 쭉 내려가다보면, 

```xml
<Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">

...

</Host>
```

이와 같이 생긴 Host tag가 보인다. 이 사이에 한줄 추가해주자.
```xml
<Context path="My" docBase="C:\myFolder" privileged="true"/>
```

![conf/server.xml](/assets/images/2020-09-17-13-41-14_2020-09-17-jsp_learning_02.md.png){: .alignCenter}

{: .caption}
대소문자를 주의하자.


저장하고, 톰캣을 재시작한뒤, localhost:8080/My/demo.txt 를 입력해보면,  
잘 동작하는 것을 확인할 수 있다.

즉, `My` 라는 가상의 경로를, `C:\myFolder` 라는 실제 경로로 mapping 시켜준 것이다.

# Servlet
: 오케이, 이제 이 Context라는 곳에서 우리가 개발을 하면 된다는 것을 알았다.  
이제, Servlet이라는 것을 한번 만들어보자.

C:\myFolder 하위에, myServlet.java 라는 파일을 하나 만들어보자.

```java
import javax.servlet.*;
import javax.servlet.http.*;
import java.io.*;

public class myServlet extends HttpServlet {
    public void service(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException {
        System.out.println("This is myServlet.");
    }
}
```

아래와 같이 코드를 작성하고, 이를 컴파일 시켜줘야 한다.  

이 때, HttpServlet 이라고 하는 클래스는, 자바에 기본적으로 내장된 클래스가 아니다.  
따라서, 이 클래스를 갖고 있는 라이브러리와 함께 컴파일 해줘야, myServlet이 올바르게 컴파일 될 수 있다.

cmd를 열어서, 다음과 같이 작성하자.

`javac -cp C:\TOOLS\apache-tomcat-9.0.37\lib\servlet-api.jar C:\myFolder\myServlet.java`

이 때, `C:\TOOLS\apache-tomcat-9.0.37\lib\servlet-api.jar` 라고 하는 부분은,  
**본인이 갖고 있는 tomcat 폴더**로 작성해야 한다.  

이렇게 컴파일 하고 나면, myServlet.class 를 확인할 수 있다.

![compile_myServlet](/assets/images/2020-09-17-13-49-16_2020-09-17-jsp_learning_02.md.png){: .alignCenter}

이 class file은, myFolder/WEB-INF/classes/ 라는 **예약된 곳**에 위치해야 하는데,   
해당 폴더를 그냥 만들게되면, 여러 **web.xml과 같은 설정파일이 없으므로**,   
일단은 **ROOT/WEB-INF/classes** 라는 폴더(classes는 기존에 없으므로 만들자)에 옮겨놓자.  

(IntelliJ 같은 IDE의 도움을 받으면, 이런 파일들이 자동으로 생성되어서,  
myFolder에서 곧바로 사용할 수 있지만, 우리는 아직 IDE를 사용하지 않으므로 ROOT 폴더를 활용하자)

![classes](/assets/images/2020-09-17-14-12-15_2020-09-17-jsp_learning_02.md.png){: .alignCenter}

그리고, **여기서 끝이 아니다.**  
방금전에 언급한 webapps/ROOT/WEB-INF/web.xml에서,  
우리가 해당 서블릿을 만들었다는 사실을 톰캣에게 알려줘야 한다.

```xml
<servlet>
        <servlet-name>my</servlet-name>
        <servlet-class>myServlet</servlet-class>
</servlet>

<servlet-mapping>
        <servlet-name>my</servlet-name>
        <url-pattern>/theServlet</url-pattern>
</servlet-mapping>
```

위 코드를 조금만 들여다보면 알 수 있다 시피,
servler-name 이라는 태그를 통해

**"my 라는 name의 서블릿을 만들었고, 이 서블릿을 실행하는 클래스는 myServlet 입니다"**  
**theServlet이라는 이름의 url이 들어왔을 때, 해당하는 my 라는 name을 가진 서블릿을 호출하겠습니다.**  

라고 두 가지를 **알려줘야 한다.**

이렇게 하고, 톰캣을 실행시켜서 결과를 확인해보자.

![result](/assets/images/2020-09-17-14-13-33_2020-09-17-jsp_learning_02.md.png){: .alignCenter}

그러나, 보면 실행 결과는 **아무것도 없다.**  
왜 그런가 하면, 우리가 해당 class로 하는 일은,  
System.out.println 으로 **콘솔에 찍어주는 일** 밖에 없기 때문이다.

따라서, 실행시켰던 Tomcat을 확인해보자.

![tomcat_console](/assets/images/2020-09-17-14-14-40_2020-09-17-jsp_learning_02.md.png){: .alignCenter}

보면, 가장 마지막에 This is myServlet 이라는 서블릿이 **정상적으로 호출**되었음을 확인할 수 있다.

다음 포스팅에서는, Servlet을 통해 **"브라우저에"** 입출력 하는 방법에 대해서 알아보자.


