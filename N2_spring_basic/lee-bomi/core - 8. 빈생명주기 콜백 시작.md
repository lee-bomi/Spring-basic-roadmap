# 스프링 핵심원리 - 기본편

### 8.  빈 생명주기 콜백 

#### 학습목표

- 미리 외부와 연결을 해야한다면, 객체의 초기와화 종료작업이 필요함

  이때 필요한 **스프링으로 객체 초기화작업 및 종료작업의 진행과정**을 알아본다


------

1. #### 빈생명주기 콜백 시작

   - 스프링빈의 라이프 사이클

     - 객체생성(값이 없는상태) => 의존관계주입 

       (*생성자 주입의경우는 제외-**객체를 만들때, 파라미터로 스프링빈이 들어와야함**)

       위의 과정이 끝나야 데이터를 사용할 수 있다

       (의존관계주입까지 끝나지않았는데 객체 초기화및 종료작업이 되어선 안된다)

     <br>  

   - 의존관계주입이 끝난시점 어떻게 알지?

     👉**스프링은 의존관계주입 완료 시 스프링 빈에게 콜백메서드를 통해 초기화시점을 알려준다.**

     👉**스프링은 스프링컨테이너가 종료되기 직전에도 소멸 콜백을 준다**

     <br>

     **🔰스프링 빈의 이벤트 라이프사이클🔰**

     > **스프링컨테이너 생성 > 스프링 빈 생성 > 의존관계 주입 > 초기화콜백 > 사용 > 소멸전 콜백 > 스프링종료**

     - 초기화 콜백 : 빈 생성 후, 빈의 의존관계 주입이 완료된 후 호출
     - 소멸전 콜백 : 빈이 소멸되기 직전에 호출

     📌 스프링은 다양한 생명주기 콜백을 지원한다.

     <br>

   - 애초에 생성자에 필요값을 넘겨서 초기화하면 되지않을까?

     ❌ DIP에 위반됨 

      - 생성자 : 필수정보(파라미터)를 받아 메모리에 할당하여 객체 생성하는 책임
      - 초기화 : 생성된 값들을 활용하여 외부커넥션에 연결하는 무거운 책임(생성된 객체가 움직이는 행위)

     👉따라서 생성자 내에서 무거운 초기화 작업을 하기보다는, 각자의 부분을 나누는게 유지보수관점에서 좋다.

     <br> 

   - 스프링이 빈 생명주기 콜백을 지원하는 방법

     - 인터페이스(InitializingBean, DisposableBean)

     - 설정정보에 초기화 메서드, 종료 메서드 지정

     - @PostConstruct, @PreDestroy 어노테이션 지원 **(✔이 방법을 사용할 것!)**

     <br>  


2. #### 인터페이스(InitializingBean, DisposableBean)

   - 해당 스프링 인터페이스를 물려받으면, 스프링 의존관계 생성 후 afterPropertiesSet메서드가 실행

   ![캡처2](https://user-images.githubusercontent.com/68681443/128461952-cbb80b6b-d512-47c7-ba97-49e2dc0891bc.PNG)

   ![캡처3](https://user-images.githubusercontent.com/68681443/128461955-fdf2f60b-5afe-4914-9f90-954ec54af5af.PNG)

   - 아래와 같이 스프링컨테이너가 소멸되면 destroy 메서드가 실행된다

   ![캡처](https://user-images.githubusercontent.com/68681443/128461728-dd65de12-d0ca-48bb-ad6f-9fc86c68cd63.PNG)

   📌초기화 소멸메서드의 단점

    - 스프링전용 인터페이스라서 해당코드가 스프링 인터페이스에 의존한다
    - 메서드 이름 변경 불가

   <br>

3. #### 빈 등록 초기화, 소멸메서드

   - 원하는 메서드를 만들어준다

     ![4](https://user-images.githubusercontent.com/68681443/128463804-893a5f08-f310-4a31-b4c3-99ff763e0343.PNG)

   - 해당 스프링 빈에 초기화, 소멸 등록해준다

     ![6](https://user-images.githubusercontent.com/68681443/128463807-52f1cf13-5c67-4f63-9440-d84fe895ddc3.png)

     

     **📌 메서드를 이용한 초기화, 소멸의 장점**

     - 메서드이름 자유롭게 지정가능
     - 스프링 빈이 스프링코드에 의존하지않음
     - 외부라이브러리에 연결 시 적용 가능

   <br>

4. #### 어노테이션 @PostConstruct, @PreDestroy  

   - 인터페이스를 상속받을 필요도, 코드를 적어주지 않아도되서 **"제일 권장되는 방법"**

   - javax를 import해오기 때문에 다른 프레임워크를 사용해도 사용가능

   - 단점 : 외부라이브러리에 사용 불가

     ![7](https://user-images.githubusercontent.com/68681443/128464828-62df2868-2830-40e5-aec1-9652582e6c1c.png)

<br>

------

#### 학습결과

- **스프링으로 객체 초기화작업 및 종료작업의 진행과정**

  👉 어노테이션 @PostConstruct, @PreDestroy  을 기본으로 사용하되

  👉 외부라이브러리를 사용해야 한다면 빈 등록 초기화, 소멸메서드를 사용할것

