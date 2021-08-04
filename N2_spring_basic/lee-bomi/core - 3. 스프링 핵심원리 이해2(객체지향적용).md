# 스프링 핵심원리 - 기본편

### 3. 스프링 핵심 원리 이해2 - 객체지향 원리 적용

#### 학습목표

- 구현체가 여러개일 경우 어떻게 갈아끼우는가?

- 갈아끼우는 과정이 OCP와 DIP를 잘 지키고 있는가?

- 순수Java vs Spring사용의 차이

  

------

1. 새로운 할인정책 개발

   - 정률 할인정책을 추가

     ![image-20210730212048297](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210730212048297.png)

   - 새로운 할인정책을 바꿔 껴준다

     ![image-20210730214915762](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210730214915762.png)

2. 새로운 할인정책의 문제점

   ⚠ DIP위반 - 인터페이스 뿐만아니라 구현체에도 의존

   ![image-20210730214732982](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210730214732982.png)

   

   ⚠ OCP위반 - 할인정책을 바꿔주려면 OrderServiceImpl의 소스코드도 바꿔줘야한다(변경하지 않고 확장 = OCP)

   ![image-20210730214915762](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210730214915762.png)

   👉 OCP를 위반하지 않고 구현체를 사용하려면,  new로 구현체를 직접 넣어줄게아니라 

   ​      **누군가가 OrderServiceImpl에 구현객체를 새로 생성**하고 주입해줘야한다.

   

3. 관심사의 분리

   - 위의 코드에서는 OrderServiceImpl이 어떤 인터페이스 새 객체를 만들면서 "다양한 책임"을 갖게된다

   - AppConfig(공연기획자)는 배역(인터페이스)를 지정하고 배역의 캐스팅(구현체를 넣어주는것)을 수행해야함

   - 위의 코드에서는 배역(OrderServiceImpl)가 직접 캐스팅(new)을 하고있다

   - AppConfig가 구현객체를 생성 및 연결하고, ""인터페이스""는 그 역할에만 충실하도록 관심사를 분리해야한다

     

   **✔ServiceImpl은 생성자로 주입받을수 있게 수정**

   ![image-20210731115547346](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210731115547346.png)

   **✔AppConfig(공연기획자) 에서 원하는  구현체 생성 및 연결**

   ![image-20210731115617092](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210731115617092.png)

👉 이로인해 AppConfig은 구현체 연결을 담당하고, 연결된 구현체들은 해당 코드를 실행하는 책임만 갖는다

![image-20210731120546525](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210731120546525.png)

👉DIP완성 : MemberServiceImpl은 MemberRepository인 추상클래스에만 의존하고, 구체클래스는 몰라도된다.

(MemberServiceImpl 구동에 필요한 부가적인 역할은 AppConfig가 중재해준다)

4. AppConfig 리팩터링

   기존 AppConfig 코드는 **<u>역할에 따른 구현이 잘 보이지 않음</u>** 

   👉  Ctrt+alt+M(Extract Method)를 사용하여 역할과 구분이 한눈에 보이게끔 리팩토링한다(중복제거 및 메서드화)

![image-20210731194148801](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210731194148801.png)

5. SOLID적용

   - SRP(단일책임원칙) : 구현객체 생성 및 연결 = AppConfig / 해당코드실행 = 클라이언트 객체

   - DIP(추상에의존) : 생성자주입으로 DI를 받아, 추상에 의존

   - OCP(클라이언트코드수정X) : AppConfig코드만을 수정하여 구현체를 바꿀수있게 수정가능

     

6. IoC컨테이너, DI

   - IoC컨테이너(=DI컨테이너) : AppConfig처럼 제어권을 가져와서 의존관계를 조립해주는것

     (클라이언트객체는 구현객체가 어떤게 연결되는지 등의 제어권을 갖고있지 않음)

     

7. 스프링으로 전환하기

   - 순수 Java코드 사용![image-20210731203255118](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210731203255118.png)

     - appConfig를 사용하여 직접 인스턴스를 호출

       

   - 스프링 사용![image-20210731203326006](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210731203326006.png)

     - ApplicationContext = 스프링컨테이너

     - AppConfig에  return하는 객체가 스프링빈에 등록된다.

     - 등록된 스프링빈을 가져오는 것 => applicationContext.getBean("AppConfig의 메서드명", "타입")

       

    ❗ 스프링에 환경정보를 주고, 원하는 bean을 가져올수있도록 스프링이 대신해준다

   ❓ 하지만 이렇게만보면 스프링을 사용하는것에 대한 이점이 많이 없는듯하다. (이 의문은 다음섹션에서)



------

#### 학습결과

- 구현체가 여러개일 경우 어떻게 갈아끼우는가?  

  👉 중재자(AppConfig)가 구현체 선택 및 연결을 담당, 각 구현체는 실행에 집중

- 갈아끼우는 과정이 OCP와 DIP를 잘 지키고 있는가? 

  👉 각 구현체에서 new로 새로운 인스턴스 생성하지않고 중재자를 거쳐 DI를 해주면 

  ​       OCP(클라이언트_ServiceImpl_를 수정하지 않고 변경하는것) ,DIP(인터페이스만 의존하는것)

- 순수Java vs Spring사용의 차이

  👉 Spring에 환경설정을 등록함으로써, 등록된 빈을 자유로이 가져와 쓸 수 있게되었다(아직은 여기까지만!)

