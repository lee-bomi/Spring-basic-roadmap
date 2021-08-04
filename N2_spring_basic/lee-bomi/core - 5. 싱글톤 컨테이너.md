# 스프링 핵심원리 - 기본편

### 4. 싱글톤 컨테이너

------

1. 웹 어플리케이션과 싱글톤

   - 웹 어플리케이션 : 고객의 요청이 많다 => 요청마다 new로 새 인스턴스를 만들면 부하가 걸린다

     - 순수 Java코드로 이를 확인

       ![image-20210802055924969](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210802055924969.png)

       ![image-20210802060053861](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210802060053861.png)

       ​				=> 서로 다른 객체가 생성됨(효율적이지 않음)

       

     - Spring을 이용한 코드로 확인**(1개의 객체가 생성되어 공유하도록! == "싱글톤")**

     - 싱글톤 패턴

       - 클래스의 인스턴스가 1개만 생성되는것을 보장하는 디자인 패턴

         ![image-20210802061436277](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210802061436277.png)

         ![image-20210802061509039](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210802061509039.png)

         

       

       

2. 싱글톤 패턴

   - 싱글톤의 단점

     - 아래와 같이 싱글톤을 사용하기위한 인스턴스를 만들어야한다
     - DIP위반 : 구현체에 의존(구현체에의존하기때문에 수정이 용이하지 않음 => OCP위반 가능성 높음)

     👉 스프링으로 이 단점을 극복가능

   ![image-20210802192118399](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210802192118399.png)

   ![image-20210802192156220](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210802192156220.png)

3. 싱글톤 컨테이너

   - 스프링컨테이너를 사용하면 객체인스턴스가 싱글톤으로 관리된다

   ![image-20210802193023515](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210802193023515.png)

   



------


