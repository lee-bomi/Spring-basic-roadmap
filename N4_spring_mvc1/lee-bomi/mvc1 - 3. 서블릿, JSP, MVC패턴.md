# Spring MVC 1

### 3. 서블릿, JSP, MVC패턴

1. #### 서블릿으로 회원관리 웹 어플리케이션 만들기

   - 서블릿 + 자바코드만으로 HTML을 만듦

   - 서블릿 => 동적으로 원하는 HTML을 만들수있지만, HTML코드를 비효율적으로 코딩

     👉자바코드안에 서블릿을 사용하여 HTML을 넣는방법은 비효율 적임**(템플릿엔진의 필요성)**

     

2. #### JSP로 회원관리 웹 어플리케이션 만들기

   - 코드의 절반은 회원정보를 저장하기 위한 비즈니스로직(모든코드가 노출)

   - 하위절반이 HTML 로 구성된 뷰 영역

   - JSP가 너무많은 역할을 함(MVC패턴의 필요성)

     

3. #### MVC패턴 개요

   - Controller : http요청받아 파라미터검증, 비즈니스로직실행, view에 전달할 결과데이터를 모델에 담음

   - Model : 뷰에 출력할데이터 담음

   - View : 모델에 담긴 데이터를 화면에 그리는 역할(html생성)

     

4. #### MVC패턴 적용

   -request.setAttribute()를 model처럼 사용!

   ​		[JSP에서 원래 이렇게 받아야함]

   - ```html
     <ul>
         <li>id=<%=((Member)request.getAttribute("member")).getId()%></li>
         <li>username=<%=((Member)request.getAttribute("member")).getUsername()%></li>
         <li>age=<%=((Member)request.getAttribute("member")).getAge()%></li>
     </ul>		
     ```

     [JSP가 제공하는 **표현식** 사용]

     => 내부에선 request.getAttribute().getId로 값을 가져옴

   - ```html
     <ul>
         <li>id=${member.id}</li>
         <li>username=${member.username}</li>
         <li>age=${member.age}</li>
     </ul>
     ```

     [기존엔 JSP에 모든 로직 다 때려박음]

   - ```html
     <%@ page import="hello.servlet.domain.member.Member" %>
     <%@ page import="hello.servlet.domain.member.MemberRepository" %>
     <%@ page contentType="text/html;charset=UTF-8" language="java" %>
     <%
         //response, request는 지원가능(jsp하부는 servlet으로 구동하기때문)
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

     [위와달리 서블릿이 Controller역할을 충실히 해냄]

   - ```java
     @WebServlet(name = "mvcMemberSaveServlet", urlPatterns = "/servlet-mvc/members/save")
     public class MvcMemberSaveServlet extends HttpServlet {
     
         private MemberRepository memberRepository = MemberRepository.getInstance();
     
         @Override
         protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
             //1. 파라미터 받고
             String username = request.getParameter("username");
             int age = Integer.parseInt(request.getParameter("age"));
     
             //2. 서비스로직 호출하고
             Member member = new Member(username, age);
             memberRepository.save(member);
     
             //3. Model에 데이터를 보관해서
             request.setAttribute("member", member);
     
             //4. view로 던져서 마무리
             String viewPath = "/WEB-INF/views/save-result.jsp";
             RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);  //컨트롤러->뷰
             dispatcher.forward(request, response);  //서버내부에서 호출
         }
     }
     ```

     [MVC패턴으로 인해 View가 간결해짐]

   - ```html
     <%@ page contentType="text/html;charset=UTF-8" language="java" %>
     <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>     <!--반복문을 쓰기위함/ java코드 하나도없이 깔끔하게 구현가능-->
     
     <html>
     <head>
         <title>Title</title>
     </head>
     <body>
     <a href="/index.html">메인</a>
         <table>
             <thead>
             <th>id</th>
             <th>username</th>
             <th>age</th>
             </thead>
             <c:forEach var="item" items="${members}">
                 <tr>
                     <td>${item.id}</td>
                     <td>${item.username}</td>
                     <td>${item.age}</td>
                 </tr>
             </c:forEach>
         </table>
     </body>
     </html>
     ```

     

5. #### MVC패턴 한계

   ❌ 컨트롤러부분에 중복된 코드가 많다(view로 던져지는 부분)

   👉 공통기능을 처리하기위한 **프론트컨트롤러패턴**이필요함(중구난방 진입x, 수문장 역할) 

   📌 **스프링MVC의 핵심 = 프론트컨트롤러**

   

   

