---
# author: cjlee
categories: jsp
# cover: /assets/covers/coding.png
date: "2020-09-11T16:45:00Z"
tags: ['null']
title: '# 0. JSP 학습기록 - Intellij JSP 프로젝트 생성'
---

# Prerequisite
1. Intellij **Ultimate**
2. JDK
3. Tomcat
4. Environment Variable Settings

# Create
:

1) Welcome 화면에서, 새 프로젝트(New Project) 만든다.  
   
2) Java - Project SDK에서 JDK를 설정하고,

3) Additional Libraries and Frameworks에서 JAVE EE - Web Application - WebService를 체크하고 Next 버튼을 누른다..?


![failed](/assets/images/2020-09-11-16-47-45_2020-09-11-jsp_project_with_intellij.md.png){: .alignCenter}

왠지 모르겠는데 나는 없다.

[공식 문서](https://www.jetbrains.com/help/idea/preparing-to-develop-a-web-service.html)에도 이렇게 하라고 하는데, 나는 없다.

각고의 구글링 끝에, [나와 비슷한 문제가 있던 질문 글을 발견](https://intellij-support.jetbrains.com/hc/en-us/community/posts/360009485660-No-Java-EE-tab-in-New-Project-)

얘기인즉, 좌측 Java Enterprise로 가서, SDK를 설정하고 Next 버튼을 누르면, 이를 선택할 수 있는 옵션이 나온다.

(추가)  
[스택오버플로우](https://stackoverflow.com/questions/63606475/how-to-create-a-new-java-ee-project-in-intellij-2020-2-without-gradle-or-maven/63606566#63606566)에 따르면, 2020.2 버전부터 사라졌다고 한다. 다음을 참고하자

---

> It's no longer possible. The only option is to downgrade to 2020.1 version.
> 
> The previous wizard was using obsolete technologies and frameworks with the libraries hosted on JetBrains servers always lagging updates and extremely hard to maintain. Now > we moved to Gradle/Maven projects where all the dependencies can be fetched from the public servers and the project can use up-to-date Java EE and library versions.

___


![success](/assets/images/2020-09-11-16-49-58_2020-09-11-jsp_project_with_intellij.md.png){: .alignCenter}

4) Web Profile을 체크한 후 Next.


![framework](/assets/images/2020-09-11-16-52-27_2020-09-11-jsp_project_with_intellij.md.png){: .alignCenter}

5) 적당히 이름을 붙여주고 프로젝트 생성

6) 상단의 Run - Edit Configurations 클릭

7) 좌측 상단의 + 버튼 누른 후, Tomcat Server - local 클릭

![tomcat](/assets/images/2020-09-11-16-58-01_2020-09-11-jsp_project_with_intellij.md.png){: .alignCenter}

8) 위 사진처럼, Application Server의 Configure 를 클릭하여 톰캣이 설치된 위치 설정
   
9) 우측 하단의 Fix 클릭 -> artifact를 projectName:war 로 선택

10) Deployment 탭으로 넘어와서, Application context를 /ProjectName_war 를 /로 변경

11) webapp 우클릭 -> new -> JSP/JSPX로 index.jsp 생성
    
12) body 태그 사이에 Hello world! 작성
![](/assets/images/2020-09-11-17-05-16_2020-09-11-jsp_project_with_intellij.md.png){: .alignCenter}

13) Run & 브라우저 열어서 localhost:8080

![done](/assets/images/2020-09-11-17-04-17_2020-09-11-jsp_project_with_intellij.md.png){: .alignCenter}

잘 된다. 중간에 이상했던 부분때문에 꽤 애를 먹었다 ㅠ.

if 안될 경우.. Check list
1. JDK 경로 설정 확인
2. Tomcat 경로 설정 확인
3. 환경변수 확인(cmd.exe -> java -version , javac -version)
4. 기존에 Tomcat을 실행시켰는지 확인
5. 8080 port가 이미 사용중인지 확인.

끝!