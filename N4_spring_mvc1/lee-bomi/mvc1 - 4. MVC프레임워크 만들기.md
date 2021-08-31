# Spring MVC 1

## 4. MVC프레임워크 만들기

1. ### 프론트 컨트롤러 패턴

   - 프론트컨트롤러 서블릿 하나로 클라이언트 요청받음(입구를 하나로!)
   - 프론트컨트롤러가 요청에 맞는 컨트롤러 찾아서 호출
   - 공통처리가능
   - 프론트컨트롤러를 제외한 나머지 컨트롤러는 서블릿사용하지 않아도 됨

   ![image](https://user-images.githubusercontent.com/68681443/130757037-c89e0531-d823-434e-8641-3c4784ddafeb.png)

   

2. ### V1 - 프론트컨트롤러 도입

   - 여러개의 컨트롤러를 만들고, 프론트컨트롤러로 서블릿 요청이들어오면 그에맞는 컨트롤러로 연결

   -  jsp를 재사용할 수 있음

     ```JAVA
     @WebServlet(name = "frontControllerServletV1", urlPatterns = "/front-controller/v1/*")
     public class FrontControllerServletV1 extends HttpServlet{
     
         private Map<String, ControllerV1> controllerMap = new HashMap<>();
     	
         //컨트롤러를 KEY, VALUE로 저장
         public FrontControllerServletV1() {
             controllerMap.put("/front-controller/v1/members/new-form", new MemberFormControllerV1());
             controllerMap.put("/front-controller/v1/members/save", new MemberSaveControllerV1());
             controllerMap.put("/front-controller/v1/members", new MemberListControllerV1());
         }
     
         @Override
         protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
             System.out.println("FrontControllerServletV1.service");
     		
             //요청된 URI정보를 추출
             String requestURI = request.getRequestURI();
             
     		//추출한 URI정보와 저장된 컨트롤러값이 일치하는지 확인
             ControllerV1 controller = controllerMap.get(requestURI);
             if (controller == null) {
                 response.setStatus(HttpServletResponse.SC_NOT_FOUND);
                 return;
             }
     
             controller.process(request, response);
         }
     }
     ```

     

3. ### V2 - view분리

   - view를 띄우기위해 중복코드를 제거하기 위한 객체를 만든다(MyView)

   ![image](https://user-images.githubusercontent.com/68681443/130887446-58d9bdb9-6cc0-42e0-9b6f-3fd7567a0a5f.png)

   - 인터페이스 - 컨트롤러의 반환값은 MyView 타입

     ```java
     public interface ControllerV2 {
         MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
     }
     ```

   - FrontControllerServlet2 

     ```java
     @WebServlet(name = "frontControllerServletV2", urlPatterns = "/front-controller/v2/*")
     public class FrontControllerServletV2 extends HttpServlet {
     
         private Map<String, ControllerV2> controllerMap = new HashMap<>();
     
         public FrontControllerServletV2() {
             controllerMap.put("/front-controller/v2/members/new-form", new MemberFormControllerV2());
             controllerMap.put("/front-controller/v2/members/save", new MemberSaveControllerV2());
             controllerMap.put("/front-controller/v2/members", new MemberListControllerV2());
         }
     
         @Override
         protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
             System.out.println("FrontControllerServletV2.service");
     
             String requestURI = request.getRequestURI();
     
             ControllerV2 controller = controllerMap.get(requestURI);
             if (controller == null) {
                 response.setStatus(HttpServletResponse.SC_NOT_FOUND);
                 return;
             }
     
             MyView view = controller.process(request, response);
             view.render(request,response);
     
         }
     }
     ```

     👉view를 객체화해서 컨트롤러가 myview를 반환하면, 그 반환값에서 .render로 view를 렌더링하는방식

     1️⃣프론트컨트롤러에서 uri로 넘어온값에 매칭되는 컨트롤러를 연결

     2️⃣해당컨트롤러에서 파라미터받고, 서비스호출, 모델에 데이터보관, 뷰를 던짐

     3️⃣이때 던지는 뷰는 MyView타입으로  던져서, 그 뷰타입에서 render()메서드 실행

     

4. ### V3 - Model

   - 서블릿 종속성 제거 - HttpServletRequest, HttpServletResponse 제거, 별도 Model객체 반환
   - 뷰이름 중복 제거 - 물리이름❌ 논리이름⭕(뷰의 폴더위치 변경시 프론트컨트롤러만 수정하면됨)

   ![image](https://user-images.githubusercontent.com/68681443/130924067-7018cbb8-4f59-4e4c-923b-bbd66f2af13c.png)

   

   [ModelView] - 임시저장소인 model, view이름 전달하는 객체생성

   ```java
   @Getter @Setter
   public class ModelView {
   
       private String viewName;
       private Map<String, Object> model = new HashMap<>();
   
       public ModelView(String viewName) {
           this.viewName = viewName;
       }
   }		
   ```

   [FrontControllerServletV3] - model에 정보저장하고, view정보를 리턴하는 기능구현

   - frontcontroller는 서블릿을쓰므로, 거기서 request의 파라미터값을 받아 map에 담아준다

     ```java
     @WebServlet(name = "frontControllerServletV3", urlPatterns = "/front-controller/v3/*")
     public class FrontControllerServletV3 extends HttpServlet {
     
         private Map<String, ControllerV3> controllerMap = new HashMap<>();
     
         public FrontControllerServletV3() {
             controllerMap.put("/front-controller/v3/members/new-form", new MemberFormControllerV3());
             controllerMap.put("/front-controller/v3/members/save", new MemberSaveControllerV3());
             controllerMap.put("/front-controller/v3/members", new MemberListControllerV3());
         }
     
         @Override
         protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
             System.out.println("FrontControllerServletV3.service");
     
             //1.컨트롤러 조회
             String requestURI = request.getRequestURI();
     
             ControllerV3 controller = controllerMap.get(requestURI);   
             if (controller == null) {
                 response.setStatus(HttpServletResponse.SC_NOT_FOUND);
                 return;
             }
     
             //2.컨트롤러 호출/ modelview반환
             Map<String, String> paramMap = createParamMap(request); //파라미터가져오는 메서드
             ModelView mv = controller.process(paramMap);    //해당컨트롤러에 데이터 전달
     
             String viewName = mv.getViewName();
     
             //3. viewResolver호출 및 view반환
             MyView view = viewResolver(viewName);
     
             //4. render(model)호출
             view.render(mv.getModel(),request,response);
     
         }
     
         //논리이름을 넣으면, 물리이름을 반환하는 메서드
         private MyView viewResolver(String viewName) {
             return new MyView("/WEB-INF/views/" + viewName + ".jsp");
         }
     
         //request parameter의 모든값을 가져오는 메서드
         private Map<String, String> createParamMap(HttpServletRequest request) {
             Map<String, String> paramMap = new HashMap<>();
             request.getParameterNames().asIterator()
                     .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
             return paramMap;
         }
     }
     ```

     

5. ### V4 - 단순하고 실용적인 컨트롤러

   - V3은 서블릿 종속성을제거하고, 중복을 제거하는 등 잘 설계되었지만 ModelView를 생성하고 반환하는게 번거롭다.(실용성이 떨어짐)
   - modelview가 아닌 view를 반환

   ![image](https://user-images.githubusercontent.com/68681443/130930354-d083effc-bc54-4f06-91a2-98f9b0a06c2c.png)

   - [ControllerV4] - model을 파라미터로 전달

   ```java
   public interface ControllerV4 {
       String process(Map<String, String> paramMap, Map<String, Object> model);
   }
   ```

   

6. ### V5 - 유연한 컨트롤러1

   - ControllerV3, ControllerV4를 동시에 쓰고싶다면?

     -> 현재는 각각 다른 인터페이스를 따르는 클래스를 썼으므로, 동시에 불가능하다!(**어답터의 필요**)

     

7. ### V6 - 유연한컨트롤러2

8. 
