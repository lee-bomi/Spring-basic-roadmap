# 스프링 핵심원리 - 기본편

### 5. 컴포넌트스캔

#### 학습목표

- @Bean이 아닌 다른방식으로 스프링 빈 등록하기




------

1. 컴포넌트 스캔과 의존관계 자동 주입하기

   - @Bean으로 수십, 수백개의 bean을 등록하기에는 무리가 있다 => 컴포넌트 스캔 + @Autowired

   - @Configuration파일을 만들고, 기존에 빌드했던 파일을 제외(빈으로 등록하고자 하는 클래스에 어노테이션 붙여주기)

     ![image-20210803075029229](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210803075029229.png)

   - @Bean으로 등록한 클래스가 없을때, 컴포넌트스캔(+@Autowired)만으로 아래와 같이 연결 및 주입이 된다

     ![image-20210803074934867](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210803074934867.png)![image-20210803075507980](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210803075507980.png)

     🔴 @Component 어노테이션이 감지됨

     🔵 싱글톤 빈으로 스프링빈에 등록됨

     🟡 @Autowired로 의존관계가 연결됨

     

     📌 @ComponentScan은 @Component가 붙은 모든 클래스를 빈으로 등록하며,

     ​      스프링 빈의 이름은 **"클래스 명을 사용하되, 맨 앞글자를 소문자로 사용"**

     📌 생성자에 @Autowired를 지정 시, 스프링컨테이너가 자동으로 해당스프링 빈을 찾아서 주입한다

     ​       (같은 **타입** 빈을 주입)
     
     

2. 탐색위치와 기본 스캔 대상

   - basePackages : 탐색할 패키지의 위치를 지정한다.(패키지를 여러개도 지정 가능)

   - 만약 지정하지 않으면 👉@ComponentScan이 붙은 클래스의 패키지가 시작위치가됨

     (아래의 경우 @ComponentScan이 붙어있으므로 최상단의 "hello.core"패키지부터 하위로 스캔한다)

   ![image-20210804205635002](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210804205635002.png)

   

   - @ComponentScan을 지정해주지 않는다면 👉 @SpringBootApplication이 붙은 패지지부터 스캔한다

     (***스프링부트를 사용한다면 굳이 신경쓰지않아도 자동생성되는 클래스 및 어노테이션으로 인해 최상단부터 하위로 스캔한다*** )

     ![image-20210804210142144](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210804210142144.png)

     

   - @Component가 아니더라도 아래와 같은 어노테이션이 똑같이 컴포넌트 스캔이 가능하도록 해준다

     ![image-20210804211105945](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210804211105945.png)



3. 필터

   : @Component대신 컴포넌트스캔을 할 수 있는 어노테이션을 만들 수 있다

   - 컴포넌트스캔에 추가 or 제외할 어노테이션 만들기

     => @MyIncludeComponent를 사용하여 컴포넌트스캔에 포함시킬 수 있다

   ![image-20210804212152784](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210804212152784.png)

   ![image-20210804212218769](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210804212218769.png)

   - 컴포넌트 스캔을 할 클래스

     ![image-20210804212440033](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210804212440033.png)

     ![image-20210804212449913](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210804212449913.png)

   - 설정정보와 전체 테스트 코드

     => @MyIncludeComponent타입으로 필터를 적용하여, 해당클래스인 beanA를 스프링빈에 등록가능

     ![image-20210804212736972](C:\Users\BOMI\AppData\Roaming\Typora\typora-user-images\image-20210804212736972.png)

     

   - 일반적으로 @Component스캔을 사용하거나, @ExcludeComponent를 가끔 사용하는 정도

     (관례는 @Component를 쓰도록 장려함)

     

4. 중복등록과 충돌

   : 컴포넌트스캔에서 빈 이름이 같다면?

   👉ConflictingBeanDefinitionException발생





------

#### 학습결과

- @Bean이 아닌 다른방식으로 스프링 빈 등록하기 

  👉@SpringbootApplication하위로 @Configuration or @Controller와 같은 어노테이션들을 클래스에 붙여서 스프링빈으로 등록하고,    👉@Autowired를 사용하여 스프링빈에 등록된 것들중에 의존이 필요한것들을 서로 연결해준다

  👉컴포넌트 스캔을 시작하는 위치는 설정할 수 있지만, 최상단 환경설정 클래스로부터 뻗어나오는 형식이 대중적으로 쓰이며,

  👉컴포넌트스캔에 포함하기위한 어노테이션을 커스터마이징할 순 있지만, 널리 쓰이진 않는다

