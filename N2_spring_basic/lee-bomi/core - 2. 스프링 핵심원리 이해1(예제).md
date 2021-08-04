# 스프링 핵심원리 - 기본편

### 2. 스프링 핵심 원리 이해1 - 예제만들기

#### 학습목표

- 순수 Java만을 사용하여 개발(스프링 사용X)
- 요구사항이 변경되면 유연하게 수정이 가능한가?
- 객체지향의 핵심인 다형성, OCP, DIP등이 잘 지켜지고 있는가?

------

1. 프로젝트 생성

   - start.spring.io 에서 dependency없이 package생성 

     => 순수 java를 사용하여 약간은 불편하지만, 동작하는 원리를 익히기 위함

     

2. 비즈니스 요구사항과 설계

   📌 역할과 구현을 구분!(구현체가 정해지지 않은경우가 있기때문)

   📌 목적별 다이어그램 구분

   	- 회원클래스 다이어그램 : 실제 인터페이스, 클래스 설계를 위한 그림(정적)
   	- 회원객체 다이어그램 : test, 서버구동 등  흐름을 확인하기 위한 그림(동적)
   
   ![image-20210728220146943](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210728220146943.png)
   
   ​		
   
3. 회원도메인설계

   - 회원을 가입, 조회할 수 있다 👉회원클래스와 해당기능의 인터페이스설계

   - 회원은 일반과 vip등급이 있다 👉 Enum class로 basic, vip를 명시

   - 회원데이터는 자체 DB구축할 수 있고, 외부와 연동할 수 있다 

     👉 다형성을 활용하여, 자체DB사용하는 구현체를 만든다(MemoryMemberRepositoryl) 

     

   📌**순수 Java코드를 이용한 도메인 설계의 문제점 - DIP위반**(역할에의존하되⭕, 구현에의존하면안됨❌)

   	- MemberServiceImpl은 MemberRepository뿐 아니라 그의 구현체(MemoryMemberRepository)까지 의존하고있다
   	- 이는 추후에 유연하게 수정할 수 없음을 의미(각 구현체를 고쳐줘야하는 번거로움이 있다)

   ![image-20210729075826048](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210729075826048.png)
   
   
   
4. 주문과 할인 도메인 설계

   - 회원등급에 따라 할인정책을 적용할수있다 👉 FixDiscountPolicy에서 등급에 따른 할인금액 return

   - 할인방식은 변경될 수 있다 👉 추후에 구현체를 교체할 수 있도록 설계

     ![image-20210730075823114](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210730075823114.png)

     

   

------

#### 학습결과

- 요구사항이 변경되면 유연하게 수정이 가능한가? 👉 NO, 클라이언트코드를 수정해 줘야한다(OrderServiceImpl)
- 객체지향의 핵심인 다형성, OCP, DIP등이 잘 지켜지고 있는가? 👉 수정에 닫혀있으며, 주입을 받고있지 않다
