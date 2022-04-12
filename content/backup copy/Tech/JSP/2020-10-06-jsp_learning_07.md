---
# author: cjlee
categories: jsp
# cover: /assets/covers/coding.png
date: "2020-10-06T22:21:00Z"
tags: ['null']
title: '# 7. JSP 학습기록 - MVC 패턴과 JSP Model 1, Model 2'
---

# 들어가며

: 지난 포스팅에서, Jasper 라는 도구를 이용하여 HTML Code에 직접 자바코드를 때려박는 방법을,  
그리고 이렇게 생성되는 파일을 JSP 라고 부른다는 것을 배웠다.  

그렇다면, 기존의 Java Code는 다 그대로 JSP로 옮겨버려도 될까?  
흠.. 뭔가 그렇지는 않을 것 같은데, 명확히 이유를 짚어내기는 어렵다.

이번 포스팅에서는 그 "이유"에 대해서 알아보자.

# 코드 구분의 필요성
: 우선, 다음과 같이, 기본적인 덧셈, 뺄셈을 계산해주는 코드를 만들어보자.

```jsp
<html>
<head>
    <title>calculator</title>
</head>

<body>
    <div>
        <form action="Adder.jsp">
            <div>
                <label> first_number </label>
                <input name="n1" type="text">
            </div>
            <div>
                <label> second_number </label>
                <input name="n2" type="text">
            </div>
            <input type="submit" name="calc" value="add"/>
            <input type="submit" name="calc" value="minus"/>

        </form>
    </div>
    <%
        int num1 = 0;
        int num2 = 0;

        String num1_ = request.getParameter("n1");
        String num2_ = request.getParameter("n2");

        if(num1_ != null && !num1_.equals("")) {
            num1 = Integer.parseInt(num1_);
        }
        if(num2_ != null && !num2_.equals("")) {
            num2 = Integer.parseInt(num2_);
        }

        String op = request.getParameter("calc");
        if (op == null || op.equals("")) {
            // do nothing
        }
        else if (op.equals("add")) {
    %>
    result of add is <%= num1+num2 %>
    <% } else { %>
    result of minus is <%= num1-num2 %>
    <% } %>
</body>

</html>
```

테스트 해보면, 다음과 같다.

![](/assets/images/2020-10-08-21-05-04_2020-10-06-jsp_learning_07.md.png)

{: .caption}
초기 화면

![](/assets/images/2020-10-08-21-05-22_2020-10-06-jsp_learning_07.md.png)

{: .caption}
덧셈 결과

![](/assets/images/2020-10-08-21-05-41_2020-10-06-jsp_learning_07.md.png)

{: .caption}
뺄셈 결과  

---

> Note. 실수로 이름을 Adder.jsp 라고 지었는데, 이후 Calculator.jsp 로 수정하였다.

---

두개의 입력을 받기 전에는 0으로 두고,  
하나 혹은 두개의 값을 입력받으면, 두, 세 번째와 같이 결과를 출력한다.

그러나, 이 정도로 간단한 코드는 괜찮지만,  
만약 내가 실제로 어떤 서비스를 제공하고자 한다면,  
그 코드의 길이는 얼마나 될까?

당장 이 글을 쓰고 있는 Jekyll 의 Layout만 보더라도,  
Liquid를 몇번 사용하고나면, **굉장히 가독성이 떨어진다.**

또한,  JSP의 뼈대가 되는 HTML 의 길이 또한 꽤 되는데,  
여기에 Java Code까지 추가하면 **얼마나 길어질까?**

이를 해결하기 위한, 다시 말해 코드를 구분하기 위한 방법론은 크게 두 가지가 있으며,  
이를 각각 Model 1, Model 2 라고 부른다.

MVC 패턴이 무엇인지에 대해서 먼저 잠깐 짚고, 그리고 Model 1, 2에 대해서 알아보자.

# MVC Pattern

![](/assets/images/2020-10-08-21-51-17_2020-10-06-jsp_learning_07.md.png)

: MVC pattern이란, 소프트웨어 공학에서 제시하는 소프트웨어 개발 방법론 중 하나이다.   
즉, "이런 식으로 개발하면, 알차고 우아하게 짤 수 있습니다" 라고 제시하는 방법론 중 하나이다.  
MVC 에서 각각의 M, V, C가 의미하는 바는 다음과 같다.

* M : Model - Controller에 의해 제어되어, View로 전달되는 객체
* V : View - 사용자에게 보여지는 UI, 결과물.
* C : Controller - 사용자의 요청을 적절히 제어해서, View로 명령을 보냄.

이러한 식으로 프로젝트를 구성하면, UI와 비즈니스 로직을 분리해서,  
서로의 변경에 의한 영향을 최소화하면서 수정할 수 있다는 장점이 있다.

<!-- 앞으로 설명할 Model 1, Model 2에 대해서 읽을 때에,   
**"곱셈, 나눗셈을 추가해야 한다면 코드 작성에 얼마나 차이가 있을까?"**  
를 염두해보자. -->

# Model 1
![](/assets/images/2020-10-08-21-07-01_2020-10-06-jsp_learning_07.md.png)

: Model 1은, 쉽게 말해서 단순히 코드만을 구분짓는 것을 의미한다.
JSP page로 들어온 요청에 대해서, 요청 및 로직을 처리하는 행위를 해당 JSP가 책임지고 핸들링한다.

그러나, **"가독성"** 을 위해서,(다시말해 유지보수를 위해서)  
요청을 받은 부분과, 로직을 처리하는 부분만을 구분해서 흐름을 제어하는 것이 Model 1의 특징이다.

역시, 말로 하면 잘 이해가 안된다. 위에서 보여줬던 예시를 수정하면 다음과 같다.

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    int num1 = 0;
    int num2 = 0;

    String num1_ = request.getParameter("n1");
    String num2_ = request.getParameter("n2");

    if(num1_ != null && !num1_.equals("")) {
        num1 = Integer.parseInt(num1_);
    }
    if(num2_ != null && !num2_.equals("")) {
        num2 = Integer.parseInt(num2_);
    }

    String op = request.getParameter("calc");
    String outString = " ";
    if (op == null || op.equals("")) {
        // do nothing
    } else if (op.equals("add")) {
        outString = "result of " + op + " is " + (num1+num2);
    } else {
        outString = "result of " + op + " is " + (num1-num2);
    }
%>

<html>
<head>
    <title>calculator</title>
</head>

<body>
<div>
    <form action="Adder.jsp">
        <div>
            <label> first_number </label>
            <input name="n1" type="text">
        </div>
        <div>
            <label> second_number </label>
            <input name="n2" type="text">
        </div>
        <input type="submit" name="calc" value="add"/>
        <input type="submit" name="calc" value="minus"/>

    </form>
</div>

<%= outString %>

</body>

</html>
```

바뀐 점은 다음과 같다.  

1. parameter를 int형으로 변환하는 과정을, 위쪽으로 옮겼다.
2. if else문을 위쪽에서 처리하고, HTML 태그에서는 곧바로 출력만 하도록 했다.

이렇게 함으로서, 3개의 코드블럭을 2개로 줄일 수 있었다.  
또한, 곱셈, 나눗셈에 대해서 처리할 때에도 크게 귀찮지는 않을 것이다.  

또한, MVC 패턴을 따랐다고 볼 수도 있다.  
**위쪽의 Code Block이 Controller**  
**아래쪽의 HTML 코드가 View**  
**outString이 Model**  
이 된다.

그러나, 여전히 문제점은 남아 있다.

1. 파일이 하나이기 때문에, 다른 사람과 **협업**을 하려고 한다면, 어려움을 크게 겪을 것이다.
2. 만약 한 page에 대해서 처리하는 양이 많다면, page에 대한 요청이 들어올 때마다,
   **무거운** servlet으로 **변환하고**, **컴파일**하는 행위를 **반복**해야 한다.

# Model 2

![](/assets/images/2020-10-08-21-11-36_2020-10-06-jsp_learning_07.md.png)

: Model 2의 형태로 구성하기 위해서는 우선, Model 1에서  
**"코드를 물리적으로 구분하는 것"**이 첫번째다.

```java
@WebServlet("/calculator")
public class Adder extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        int num1 = 0;
        int num2 = 0;

        String num1_ = req.getParameter("n1");
        String num2_ = req.getParameter("n2");

        if(num1_ != null && !num1_.equals("")) {
            num1 = Integer.parseInt(num1_);
        }
        if(num2_ != null && !num2_.equals("")) {
            num2 = Integer.parseInt(num2_);
        }

        String op = req.getParameter("calc");
        String outString = "";
        if (op == null || op.equals("")) {
            // do nothing
        } else if (op.equals("add")) {
            outString = "result of " + op + " is " + (num1+num2);
        } else {
            outString = "result of " + op + " is " + (num1-num2);
        }
    }
}
```

그러므로 기존의 코드블럭을 모두 **잘라내서**, calculator.java에 **집어넣자**.
그런데, 이렇게만 하면 아무것도 할 수 없다.

calculator.jsp 에서 calculator.java 로 넘어왔으면,  
여기서 받은 값을 다시 calculator.jsp 에서 **"지속"** 할 수 있어야 하지 않을까?  

이 때 사용할 수 있는 것이 **dispatcher** 라는 녀석의 **forward** 다.

```java
// request라는 저장소에 outString을 "outString" 이라는 이름으로 저장.
req.setAttribute("outString", outString); 


RequestDispatcher dispatcher = req.getRequestDispatcher("calculator.jsp");
dispatcher.forward(req, resp);
```

---

> Note. 다른 페이지로 넘겨주는 방식에는, redirect와, forward가 있다.  
> redirect를 통해 calculator.jsp 로 넘겨주면, **"새로운"** calculator servlet이 생성된다.
> 따라서, 유지한 상태를 **넘겨주기** 위해서는 forward를 통해서 값을 넘겨줘야 한다.

---

이렇게 forward로 넘겨받은 값을, jsp 에서 활용하면 된다.

```jsp
<html>
<head>
    <title>calculator</title>
</head>

<body>
<div>
    <!-- calculator.jsp에서 calculator로 바꿔줘야 한다 -->
    <form action="calculator">
        <div>
            <label> first_number </label>
            <input name="n1" type="text">
        </div>
        <div>
            <label> second_number </label>
            <input name="n2" type="text">
        </div>
        <input type="submit" name="calc" value="add"/>
        <input type="submit" name="calc" value="minus"/>
    </form>
</div>

<%= request.getAttribute("outString") %>
</body>
</html>
```

여기서 잠깐,  
`<%= request.getAttribute("outString") %>`  
이 녀석을, 지난 시간에 배웠던 **EL** 이라는 녀석으로 대체하자.  
`${outString}`

이렇게 하면, jsp에 있던 모든 코드블럭을 없앰으로써,  
**여전히 JSP, 즉 동적 페이지로서의 기능은 하면서**  
**MVC 패턴을 지켰기 때문에 유지보수에도 용이한**  
코드를 작성할 수 있다.

이와 더불어, Model 1 에서 겪었던 단점도 해결하였다.

1. 파일이 하나이기 때문에, 다른 사람과 **협업**을 하려고 한다면, 어려움을 크게 겪을 것이다.   
   ==> **파일 자체를 구분하였기 때문에, 협업에 용이하다**
2. 만약 한 page에 대해서 처리하는 양이 많다면, page에 대한 요청이 들어올 때마다,
   **무거운** servlet을 **만들고**, **컴파일**하는 행위를 **반복**해야 한다.  
   ==> Controller, Dispatcher는 미리 Compile되어있고, **JSP만 Servlet**으로 변환하면 된다.


# 정리
: 기존의 Code Block이 있던 JSP, Model 1, Model 2 로 옮겼을때를 짧게 정리하면 다음과 같다.

| | 기존 | Model 1 | Model 2 |
|:--:|:--:|:--:|:--:|
|개발편의성(러닝커브)|좋음|보통|나쁨|
|유지보수(가독성)|나쁨|보통|좋음|
|적합한규모|소|소|대|



