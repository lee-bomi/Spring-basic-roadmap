# Spring MVC 1

## 6. 스프링MVC  - 기본기능

1. ### 로깅

   - 로킹 라이브러리 :  SLF4J(인터페이스)+Logback(구현체-기본사용)
     - 로그선언 : private Logger log = LoggerFactory.getLogger(getClass()); , @Slf4j 사용가능

     - 로그호출 : log.info("hello")

     - 로그레벨 : TRACE > DEBUG(개발서버) > INFO(운영서버) > WARN > ERROR

     - 로그레벨설정 

       ```java
       #전체 로그 레벨 설정(기본 info)
       logging.level.root=info
           
       #hello.springmvc 패키지와 그 하위 로그 레벨 설정
       logging.level.hello.springmvc=debug
       ```

     - 로그사용법

       ```java
       log.debug("data={}", data)
       ```

     - 로그사용의 장점

       - 쓰레드정보, 시간, 클래스이름과 같은 부가정보 볼 수있다

       - 로그를 상황(디버깅, 운영)에 맞게 출력할 수 있다

       - 성능이 일반 sysout보다 좋다

         

2. ### 요청매핑

   - 기본매핑

   - ```java
     @RestController
     public class MappingController {
     
         private Logger log = LoggerFactory.getLogger(getClass());
     
         /**
          * http메서드 모두허용(method 작성안할경우)
          */
         @RequestMapping("/hello-basic")
         public String helloBasic() {
             log.info("hellobasic");
             return "ok";
         }
      	/**
          * 기본요청, 요청url을 배열화 가능(어떤 url로 요청을해도 ok를 확인할 수 )
          */
         @RequestMapping({"/hello-basic", "/hello-ok"})
         public String helloBasic2() {
             log.info("hellobasic2");
             return "ok";
         }
     }
     ```

   - PachVariable(경로변수)사용

     ```java 
         @GetMapping("/mapping/{userId}")
         public String mappingPath(@PathVariable("userId") String data) {
             log.info("mappingPath userId={}", data);
             return "ok";
         }
     ```

     

3. ### 요청매핑 - API 예시

   ```JAVA
   @RestController
   @RequestMapping("/mapping/users")
   public class MappingClassController {
   
       @GetMapping
       public String user() {
           return "get users";
       }
   
       @PostMapping
       public String addUser() {
           return "post user";
       }
   
       @GetMapping("/{userId}")
       public String findUser(@PathVariable String userId) {
           return "get userId=" + userId;
       }
   
       @PatchMapping("/{userId}")
       public String updateUser(@PathVariable String userId) {
           return "update userId=" + userId;
       }
   
       @DeleteMapping("/{userId}")
       public String deleteUser(@PathVariable String userId) {
           return "delete userId=" + userId;
       }
   }
   ```

   

4. ### Http요청 - 기본, 헤더조회

   - 어노테이션기반의 스프링 컨트롤러는 다양한 파라미터를 지원
   - HTTP 헤더 정보를 조회하는 방법은 아래와 같다

   ```java
   @Slf4j
   @RestController
   public class RequestHeaderController {
   
       @RequestMapping("/headers")
       public String headers(HttpServletRequest request,
                             HttpServletResponse response,
                             HttpMethod httpMethod,
                             Locale locale,
                             @RequestHeader MultiValueMap<String, String> headerMap,
                             @RequestHeader("host") String host,
                             @CookieValue(value = "myCookie", required = false) String cookie
                           ) {
           log.info("request={}", request);
           log.info("response={}", response);
           log.info("httpMethod={}", httpMethod);
           log.info("locale={}", locale);
           log.info("headerMap={}", headerMap);
           log.info("header host={}", host);
           log.info("myCookie={}", cookie);
   
           return "ok";
       }
   }
   ```

   - @RequestHeader **MultiValueMap<String, String>** headerMap : 모든 http헤더를 조회(하나의키게 여러값받음)

   - @RequestHeader("host") String host : 특정 http헤더 조회

     

5. ### HTTP요청 파라미터 - 쿼리파라미터, HTML form

   - 클라이언트->서버로 데이터를 전송하는방법

     - GET, POST, HTTP message body로 전송

     - GET : 쿼리파라미터에 /request-param-v1?username=bomi&age=32로 전송하면  해당로그 찍힘

     - POST : static/basic/hello-form.html에서 post로 요청해본다

       (static은 외부에 공개됨 (index.html이 기본으로 공개됨))

       ```html
       <html>
       <head>
         <meta charset="UTF-8">
         <title>Title</title>
       </head>
       <body>
       <form action="/request-param-v1" method="post">
         username: <input type="text" name="username" />
         age: <input type="text" name="age" />
         <button type="submit">전송</button>
       </form>
       </body>
       </html>
       ```

       ```java
       @Slf4j
       @Controller
       public class RequestParamController {
       
           @RequestMapping("/request-param-v1")
           public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
               String username = request.getParameter("username");
               int age = Integer.parseInt(request.getParameter("age"));
               log.info("username={}, age={}", username, age);
       
               response.getWriter().write("ok");
           }
       }
       ```

       ![image](https://user-images.githubusercontent.com/68681443/131473760-f4a3bcde-aa47-4234-86e1-907d2d7574c7.png)

   - post로 보낸 값들이 log에 잘 찍힌다

     

6. ㄴ