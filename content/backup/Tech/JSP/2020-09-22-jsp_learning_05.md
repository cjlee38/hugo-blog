---
# author: cjlee
categories: jsp
# cover: /assets/covers/coding.png
date: "2020-09-22T17:26:00Z"
tags: ['null']
title: '# 5. JSP 학습기록 - Array 와 Application, Session, Cookie'
---

# 배열 다루기.
: 원래 이번 편에서 곧바로 Application / Session / Cookie 에 대해서 다루려고 했는데,  
생각해보니 데이터를 배열로 주고 받는 법에 대해서 다루지 않아서, 잠깐 짚고 가자.

[지난 편](http://cjlee38.github.io/jsp/jsp_learning_04)에서 다뤘던 firstServlet.html은 다음과 같다.
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div>
    <form action="firstServlet" method="POST">
        <div>
            <label> input your title </label>
            <input name = "title" type="text"/>
        </div>
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
</body>
</html>
```
여기서, 대충 title을 두 개를 넣고 싶다고 해보자.  
*소제목으로 구분하자* 뭐 이런게 아니라,   
그냥 **말 그대로 똑같은 title이라는 input**을 하나 더 추가해보자는 것이다.

간단하게, 

```html
<div>
    <label> input your title </label>
    <input name = "title" type="text"/>
</div>

```
요 코드를 복사해서 그 바로 밑에 붙여넣으면 된다.

그러면, firstServlet에서 첫 번째 title을 받을까, 두 번째 title을 받을까 ?  
이 질문에 대한 내 대답은 "배열로 받으면 되지, 왜 그걸 고민해" 이다.

기존의 `req.getParameter("title");` 에서, 6글자만 추가하면 된다.  

`req.getParameterValues()`  

그러면, 여러개의 Parameter가 넘어오므로, String의 배열로 받아주고,  
이를 for문을 돌려서 출력해보자.

코드는 다음과 같다.

```java
@WebServlet("/firstServlet")
public class firstServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter out = resp.getWriter();

        String[] title = req.getParameterValues("title");
        String content = req.getParameter("content");

        for (int i=0; i<title.length; i++) {
            out.println(i + " 번째 타이틀 : " + title);
        }

        out.println("컨텐츠 : " + content);
    }
}

```

그리고, **localhost:8080/firstServlet.html** 로 들어가서 대충 작성해보자.  

---

![array_submit](/assets/images/2020-09-22-19-55-14_2020-09-22-jsp_learning_05.md.png){: .alignCenter}

---

![array_result](/assets/images/2020-09-22-19-55-02_2020-09-22-jsp_learning_05.md.png){: .alignCenter}

---

잘 동작하는 모습을 볼 수 있다.

# Application, Session, Cookie
## 이론
: Application, Session, Cookie 는 셋 모두   
어떤 **상태를 유지**하기 위하여,   
다시 말해 **"데이터를 보존하기 위해서"** 사용한다.    
그러나, 같은 목적을 가지고 있더라도, 다른 방법을 지향하기 때문에, 상황에 따라 적절히 사용해야 한다.


| |Application|Session|Cookie|
|:--:|:--:|:--:|:--:|
|사용범위| 전역 범위 | *세션 범위 | 특정 URL 범위(여러 개 가능) |
|생명주기| WAS의 시작&종료와 함께 | 세션의 시작&종료와 함께 | TimeOut까지 |
|저장위치| WAS 메모리 | WAS의 메모리 | Client의 Memory 혹은 File |

*세션 범위 : 특정 사용자만 사용할 수 있는 범위

가령, Cookie 같은 경우, **데이터가 Client에게 저장되어 있기 때문에,**  
Client가 **악의적인 목적으로 이를 수정하는 것을 막기 위해**,  
Application 혹은 Session을 활용해야 한다.  
혹은, **오랫동안 갖고 있어야 할** (중요치 않은) 어떤 Data가 있다면, 이는 Cookie로 저장해야지,   
Session이나 Application에 갖고 있다면 감당하지 못할 것이다.  

또한, 어떤 Data를 Application 으로 설정했을 경우,  
다른 사용자가 접속했을 때 해당 Data를 볼 수 있으므로, 이러한 경우를 방지하기 위해  
Session을 사용해야 한다.  
비유하자면, Application은 전역변수고, Session은 지역변수라고 볼 수 있다.

___ 

> Note. 위 개념을 잡기 위해, [얄코님의 영상](https://youtu.be/OpoVuwxGRDI)에 큰 도움을 받았다.  
> 헷갈리는 사람은 이를 참고하자.

---

## 활용
### Application
: Application을 사용하려면, 다음과 같이 작성하면 된다.  
```java
ServletContext app = req.getServletContext();
```
그리고, HashMap의 put을 사용하는 것처럼, 다음과 같이 사용하면 된다.  
```java
app.setAttribute("value", someValue);
```

### Session
: Session 또한, Application과 비슷하다.
```java
HttpSession session = req.getSession();
session.setAttribute("value", somevalue);
```

### Cookie
: Cookie는 위 두가지와 약간 궤를 달리하기 때문에, 사용법 또한 조금 다르다.
단순히 put()만 해주면 안되고, 쿠키 객체를 만들어서 이를 resp에 덧붙여줘야 한다.
```java
// 기존 Cookie 읽기
Cookie[] cookies = req.getCookies();

// Cookie 만들어서 추가하기
Cookie cookie = new Cookie("value", someValue) // key value
resp.addCookie(cookie);

// Cookie 경로 설정
cookie.setPath("/"); // 모든 Path
cookie.setPath("/firstServlet"); // firstServlet 이 포함된 Path
```

전부 다 하기엔 시간적 여유가 없으니,  
Cookie만 다뤄보자.

예상 Flow는 다음과 같다.

* localhost:8080/firstServlet.html에서 입력받은 Title Array를, 각각 Title_1, Title_2 라고 이름 붙여서 쿠키에 추가한다.
* lolcahost:8080/firstServlet 페이지에서, 쿠키를 확인한다.
* Path는 "/firstServlet" 으로 정한다. 따라서, 캐시된 쿠키를 지우고 다른 페이지를 가면 쿠키가 없어야 한다.

firstServlet.java
```java
@WebServlet("/firstServlet")
public class firstServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter out = resp.getWriter();

        String[] title = req.getParameterValues("title");
        String content = req.getParameter("content");

        for (int i=0; i<title.length; i++) {
            out.println(i + " 번째 타이틀 : " + title[i]);
        }

        Cookie cookie_1 = new Cookie("Title_1", title[0]);
        Cookie cookie_2 = new Cookie("Title_2", title[1]);

        cookie_1.setPath("/firstServlet");
        cookie_2.setPath("/firstServlet");

        resp.addCookie(cookie_1);
        resp.addCookie(cookie_2);
    }
}
```

![cookie_result](/assets/images/2020-09-22-20-34-59_2020-09-22-jsp_learning_05.md.png){: .alignCenter}

먼저, 정상적으로 작동하는 모습을 볼 수 있다.
캐시된 쿠키를 지우고 루트 페이지로 가보자.

![cookie_result](/assets/images/2020-09-22-20-37-41_2020-09-22-jsp_learning_05.md.png){: .alignCenter}

아무리 새로고침을 해봐도 쿠키는 들어오지 않는다.

