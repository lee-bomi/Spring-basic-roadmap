# Spring MVC 1

## 6. 스프링MVC  - 기본기능

1. ### 로깅

   - 로킹 라이브러리 :  SLF4J(인터페이스)+Logback(구현체-기본사용)
     - 로그선언 : private Logger log = LoggerFactory.getLogger(getClass()); , @Slf4j 사용가능

     - 로그호출 : log.info("hello")

     - 로그레벨 : TRACE > DEBUG(개발서버) > INFO(운영서버) > WARN > ERROR

     - 로그레벨설정 (application.properties)

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
          * 기본요청, 요청url을 배열화 가능(어떤 url로 요청을해도 ok를 확인할 수 있음)
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

   - @RequestHeader **MultiValueMap<String, String>** headerMap : 모든 http헤더를 조회(하나의키에 여러값받음)

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

     

6. HTTP요청 파라미터 - @RequestParam

   - **@RequestParam** 표시방법

     ```java
     @Slf4j
     @Controller
     public class RequestParamController {
     
         //기본형1
         @RequestMapping("/request-param-v1")
         public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
             String username = request.getParameter("username");
             int age = Integer.parseInt(request.getParameter("age"));
             log.info("username={}, age={}", username, age);
     
             response.getWriter().write("ok");
         }
     	//기본형2
         @ResponseBody   //@RestController가 아닌 @Controller사용시 return값을 본문에 표시하려면 						@ResponseBody를 해당 메서드에 사용한다
         @RequestMapping("/request-param-v2")
         public String RequestParamV2(
                 @RequestParam("username") String username,
                 @RequestParam("age") int age
                 ){
             log.info("username={}, age={}", username, age);
             return "ok";
         }
     	// ✔추천형 -------------------------------------------------------------------------------
         @ResponseBody
         @RequestMapping("/request-param-v3")
         public String RequestParamV3(
                 @RequestParam String username,  //요청파라미터와 변수명이 같다면 생략가능
                 @RequestParam int age
                 ){
             log.info("username={}, age={}", username, age);
             return "ok";
         }//---------------------------------------------------------------------------------------
     	//너무 축약되어서 파라미터로 받는지 모를 수 있다
         @ResponseBody
         @RequestMapping("/request-param-v4")
         public String RequestParamV4(String username,int age){
             log.info("username={}, age={}", username, age);
             return "ok";
         }
     }
     ```

   - **required** 사용방법 (이 값이 무조건 들어와야해! 여부)

     - require=true(기본값)일때 파라미터로 값이 안들어오면 에러남
     - 에러 안나게하려면 required=false 처리!

     ```java
     @ResponseBody
         @RequestMapping("/request-param-required")
         public String RequestParamRequired(
                 @RequestParam(required = true) String username,
                 @RequestParam(required = false) Integer age  //요청파라미터가 없어도 에러안남
             											 //(bad request 400에러)
                 ){
             log.info("username={}, age={}", username, age);
             return "ok";
         }
     ```

   - **defaultValue** 사용방법(기본값 설정)

     - 파라미터에 담긴값이 없을경우 defaultValue로 설정한 값이 들어옴

     ```java
     @ResponseBody
         @RequestMapping("/request-param-default")
         public String RequestParamdefault(
                 @RequestParam(defaultValue = "guest") String username,
                 @RequestParam(defaultValue = "-1") Integer age     
             	//defaultValue를 설정한경우, 파라미터에 담긴값이 없는경우 표시된다
         ){
             log.info("username={}, age={}", username, age);
             return "ok";
         }
     ```

   - parameter의 값을 Map으로 한번에 받고싶을때

     ```java
      @ResponseBody
         @RequestMapping("/request-param-map")
         public String RequestParamMap(@RequestParam Map<String, Object> paramMap){
     
             log.info("username={}, age={}", paramMap.get("username"), paramMap.get("age"));
             return "ok";
         }
     ```

     

7. HTTP요청 파라미터 - @ModelAttribute

   - 실제 개발에서는 요청파라미터를 받아서 필요한 객체를 만들고, 그 객체에 값을 넣어주어야한다

     ```java
     @ResponseBody
     @RequestMapping("/model-attribute-v1")
     public String modelAttributeV1(@RequestParam String username, @RequestParam int age) {
         HelloData helloData = new HelloData();
         helloData.setUsername(username);
         helloData.setAge(age);
         log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
         return "ok";
     }
     ```

   - 하지만 @ModelAttribute는 이 과정을 자동화 해준다

     - 1️⃣ HelloData객체 생성
     - 2️⃣ 요청 파라미터의 이름으로HelloData객체의 프로퍼티를 찾아서 setter호출하여 파라미터의 값을 바인딩

     ```java
     @ResponseBody
     @RequestMapping("/model-attribute-v2")
     public String modelAttributeV2(@ModelAttribute HelloData helloData) {//HelloData객체 생성, 															       요청파라미터가 모두들어가있음
         log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
         log.info("helloDate={}", helloData);
         return "ok";
     }
     ```

     | 📌 스프링 컨트롤러의 파라미터에서 생략시 아래와 같은 규칙을 갖는다 |
     | ------------------------------------------------------------ |
     | 👉 String, int, Integer과 같은 단순타입 = @RequstParam        |
     | 👉 나머지 = @ModelAttribute                                   |

     | 📌 **프로퍼티?**                                              |
     | ------------------------------------------------------------ |
     | : 객체안에 getUsername, setUsername이 있다면 이 객체는 username이라는 프로퍼티를 갖고있다고 말함 |

     
     
     🔰지금까지 get, post, html form 으로 전송되는 값을 받는방법을 알아봄🔰
     
     

8. HTTP요청 메시지 -  단순텍스트

   ```java
   @Slf4j
   @Controller
   public class RequestBodyStringController {
   
       @PostMapping("/request-body-string-v1")
       public void RequestBodyString (HttpServletRequest request, HttpServletResponse response) throws IOException {
           ServletInputStream inputStream = request.getInputStream();
           String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
           log.info("messageBody={}", messageBody);
           response.getWriter().write("ok");
       }
   
       //servlet을 다쓸필요도없는데!!귀찮다!!
       @PostMapping("/request-body-string-v2")
       public void RequestBodyStringV2 (InputStream inputStream, Writer responseWriter) throws IOException {
           String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
           log.info("messageBody={}", messageBody);
           responseWriter.write("ok");
       }
   
       //inputstream도 안쓸꺼야!!!!!!! => http 메세지컨버터
       @PostMapping("/request-body-string-v3")
       public HttpEntity<String> RequestBodyStringV3 (HttpEntity<String> httpEntity) throws IOException {
           //Http메세지컨버터동작 = HttpEntity가 Httpbody에 있는걸 String으로 바꿔서 넣어줄께!
           //String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8); 이코																				드 자동동작
           String body = httpEntity.getBody();
           log.info("messageBody={}", body);
           return new HttpEntity<>("ok");
       }
   
       //entity쓰는것도귀찮아!! => @RequestBody어노테이션으로 대체하자!
       @ResponseBody
       @PostMapping("/request-body-string-v4")
       public String RequestBodyStringV4 (@RequestBody String messageBody) throws IOException {
           log.info("messageBody={}", messageBody);
           return "ok";
       }
   }
   ```

   ✨ 요청파라미터(쿼리스트링, html form데이터로 전달되는 데이터)를 컨트롤러에서 받을때

   ​		=> @RequestParam(primitive type), @ModelAttribute(객체타입)

   ✨ 메시지 바디를 직접조회 및 응답

   ​		=> @HttpEntity / **@RequestBody**(단순텍스트)

   

9. HTTP요청 메시지 - JSON

   - objectMapper : 가져온 messageBody를 자바객체로 변환

   - @RequestBody에 직접만든 객체 지정가능! (HttpEntity, @RequestBody를 사용하면 Http메시지 컨버터가 HTTP메시지 바디의 내용(단순text, json모두지원)을 내가 원하는 문자나 객체로 변환해준다)

   - @RequestBody는 생략❌ : 생략시 @ModelAttribute로 인식됨(그래서 파라미터 처리로직으로 바뀜)

     

     ```java
     @Slf4j
     @Controller
     public class RequestBodyJsonController {
     
         ObjectMapper objectMapper = new ObjectMapper();
     	//기본형
         @PostMapping("/request-body-json-v1")   //HTTP header-ContentType이 jason인지 확인
         public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) 																throws IOException {
         ServletInputStream inputStream = request.getInputStream();
         String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
     
             log.info("messageBody={}", messageBody);
             HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
             log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
     
             response.getWriter().write("ok");
         }
     
         //inputStream쓰기싫다!
         @ResponseBody
         @PostMapping("/request-body-json-v2")   //"HTTP header-ContentType이 jason인지 확인"
         public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {
     
             log.info("messageBody={}", messageBody);
             HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
             log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
     
             return "ok";
         }
     
         //objectMapper쓰기싫다!(json을 문자로 변환하고, 다시 json으로 변환하기싫다)
         //@RequestBody에 직접만든 객체 지정가능!!---------------------------------------------------
         @ResponseBody
         @PostMapping("/request-body-json-v3")  
         public String requestBodyJsonV3(@RequestBody HelloData data) throws IOException {
             log.info("username={}, age={}", data.getUsername(), data.getAge());
             return "ok";
         }//---------------------------------------------------------------------------------------
     
         //이렇게 써도 상관없음!
         @ResponseBody
         @PostMapping("/request-body-json-v4")
         public String requestBodyJsonV4(HttpEntity<HelloData> httpEntity) throws IOException {
             HelloData helloData = httpEntity.getBody();
             log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
             return "ok";
         }
     
         //응답할때 Http메시지 바디에 직접 넣어줄 수 있음
         @ResponseBody
         @PostMapping("/request-body-json-v5")
         public HelloData requestBodyJsonV5(@RequestBody HelloData data) throws IOException {
             log.info("username={}, age={}", data.getUsername(), data.getAge());
             return data;    //응답할때도 객체타입으로 반환가능
         }
     }
     
     ```

     | @Request요청 : JSON요청 -> HTTP메시지 컨버터 -> 객체         |
     | ------------------------------------------------------------ |
     | **@ResponseBody응답** : 객체 -> HTTP메시지 컨버터 -> JSON응답 |

     

10. ### 응답 - 정적 리소스, 뷰 템플릿

    - @ResponseBody가 있으면 -> HTTP Body에 글자 그대로 박힘

    - @ResponseBody가 없으면 -> return값의 논리이름으로 뷰를 찾고 렌더링함

      ```java
      @Controller
      public class responseViewController {
      
          @RequestMapping("/response-view-v1")
          public ModelAndView responseViewV1() {
              ModelAndView mv = new ModelAndView("response/hello")
                      .addObject("data", "hello!!");
              return mv;
          }
      	// ✔추천방법
          @RequestMapping("/response-view-v2")
          public String responseViewV2(Model model) {
              model.addAttribute("data", "hello~!");
              return "response/hello";   			 //@Controller일때 view의 논리적 이름이 된다!
          }
      }
      ```

      

11. ### HTTP응답 - HTTP API, 메시지 바디에 직접 입력

    - HTTP 메시지바디에 JSON같은 형식의 데이터를 실어 보내는 것(정적리소스 or 뷰 템플릿 거치지❌)

      ``` java
      @Controller
      public class ResponseBodyController {
      
          @GetMapping("/response-body-string-v1")
          public void responseBodyV1(HttpServletResponse response) throws IOException {
              response.getWriter().write("ok");
          }
      
          @GetMapping("/response-body-string-v2")
          public ResponseEntity<String> responseBodyV2(HttpServletResponse response) {
              return new ResponseEntity<>("ok", HttpStatus.OK);
          }
      
          @ResponseBody
          @GetMapping("/response-body-string-v3")
          public String responseBodyV3() {
              return "ok";
          }
      
          @GetMapping("/response-body-json-v1")   //HTTP컨버터를 통해 JSON형식으로 변환되어 반환됨
          public ResponseEntity<HelloData> responseBodyJsonV1() {
              HelloData helloData = new HelloData();
              helloData.setUsername("userA");
              helloData.setAge(20);
              return new ResponseEntity<>(helloData, HttpStatus.OK);
          }
      
          @ResponseStatus(HttpStatus.OK)  //위의 방법처럼 상태코드 반환이 없으므로 어노테이션으로 사용
          @ResponseBody
          @GetMapping("/response-body-json-v2")
          public HelloData responseBodyJsonV2() {
              HelloData helloData = new HelloData();
              helloData.setUsername("userA");
              helloData.setAge(20);
              return helloData;
          }
      }
      ```

      

12. HTTP메시지 컨버터

13. 요청 매핑 핸들러 어뎁터쿠조

    