## HTTP 요청/응답 흐름
- 클라이언트는 요청(request) 를 보내고
- 서버는 해당 요청을 받아 알맞은 응답(response) 를 한다.

## Servlet : HTTP 를 편리하게 이용
- HTTP 에서 전송되는 데이터는 문자열을 형태로 전송된다.
- 이를 편리하게 사용하게 해주는 서블릿(Servlet) 기술을 이용하게 된다.

```java
public class HelloServlet extends HttpServlet {
    protected void service(HttpServletRequest request, HttpServletResponse response){
        ...
    }
}
```

## 서블릿 컨테이너
- 여러개의 서블릿을 가지고 있는 상자의 역할
- 여러개의 서블릿 중에서 요청에 알맞은 서블릿을 호출해 준다.

## 서블릿 동작 방식
1. 스프링 부트가 동작하면서 내장 톰캣 서버를 실행
2. 내장 톰캣 서버는 서블릿 컨테이너 기능을 가지고 있다. → 서블릿을 생성
3. HTTP 요청 메세지를 기반으로 `request` `response` 객체를 생성
4. 서블릿에 전달 후 비지니스 로직을 수행 후
5. response 를 응답 메세지로 클라이언트에 전달

## 서블릿의 문제점
- Java 코드 중심으로 작성하기 때문에 HTML 작성이 어렵다.
- 비지니스 로직까지 같이 있는 경우 파일이 너무 커진다.

```java
Member member = new Member(username, age);
memberRepository.save(member);

response.setContentType("text/html");
response.setCharacterEncoding("utf-8");
PrintWriter w = response.getWriter();
w.write("<html>\n" +
        "<head>\n" +
        "    <meta charset=\"UTF-8\">\n" +
        "</head>\n" +
        "<body>\n" +
        "성공\n" +
        "<ul>\n" +
        "    <li>id="+member.getId()+"</li>\n" +
        "    <li>username="+member.getUsername()+"</li>\n" +
        "    <li>age="+member.getAge()+"</li>\n" +
        "</ul>\n" +
        "<a href=\"/index.html\">메인</a>\n" +
        "</body>\n" +
        "</html>");
```

## 템플릿 엔진 JSP
- HTML 중심으로 코드를 작성하기 편리하며, 필요한 부분만 자바 코드를 넣을 수 있다
- 템플릿 엔진에는 JSP, Thymeleaf, Freemarker, Velocity 등이 있다.

```html
<!-- main/webapp/jsp/members/save.jsp -->
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page import="hello.servlet.domain.member.Member" %>
<%@ page import="hello.servlet.domain.member.MemberRepository" %>
<%
    //request, response 사용 가능
    MemberRepository memberRepository = MemberRepository.getInstance();

    System.out.println("MemberSaveServlet.service");
    String username = request.getParameter("username");
    int age = Integer.parseInt(request.getParameter("age"));

    Member member = new Member(username, age);
    memberRepository.save(member);

%>
<html>
<head>
    <title>Title</title>
</head>
<body>
성공
<ul>
    <li>id=<%=member.getId()%></li>
    <li>username=<%=member.getUsername()%></li>
    <li>age=<%=member.getAge()%></li>
</ul>
<a href="/index.html">메인</a>
</body>
</html>
```

## 서블릿 & JSP 의 문제점
- 서블릿에서 동적인 HTML 을 JSP 를 통해서 좀 더 편리하게 작성 가능하다.
- 하지만 데이터를 조회하는 리포지토리 등등 다양한 코드가 모두 JSP에 노출되어 있다.
- 즉, JSP가 너무 많은 역할을 한다. → 유지보수가 어려워진다.

## MVC 패턴의 등장
- 비즈니스 로직은 서블릿 처럼 다른곳에서 처리하고
- `JSP`는 목적에 맞게 HTML로 View (= 화면) 그리는 일에 집중.
- 결론적으로 유지보수가 편리해 진다.

## MVC = Model View Controller
- MVC 패턴은 지금까지 학습한 것 처럼 하나의 서블릿이나, JSP로 처리하던 것을
- 컨트롤러(Controller)와 뷰(View)라는 영역으로 서로 역할을 나눈 것을 말한다.

## Model
- 뷰에 출력할 데이터를 담아둔다. 뷰가 필요한 데이터를 모두 모델에 담아서 전달해주는 덕분에
- 뷰는 비즈니스 로직이나 데이터 접근을 몰라도 되고, 화면을 렌더링 하는 일에 집중할 수 있다.

## View
- 모델에 담겨있는 데이터를 사용해서 화면을 그리는 일에 집중한다. 여기서는 HTML을 생성

## Controller
- HTTP 요청을 받아서 파라미터를 검증하고, 비즈니스 로직을 실행한다.
- 뷰에 전달할 결과 데이터를 조회해서 모델에 담는다.
- 컨트롤러에 비즈니스 로직을 둘 수도 있지만, 이렇게 되면 컨트롤러가 너무 많은 역할을 담당
- 그래서 일반적으로 비즈니스 로직은 서비스(Service)라는 별도의 계층을 만든다.
- 컨트롤러는 비즈니스 로직이 있는 서비스를 호출하는 담당.