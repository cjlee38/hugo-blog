---
# author: cjlee
categories: jsp
# cover: /assets/covers/coding.png
date: "2020-09-21T15:22:00Z"
tags: ['null']
title: '# 4. JSP 학습기록 - POST handling과 Servlet Filter'
---

# POST 다루기
: [지난 3편](https://cjlee38.github.io/jsp/jsp_learning_03)에 이어서, POST를 다뤄보자.

이를 위해, 지난 편에 작성했던 html 코드를 조금 더 업그레이드 시켜보자.

```html
<div>
    <form action="firstServlet" method="POST">
        <div>
            <label> input your title </label>
            <input name = "title" type="text"/>
        </div>
        <div>
            <label> input your content </label>
            <textarea name="content"></textarea>
        </div>
        <div>
            <input type="submit" value="click here to submit"/>
        </div>
    </form>
</div>

```

기존과 달라진 점이라면  
1) 입력을 받는 area가 title과 content로 구분되었고,  
2) `<form>` 태그에 method="POST"가 추가 되었다.

이렇게하면, title과 content과 POST를 통해 넘어가게 된다.  
한번 테스트 해보자.

![html_test](/assets/images/2020-09-21-16-29-32_2020-09-21-jsp_learning_04.md.png){: .alignCenter}

위와 같이, 작성하고, **submit을 누르기 전에**,  
F12를 눌러 개발자도구를 열고, Network 탭을 열어놓자.  
그리고, submit을 누르자.

![submit_result](/assets/images/2020-09-21-16-30-40_2020-09-21-jsp_learning_04.md.png){: .alignCenter}

submit을 누르고 나면, 위와 같이, Network tab에 firstServlet이라는 document가 있는 것을 볼 수 있다.

아래 쪽을 보면, Form data라고 쓰인 부분에,  
title: some title here  
content: this is content  
를 볼 수 있다.

그렇다면, 이를 서버에서 받아보자. 받는 법은 get을 받을 때와 같다.
```java
@WebServlet("/firstServlet")
public class firstServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter out = resp.getWriter();

        String title = req.getParameter("title");
        String content = req.getParameter("content");

        out.println("title : " + title);
        out.println("content : " + content);

        
    }
}
```

똑같이 getParameter()를 활용해서, String으로 받은 뒤, 이를 출력하면 된다.  
그 결과는 다음과 같다.


![server_response](/assets/images/2020-09-21-16-35-48_2020-09-21-jsp_learning_04.md.png){: .alignCenter}

잘 나타나는 모습을 볼 수 있다.

이렇게 GET과 POST의 차이는 다음과 같다.

|GET|POST|
|:--:|:--:|
|상대적으로 빠름|상대적으로 느림|
|데이터를 URL로 전송|데이터를 Body로 전송|
|256Byte 제한|데이터 제한 없음|


# Encoding & Content-Type
: 그런데, Title과 Content에 한글을 넣어서 submit을 해보자.

![submit_korean](/assets/images/2020-09-21-16-37-50_2020-09-21-jsp_learning_04.md.png){: .alignCenter}

![submit_korean_result](/assets/images/2020-09-21-16-38-15_2020-09-21-jsp_learning_04.md.png){: .alignCenter}

위와 같이, 결과가 **깨져서** 나오는 것을 볼 수 있다.

이는 Client와 Server의 **Encoding이 서로 달라** 발생하는 문제이다.  
이 외에도, ContentType의 문제가 발생할 수 있다.


지금 당장의 문제는 encoding이 원인인데, 겸사겸사 해서 ContentType 문제까지 해결해보자.

### 1. Encoding
: 간단하다. request와 response에 둘다 setCharacterEncoding을 통해 setting 해주면 된다.   
주의할 점은, **"둘 다"** 해줘야 한다는 것이다.   

```java
req.setCharacterEncoding("UTF-8");
resp.setCharacterEncoding("UTF-8");
```

가령, response에만 encoding 을 설정하고,  `out.println("안녕하세요");` 를 작성하고 실행하게 되면,  
"안녕하세요" 는 정상적으로 출력 된다. 그러나, 위 예시와 같이 입력을 받는다면,  
UTF-8이 아니기 때문에 **애초에 받을 때부**터 String이 깨지게 된다.  

따라서, 이를 주의해야 한다.


### 2. ContentType
: Content Type이란, "이 문서가 무슨 형식으로 되어있는지" 와 관련되어 있다.  
우리가, txt 파일을 읽으면, 메모장가 열려서 txt파일을 해석해준다.  
docx 파일을 열면, Microsoft Word(혹은 다른 편집기)가 열려서 이를 해석해준다.  

웹도 마찬가지다. 우리가 보낸것이, 단순 txt인지, 혹은 html 인지를 알려줘야 한다.  
한번 테스트 해보자.

```java
out.println("this is content-Type test<br>");
```

기존 코드에 위 코드를 추가하고, (구시대의 유물인)*Internet Explorer*에서 열어보자.

![Internet_explorer](/assets/images/2020-09-21-16-56-39_2020-09-21-jsp_learning_04.md.png){: .alignCenter}

{: .caption}
익스플로러 결과

![Chrome](/assets/images/2020-09-21-16-57-36_2020-09-21-jsp_learning_04.md.png){: .alignCenter}

{: .caption}
크롬 결과

위와 같이, 익스플로러에서는 줄바꿈도 이루어지지 않았고, br 태그 또한 보이지 않는다.  
(url에 곧바로 접근했으므로 null이 나타나는 것은 정상이다.)

이는, 받은 문서를 **익스플로러는 "HTML"**로 해석했고,  
**크롬은 "TEXT"**로 해석했기 때문이다.

따라서, 이런 브라우저 간 차이를 막기 위해, ContentType을 지정해야 한다.  

html로 지정하는 방법 또한 간단하다. 
```java
resp.setContentType("text/html; charset=UTF-8");
```
이 코드를 추가해주면 된다.

### 3. 뭔가 좀 아쉽다.
: 이렇게만 하면, 굉장히 뭔가 아쉽다.  
만약 우리가, 실제 프로젝트를 진행한다면, 돌아다닐 페이지가 한 두개가 아닐텐데,  
이를 각각 다 set, set, set.. 해주는 것은 너무 귀찮다.

또한, 일종의 configuration 인데, 이렇게 class 마다 지정해주는 것도 조금 이상하다.  

그런데 그렇다고해서, *모~~든 서비스에 인코딩을 지정해주고자 톰캣 자체에 세팅을 하는것* 또한 이상하다.

따라서, 우리는 서블릿 필터를 활용해서, **"하나의 컨텍스트에 설정"**을 해줄 것이다.


# Servlet Filter
: filter란 이름 그대로, 무언가가 행해지기 전에,  
이 filter를 반드시 거쳐야 한다.

모든 컨텐츠에 대해서, 우리는 위와 같은 Encoding과 ContentType을 설정할 것이므로,  
이 Filter에다가 해당 코드를 집어넣어보자.

먼저, myProject/src/main/java 하위에, 다음과 같이 myFilter 라는 java file을 하나 만들자.

![Filter.java](/assets/images/2020-09-21-17-07-11_2020-09-21-jsp_learning_04.md.png){: .alignCenter}

그리고, 이 myFilter class는, `javax.servlet.Filter`를 구현해야 한다.

```java
import javax.servlet.*;

public class myFilter implements Filter {

}
```

이때, doFilter 라는 method를 구현해야 하므로, IDE의 도움을 받아 자동완성 시켜보자.

```java
@Override
public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {

}
```

기존의 다른 코드들과 다르게, FilterChain 이라는 인자가 하나 더 늘어났다.  
이 filterChain.doFilter()를 통해서,   
**"야, 지금 이 request와 response 의 설정을 적용해라"** 라고 지정해주는 것이다.  
그리고, 필터를 적용한 후에 뭔가 해야할 일이 있으면, 이 filterChian.doFilter() 뒤에 넣어주면 된다.  

어쨌든, 우리가 하고자 하는 Encoding과 ContentType 설정은 필터에 적용할 대상이므로,  
filterChain.doFilter이전에 작성하면 된다.

**그리고 마지막으로,** 역시 이 Filter 또한 url을 mapping 시켜줘야 한다.

똑같이 WebServlet을 쓰면 안되고, 이번에는 WebFilter라는 것이 필요하다.  
우리는, 해당 Context의 모든 page에 적용할 것이므로, "/*" 의 와일드카드를 통해 적용시키자.

그리고, 기존의 firstServlet 안에 있는 setCharacterEndoing, setContentType을 다 지워주고,  
잘 동작하는지 확인해보자.

완성된 코드는 다음과 같다.

myFilter.java
```java
@WebFilter("/*")
public class myFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        servletRequest.setCharacterEncoding("UTF-8");
        servletResponse.setCharacterEncoding("UTF-8");
        servletResponse.setContentType("text/html; charset=UTF-8");

        filterChain.doFilter(servletRequest, servletResponse);
    }

}
```

firstServlet.java
```java
@WebServlet("/firstServlet")
public class firstServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter out = resp.getWriter();

        String title = req.getParameter("title");
        String content = req.getParameter("content");

        out.println("title : " + title);
        out.println("content : " + content);

        out.println("this is content-Type test<br>");


    }
}
```

![filter_result](/assets/images/2020-09-21-17-20-03_2020-09-21-jsp_learning_04.md.png){: .alignCenter}

위와 같이, html로 지정되었기 때문에, <br> 태그가 보이지 않게 되었고,  
한글도 깨지지 않는 모습을 확인할 수 있다.

다음 포스팅에선, application, session, cookie에 대해서 다뤄보자.

