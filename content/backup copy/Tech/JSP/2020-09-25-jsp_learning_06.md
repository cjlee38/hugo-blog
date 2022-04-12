---
# author: cjlee
categories: jsp
# cover: /assets/covers/coding.png
date: "2020-09-25T16:21:00Z"
tags: ['null']
title: '# 6. JSP 학습기록 - doGet, doPost와 JSP, 그리고 EL(Expression Language) '
---


# doGet, doPost
: 기존에는 Servlet에 코드를 작성할 때, service()만 사용하고,   
get과 post를 구분하지 않았었다. 
당연히, get과 post를 구분하는 방법이 있을텐데, 우선적으로 생각할 수 있는 방법은  
if else 문으로 분기하는 것이다. 또한, 이에 걸맞은 method 도 존재한다.

```java
if (req.getMethod().equals("GET")) {
    // do something for get
} else if (req.getMethod().equals("POST")) {
    // do something for post
}
```

그러나, 이렇게 작성하는 것은 당연하게도 우아해보이지 않는다.  
이에 걸맞은 method가 있는데, 바로 **doGet()**, **doPost()** method 다.

```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    //do something for get
}

@Override
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    // do something for post
}
```

그렇다면, service() 함수와 어떤 관계를 가질까?

우리가 작성한 Servlet이 상속받은 **HttpServlet 클래스**를 찾아가보면, 다음과 같은 코드를 볼 수 있다.

```java
protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String method = req.getMethod();
        long lastModified;
        if (method.equals("GET")) {
            lastModified = this.getLastModified(req);
            if (lastModified == -1L) {
                this.doGet(req, resp);
            } else {
                long ifModifiedSince = req.getDateHeader("If-Modified-Since");
                if (ifModifiedSince < lastModified) {
                    this.maybeSetLastModified(resp, lastModified);
                    this.doGet(req, resp);
                } else {
                    resp.setStatus(304);
                }
            }
        } else if (method.equals("HEAD")) {
            lastModified = this.getLastModified(req);
            this.maybeSetLastModified(resp, lastModified);
            this.doHead(req, resp);
        } else if (method.equals("POST")) {
            this.doPost(req, resp);
        } else if (method.equals("PUT")) {
            this.doPut(req, resp);
        } else if (method.equals("DELETE")) {
            this.doDelete(req, resp);
        } else if (method.equals("OPTIONS")) {
            this.doOptions(req, resp);
        } else if (method.equals("TRACE")) {
            this.doTrace(req, resp);
        } else {
            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[]{method};
            errMsg = MessageFormat.format(errMsg, errArgs);
            resp.sendError(501, errMsg);
        }

    }
```

자세한건 잘 알아보지 못하더라도,   
대충 **각 http 방식에 걸맞는 do method를 실행**하는 것을 볼 수 있다.

따라서, 우리가 service() 함수를 작성하고,   
그 안에 **super.service(req, resp)**로   
부모 클래스인 HttpServlet의 service() 함수를 실행하면,  
service() 함수가 일종의 "Filter" 역할을 해줄 수 있는 것이다.

앞서 우리가 봤던 Servlet Filter는 특정 Path에 대해서 걸 수도 있지만,  
그보다 조금 더 **좁은 개념**으로 **service() 함수**를 활용할 수 있다.

# JSP with Jasper
: 지금까지 학습했던 내용들은, JSP 가 아니라 단순히 Servlet을 만드는 과정이었다.  
따라서, HTML 문서는 나름대로 웹페이지로서의 기능을 할 수 있었지만, 대신 정적인 페이지일 뿐이었고,  
Servlet으로 작성한 코드는 `out.println()` 과 같은, 소위 텍스트와 비슷한 결과물만을 볼 수 있었다.

우리가 원하는 것은 이것이 아니다. **WAS** 를 사용하는 목적은, **"동적인 페이지"**를 만드는 것이다.

`out.println()` 을 통해서도, 그 안에 태그를 붙여줌으로써 **동적인 HTML 문서를** 만들 수 있다.  
그러나, 그 모습은 다음과 같이 정말 **괴랄**하기 짝이 없다.  

```java
@WebServlet("/firstServlet")
public class firstServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter out = resp.getWriter();
        
        out.write("<!DOCTYPE html>");
        out.write("<html lang=\"en\">");
        out.write("<head>");
        out.write("<meta charset=\"UTF-8\">");
        out.write("<title>Title</title>");
        
        // 중략 

        out.write("</body>");
        out.write("</html>");        
    }
}

```

이런 단순 반복 행위에 불과한 코드를 작성하는 행위는, 사람이 아니라 기계에게 맡기는 것이 어떨까?  
애초에 기계라는 것이 이러한 단순반복을 대신 수행하기 위해 등장한 것이니 말이다.

이 때 바로 **JSP** 라는 것이 등장한다. 

JSP는 Java Server Page의 약자로서,   
이 JSP의 형태는 **HTML에 Java Code를 작성할 수 있는 모습이다.**

Jekyll 기반의 블로그를 운영하면서 배웠던, **Liquid라는 문법이 하는 일과 굉장히 흡사**한데,  
대신 Liquid는 Programming Language가 아니었지만, JSP는 Jasper라는 도구를 이용해  
Java 코드를 해석하고, 위 괴상한 코드를 자동으로 생성해줌으로써 최종적으로 **Servlet의 구색**을 갖출 수 있게 해준다.

일례로, 다음과 같은 모습을 볼 수 있다.

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<% for (int i=0; i<15; i++) { %>
<body>
hello world! <br/>
</body>
<% } %>

</html>
```

![JSP_sample](/assets/images/2020-09-25-17-36-48_2020-09-25-jsp_learning_06.md.png){: .alignCenter}

`<% %>` 이런식으로 생긴 녀석 *(=코드블럭)* 으로 코드를 감쌀 수 있다.  
이렇게 **"이 문장은 HTML이 아닌 Java 코드다"** 라는 것을 알려줌으로서,  
for문을 15번을 반복시켜서 Hello world! 를 15번을 출력할 수 있다.  

이러한 JSP 코드의 문법은 다음과 같다.

|이름|작성법|목적|예시|
|:--:|:--:|:--:|:--:|
|스크립트릿(Scriptlet)|<% %>|일반적인 변수 할당 등의 script|<% String name = "홍길동"; %>|
|선언문(Declarations)|<%! %>|함수 선언|<%! public void someFunction() { ... } %>
|지시자(Directivs)|<%@ %>| 라이브러리 임포트, 혹은 페이지 메타데이터 설정|<%@ page pageEncoding="UTF-8" ... %>
|표현식(Expressions)|<%= %> | Expression 출력 | <%= someFunction() %> 입니다.(태그 내) |

위에서 실행한 JSP 파일이, 어떻게 Jasper에 의해 .java 파일로 변환되었는지 살펴보자.  
(설정이 꼬였는지, 정확히 위 파일이 존재하지는 않고, **이전 포스팅에서 작성했던 index.jsp**를 살펴볼 수 있었다.)

```java
// 윗부분 생략
try {
      response.setContentType("text/html;charset=UTF-8");
      pageContext = _jspxFactory.getPageContext(this, request, response,
      			null, true, 8192, true);
      _jspx_page_context = pageContext;
      application = pageContext.getServletContext();
      config = pageContext.getServletConfig();
      session = pageContext.getSession();
      out = pageContext.getOut();
      _jspx_out = out;

      out.write("\r\n");
      out.write("\r\n");
      out.write("<html>\r\n");
      out.write("<head>\r\n");
      out.write("    <title>Title</title>\r\n");
      out.write("</head>\r\n");
      out.write("<body>\r\n");
      out.write("\r\n");
      out.write("    This is index!\r\n");
      out.write("</body>\r\n");
      out.write("</html>\r\n");
    } catch (java.lang.Throwable t) {
      if (!(t instanceof javax.servlet.jsp.SkipPageException)){
        out = _jspx_out;
        if (out != null && out.getBufferSize() != 0)
          try {
            if (response.isCommitted()) {
              out.flush();
            } else {
              out.clearBuffer();
            }
          } catch (java.io.IOException e) {}
        if (_jspx_page_context != null) _jspx_page_context.handlePageException(t);
        else throw new ServletException(t);
      }
    } finally {
      _jspxFactory.releasePageContext(_jspx_page_context);
    }
```



그러나, 이렇게 JSP 파일에 몰아서 작성했을 경우 발생할 수 있는 단점은, **JSP 코드 자체가 급격하게 보기에 나빠진다는 것**이다.  

작성된 JSP 또한 결국 jasper에 의해 Servlet으로 변환되므로,  
기존에 Servlet에서 사용하던 여러 변수 *(e.g. Parameter로 넘겨받은 HttpServletRequest req)* 들을 사용할 수 있는데,  
loop 문이라던지, 각 변수마다 `<%= request.getAttribute("something") %>` 를 작성하게 되면  
보기도 좋지 않고, **유지보수하기도 힘들 것**이 분명하다.

따라서, 다음 포스팅에 이야기할 **MVC model 1, 그리고 model 2가 등장**하게 되는데,  
일단 거기에 앞서 **EL, Expression Langauge**에 대해서 잠깐 짚고 가자.

# EL (Expression Language)
: 다음 포스팅에서 작성할 MVC model에 대해서 이해하고 나면, **이 EL이 왜 쓰이는지 더더욱 명확하게** 알 수 있다.  
EL은, 흡사 Javascript의 Template Literal과 비슷하게 생겼는데,  
이런식으로 작성한다.

```
${something}
```

그냥 저렇게 작성하는 것만으로, `<%= request.getAttribute("something") %>` 를 대체할 수 있다.

EL 문법을 정리하면, 다음과 같다.

### 사용법 

|Data Type|JSP|EL|
|:--:|:--:|:--:|
|Primitive|request.getAttribute("value");|${value}|
|Array(or list)|request.getAttribute("list").get(0);|${list[0]}|
|Map|request.getAttribute("map").get("key")|${map.key} 혹은 ${map["key"]}|

### 변수 탐색 범위
: pageContext Scope -> request Scope -> session Scope -> application Scope
(다음 포스팅에서 설명)

### 연산자
: 파이썬과 겹치는 부분이 좀 있다.

|연산자 구분|기호|EL|
|:--:|:--:|:--:|
|비교연산자|>|gt(greater than)|
|비교연산자|<|lt(less than)|
|비교연산자|>=|ge(greater or equal)|
|비교연산자|<=|le(less or equal)|
|산술연산자|/|div(divide)|
|산술연산자|%|mod(modulo)|
|관계연산자|==|eq(equal)|
|관계연산자|!=|ne(not equal)|
|논리연산자|&&|and|
|논리연산자|││|or|
|부호연산자|!|not|
|-|-| empty (isEmpty())|

반드시 EL 연산자로 작성할 필요는 없지만,   
문제가 생길 것을 대비해(e.g. '<','>' 기호가 Tag로 인식된다던지),   
EL 연산자로 작성하는 것을 권장한다.



