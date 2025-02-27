# 스프링 핵심원리 - 기본편

### 9. 빈 스코프



#### 학습목표

- 미리 외부와 연결을 해야한다면, 객체의 초기와화 종료작업이 필요함

  이때 필요한 **스프링으로 객체 초기화작업 및 종료작업의 진행과정**을 알아본다


------

1. #### 빈 스코프란?

   - 싱글톤 : 스프링컨테이너의 시작과 종료까지 유지되는 가장넓은 범위의 스코프

   - 프로토타입 : 스프링컨테이너가 빈의 생성과 주입까지만 관여하는 짧은범위의 스코프

     

2. ####  프로토타입 스코프

   - 싱글톤 빈 :  스프링컨테이너 생성시점에 init(초기화)메서드 실행(1번만 실행)
   - 같은 인스턴스 가져옴
   - 스프링컨테이너가 종료될때 스프링빈이 destroy됨

   ![8](https://user-images.githubusercontent.com/68681443/128470431-0b511cd1-9c54-4a1e-946f-50e3d969e78b.PNG)




 - 프로토타입 빈 : 스프링컨테이너가 빈을 조회할때 init메서드 실행(2번실행, 싱글톤이 아니므로)

 - 두번 조회했을 때 다른 인스턴스 생성

 - 프로토타입 빈은 의존관계 주입까지만 관여하므로, 스프링컨테이너 종료시@PreDestroy메서드가 실행되지않는다

   ![9](https://user-images.githubusercontent.com/68681443/128471411-d9ea64c7-1700-4a0e-b989-e4e8c6d0a83a.PNG)

   

   📌프로토타입 빈 특징

   - 스프링컨테이너에 요청할때마다 새로 생성된다

   - 스프링컨테인너는 프로토타입 빈의 생성과 의존관계 주입, 초기화까지 관여한다

   - 종료메서드가 호출되지 않는다

     

3. #### 프로토타입 스코프 - 싱글톤 빈과 함께 사용시 문제점

   - 스프링은 일반적으로 싱글톤 빈을 사용하기때문에, 싱글톤빈이 프로토타입 빈을 사용하게 됨

   - 싱글톤빈은 생성시점에만 의존관계를 주입받기때문에, 프로토타입빈이 생성되기는 해도 싱글톤과 유지되지않음

   - 사용할때마다 프로토타입 빈을 새로생성해서 사용하는 방법이 필요함

     

4. #### 프로토타입 스코프 - 싱글톤 빈과 함께 사용시 Provider 문제 해결

