---
# author: cjlee
categories: jsp
# cover: /assets/covers/coding.png
date: "2020-09-19T15:22:00Z"
tags: ['null']
title: '# 3. JSP 학습기록 - Servlet을 이용한 입출력'
---

# Output

: [지난 2편](https://cjlee38.github.io/jsp/jsp_learning_02) 에서 작성했던 코드는 다음과 같다.

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

___

> Note. 다른 작업을 하기 전에, [여기](https://cjlee38.github.io/jsp/jsp_project_with_intellij) 를 참고해서,  
> **IDE의 도움**을 받자. 하나하나 직접 작성하는 것도 꽤 수고스러운 일이다.  
> 학생의 경우 인증을 통해 **무료로 Ultimate 버전**을 구할 수 있으나,
> 불가능 할 경우 Eclipse를 사용하면 된다. Eclipse 설정 방법은 Intellij와 조금 다르니 유의할 것.

___

위 코드를, Project/src/main/java 하위에 새로 만들고, **이름을 firstServlet** 으로 변경하였다.

![project_structure](/assets/images/2020-09-21-10-36-39_2020-09-19-jsp_learning_03.md.png){: .alignCenter}
{: .caption}


그리고, 여기서 이제 브라우저, 즉 클라이언트 쪽으로 print 를 하기 위해서는, 다음과 같이 사용하면 된다.

```java
public class firstServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter out = resp.getWriter();
        out.println("This is first servlet.");
    }
}
```

역시, 지난 편처럼, web.xml을 수정해줘야 하는데,  
web.xml 을 수정하는 것은 설정파일을 수정하는 것이므로, 귀찮기도 할 뿐더러,  
해당 코드가 어디에 위치해있는지 찾아 나가는 것 또한 수고스러우므로,  

**annotation** (@) 을 통해, **코드 내에서 직접 연결**해줄 수 있다.  

따라서, `public class firstServlet extends HttpServlet` 위에,   
`@WebServlet("/firstServlet")` 이라고 넣어주자.

![firstServletCode](/assets/images/2020-09-21-10-41-24_2020-09-19-jsp_learning_03.md.png){: .alignCenter}

{: .caption}
이런식으로

그리고, IDE의 run 버튼을 통해 Tomcat을 실행시켜서 결과를 보자.

![firstServletResult_1](/assets/images/2020-09-21-10-42-12_2020-09-19-jsp_learning_03.md.png){: .alignCenter}

처음에는 이렇게, index.jsp를 바탕으로 한 Hello world! 가 나타난다.   
url 뒤에, /firstServlet을 붙여서 찾아가보자.

![firstServletResult_2](/assets/images/2020-09-21-10-43-31_2020-09-19-jsp_learning_03.md.png){: .alignCenter}

위와 같이, 결과가 잘 나타나는 모습을 볼 수 있다.  

# Input

: 이제 출력은 해봤으니, 입력도 한번 받아보자.

입력을 받는 방법은 두 가지가 있다.  
1. url에 query string을 직접 넣어주는 방법.
2. html 에서 입력을 받는 form을 만들어서, query string으로 자동으로 넘겨주는 방법.

2번 안에 1번이 포함되어 있으므로, 1번을 먼저 해보고, 2번을 진행해보자.

먼저, 기존의 주소인 `http://localhost:8080/firstServlet` 의 뒤에,  
`?count=3` 을 붙여보자.  현재로서는, 아무런 변화도 없다. 우리는
즉, 최종 url은 `http://localhost:8080/firstServlet?count=3` 이 된다.

현재로서는, servlet에서 무언가를 처리하는 것이 없기 때문에 당연히 아무런 변화도 없다.  
servlet으로 넘어와서, 코드를 다음과 같이 수정해보자.

```java
@WebServlet("/firstServlet")
public class firstServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter out = resp.getWriter();

        int iCount = 1;
        String sCount = req.getParameter("count"); 
        
        if (sCount != null && !sCount.equals("")) {
            iCount = Integer.parseInt(sCount);
        }
        
        for(int i=0; i<iCount; i++) {
            out.println("This is first Servlet.");
        }
    }
}
```

기본 iCount 라는 int 변수를 1로 초기화 시켜놓고, req에서 getParameter()를 통해 전달받은 count를 받는다.

이 떄, 다음을 주의해야 한다.
1. req.getParameter()로 받은 변수는 반드시 String으로 넘어온다.
2. req.getParameter()를 했을 때, 반드시 "숫자로 파싱 가능한 String이 넘어올 것이라는 보장은 없다.

1번은 Integer.parseInt() 등으로 파싱하면 되지만, 2번의 경우 다음과 같은 케이스들이 있을 수 있다.

1) /fristServlet?cnt=something 인 경우 -> String "something"이 넘어옴  
2) /firstServlet?cnt= 인 경우 -> ""(빈 문자열)이 넘어옴  
3) /firstServlet? 인 경우 -> null이 넘어옴  
4) /firstServlet 인 경우 -> null이 넘어옴  

**2-1번의 경우는 제외**하고, 나머지 케이스를 핸들링하기 위해, **null**과 **빈 문자열인 ""**를 검사해준다.  
정상적으로 Integer 형으로 parsing이 되면, 해당 count만큼 출력이 될 것이다. 실행해보자.

![firstServlet_inputResult](/assets/images/2020-09-21-15-19-38_2020-09-19-jsp_learning_03.md.png){: .alignCenter}

잘 나오는 것을 볼 수 있다.

여기서, 조금만 더 나가서, 사용자가 url이 아닌 html을 통해서 입력을 받을 수 있도록 해보자.

먼저, webapp 폴더 하위에, firstServlet.html 을 새로 만들고, 다음의 내용을 body 안에 넣어보자.

```html
<div>
    <form action="firstServlet">
        <div>
            <label> input your count</label>
        </div>
        <input type="text" name="count" />
        <input type="submit" value="click here to submit"/>
    </form>
</div>
```

그리고, 톰캣을 다시 실행시킨 뒤, firstServlet.html 로 이동하면, 다음과 같은 화면이 보일 것이다.

![firstServlet_html](/assets/images/2020-09-21-15-46-46_2020-09-19-jsp_learning_03.md.png){: .alignCenter}

여기에, 내가 원하는 임의의 숫자 값을 입력하고, submit을 해보자.

![firstServlet_html_result](/assets/images/2020-09-21-15-48-12_2020-09-19-jsp_learning_03.md.png){: .alignCenter}

잘 나타나는 것을 볼 수 있다.

이렇게 전달되는 값은, get 함수를 통해 요청된 것이다.

다음 편에서는, post 요청을 받고, 이를 파싱하는 방법에 대해서 알아보자.

