# Spring MVC 1

### 4. MVC프레임워크 만들기

1. #### 프론트 컨트롤러 패턴

   - 프론트컨트롤러 서블릿 하나로 클라이언트 요청받음(입구를 하나로!)
   - 프론트컨트롤러가 요청에 맞는 컨트롤러 찾아서 호출
   - 공통처리가능
   - 프론트컨트롤러를 제외한 나머지 컨트롤러는 서블릿사용하지 않아도 됨

   ![image](https://user-images.githubusercontent.com/68681443/130757037-c89e0531-d823-434e-8641-3c4784ddafeb.png)

   

2. #### V1 - 프론트컨트롤러 도입

   - 여러개의 컨트롤러를 만들고, 프론트컨트롤러로 서블릿 요청이들어오면 그에맞는 컨트롤러로 연결

   - 각 컨트롤러에 연결된 jsp를 재사용할 수 있음

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

     

