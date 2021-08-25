# Spring MVC 1

### 2. 서블릿

1. #### HttpServletRequest

   - 역할

      : 서블릿은 개발자가 HTTP요청 메시지를 편하게 사용하도록 해당메시지를 파싱하고, 

        HttpServletRequest객체에 담아서 제공한다(임시저장소기능 -> http요청이 시작-끝까지)

     - 저장 : request.setAttribute(name, value)
     - 조회 : request.getAttribute(name)
     - 세션관리 : request.getSesstion(create: true)

2. HttpServletRequest 기본사용법

   -  HttpServletRequest 로 HTTP 메시지의 **정보조회**를 할 수 있다 .
   
   ![image](https://user-images.githubusercontent.com/68681443/129648721-342419ee-316a-4ae0-8e15-fac94cc5af87.png)
   
   
   
3. HTTP 요청 데이터 조회방법(클라이언트 -> 서버로 데이터 전달방법)

   - **Get - 쿼리파라미터(바디에 데이터 보내지않음)**   ex) 검색, 필터, 페이징

     - **단일파라미터**, 전체파라미터, 복수파라미터

     - request.getParameter(key값)

       ![image](https://user-images.githubusercontent.com/68681443/129651372-36eed3bf-6142-43b6-8aef-d55684f0cd15.png)

   - **Post-메시지바디에 쿼리파라미터 형식으로 전달**   ex) 회원가입, 상품주문, HTML form

     - Content-Type에 form으로 전달, **쿼리파라미터형식**으로 전달되었다는 정보 확인가능

       ![image](https://user-images.githubusercontent.com/68681443/129649169-8989b58e-de18-4778-b41f-95aa24d7c7c8.png)

     - 이 방식또한 쿼리파라미터를 보내는 형식이기때문에 **request.getParameter(key값)**으로 확인가능

     

   - **HTTP message body에 데이터를 직접 담아서 요청**

     - rest API에 주로사용 - 데이터형식은 JSON 선호

     - POST, PUT, PATCH

       

     - 단순텍스트를 POST로 전송 👉 **request.getInputStream()**으로 바이트코드를 얻을 수 있다

       ![image](https://user-images.githubusercontent.com/68681443/129654819-66b3e20d-caeb-4a9f-8d84-c9e592e5394c.png)

     - JSON형식의 텍스트 전송 👉 **jackson library사용**

       👉단순 텍스트로 읽어와서, Json형태로 파싱할 수 있도록 만든 객체로 가져온다

       ![image](https://user-images.githubusercontent.com/68681443/129659083-2e67368e-e30e-410b-80e3-54042dd224be.png)

     

4. HttpServletResponse 기본사용법

   - Http응답메시지 생성, 헤더생성, 바디생성

   - 편의기능 : Content-type, 쿠키 , redirect

   -  HttpServletResponse 로 HTTP 메시지의 정보를 기입할 수 있다

     ![image](https://user-images.githubusercontent.com/68681443/129669127-f3c47eb6-dd45-4680-ae79-eb57107eed20.png)

   

5. HTTP응답 데이터

   - **단순텍스트응답** - writer.print("ok")

   - **HTML응답**

     - HTML을 반환할 경우 content-type을 text/html로 지정해야함

       ![image](https://user-images.githubusercontent.com/68681443/129669864-363d3275-201d-40f3-b528-65552295b3e1.png)

   - **HTTP API** - MessageBody JSON

     - JSON을 반환할 경우 content-type을 application/json으로 지정해야함

     ![image](https://user-images.githubusercontent.com/68681443/129671817-3bcc7ad2-976b-42f0-948f-ffe9ffd6e8cd.png)

   

