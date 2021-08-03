# 1. 웹 애플리케이션과 싱글톤

- 스프링은 태생이 기업용 온라인 서비스 기술 지원위해 탄생함
- `대부분의 스프링 애플리케이션은 웹 애플리케이션`이다. 
- 웹 애플리케이션은 보통 여러 고객이 동시에 요청을 한다.

![image](https://user-images.githubusercontent.com/38436013/127967886-68618115-0361-41b0-8c41-4a09d8e8f1f8.png)

> 요청의 개수 만큼 객체가 생성됨 -> 문제가 됨

#### 스프링 없는 순수한 DI 컨테이너 테스트

~~~java
package hello.core.singleton;

import hello.core.AppConfig;
import hello.core.member.MemberService;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.Assertions.assertThat;

public class SingletonTest {
    @Test
    @DisplayName("스프링 없는 순수한 DI 컨테이너")
    void pureContainer() {
        AppConfig appConfig = new AppConfig();
        // 1. 조회 : 호출할 때마다 객체를 생성
        MemberService memberService1 = appConfig.memberService();
        // 2. 조회 : 호출할 때마다 객체를 생성
        MemberService memberService2 = appConfig.memberService();

        // 참조값이 다른 것을 확인
        System.out.println("memberService1 = "+ memberService1);
        System.out.println("memberService2 = "+ memberService2);

        //memberService1 != memberService2
        assertThat(memberService1).isNotSameAs(memberService2);
    }
}
// 여기서는 두개 객체를 생성한 것으로 보여지지만, 객체 인터페이스, 해당 구체 클래스 모두 생성되므로 총 4개가 생성된다.
~~~

- 우리가 만들었던 스프링 없는 순수한 DI 컨테이너인 AppConfig는 요청을 할 때 마다 객체를 새로 생성한다.

- 고객 트래픽이 초당 100이 나오면 초당 100개 객체가 생성되고 소멸된다 -> 메모리 낭비 심하다

- 해결방안은 **해당 객체가 1개만 생성되고, 공유하도록 설계**한다 -> `싱글톤 패턴`

# 2. 싱글톤 패턴

- 클래스의 **인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴**이다.
- 그래서 객체 인스턴스를 2개 이상 생성하지 못하도록 막아야 한다.
  - private 생성자를 사용해서 외부에서 임의로 new 키워드를 사용하지 못하도록 막아야 한다.

싱글톤 패턴을 적용한 예제 코드를 보자. main이 아닌 test 위치에 생성하자.

~~~java
package hello.core.singleton;

public class SingletonService {

    private static final SingletonService  instance = new SingletonService();

    // 1. static 영역에 객체를 딱 1개만 생성해둔다.
    public static SingletonService getInstance(){
        return instance;
    }
    // 2. public으로 열어서 객체 인스턴스가 필요하면 이 static 메서드를 통해서만 조회하도록 허용한다.
    private SingletonService() {

    }
    //3. 생성자를 private으로 선언해서 외부에서 new 키워드를 사용한 객체 생성을 못하게 막는다
    public void logic() {
        System.out.println("싱글톤 객체 로직 호출");
    }

}

~~~

> 1. static 영역에 객체 instance를 미리 하나 생성해서 올려둔다.
> 2. 이 객체 인스턴스가 필요하면 오직 getInstance() 메서드를 통해서만 조회할 수 있다. 이 메서드를 호출하면 항상 같은 인스턴스를 반환한다.
> 3. 딱 1개의 객체 인스턴스만 존재해야 하므로, 생성자를 private으로 막아서 혹시라도 외부에서 new 키워드로 객체 인스턴스가 생성되는 것을 막는다.

#### 싱글톤 패턴을 사용하는 테스트 코드

~~~java
package hello.core.singleton;

import hello.core.AppConfig;
import hello.core.member.MemberService;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.Assertions.assertThat;

public class SingletonTest {
    @Test
    @DisplayName("싱글톤 패턴을 적용한 객체 사용")
    public void singletonServiceTest() {
        //private으로 생성자를 막아두었다. 컴파일 오류가 발생한다.
        //new SingletonService();

        //1. 조회: 호출할 때 마다 같은 객체를 반환
        SingletonService singletonService1 = SingletonService.getInstance();

        //2. 조회: 호출할 때 마다 같은 객체를 반환
        SingletonService singletonService2 = SingletonService.getInstance();

        //참조값이 같은 것을 확인
        System.out.println("singletonService1 = " + singletonService1);
        System.out.println("singletonService2 = " + singletonService2);

        // singletonService1 == singletonService2
        assertThat(singletonService1).isSameAs(singletonService2);
        singletonService1.logic();
    }
}
~~~

> singletonService1 = hello.core.singleton.SingletonService@327514f
> singletonService2 = hello.core.singleton.SingletonService@327514f
>
> 같은 객체를 참조하고 있음 

- private으로 new 키워드를 막아두었다.
- 호출할 때 마다 같은 객체 인스턴스를 반환하는 것을 확인할 수 있다.

참고: 싱글톤 패턴을 구현하는 방법은 여러가지가 있다. 여기서는 `객체를 미리 생성해두는 가장 단순하고 안전한 방법`을 선택했다.

싱글톤 패턴을 적용하면 고객의 요청이 올 때 마다 객체를 생성하는 것이 아니라, `이미 만들어진 객체를 공유해서 효율적으로 사용`할 수 있다. 하지만 싱글톤 패턴은 다음과 같은 수 많은 문제점들을 가지고 있다.

### 싱글톤 패턴 문제점

- 싱글톤 패턴을 구현하는 코드 자체가 많이 들어간다.
- 의존관계상 클라이언트가 구체 클래스에 의존한다. DIP를 위반한다.
- 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다.
- 테스트하기 어렵다.
- 내부 속성을 변경하거나 초기화 하기 어렵다.
- private 생성자로 자식 클래스를 만들기 어렵다.
- 결론적으로 유연성이 떨어진다.
- 안티패턴으로 불리기도 한다.

# 3. 싱글톤 컨테이너

### 싱글톤 컨테이너

 스프링 컨테이너는 **싱글톤 패턴의 문제점을 해결**하면서, **객체 인스턴스를 싱글톤(1개만 생성)으로 관리**한다.
지금까지 우리가 학습한 **스프링 빈이 바로 싱글톤으로 관리되는 빈**이다.

- 스프링 컨테이너는`싱글턴 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리`한다.
- 스프링 컨테이너는 싱글톤 컨테이너 역할을 한다. 이렇게 싱글톤 객체를 생성하고 관리하는 기능을 **싱글톤 레지스트리**라 한다.
- 스프링 컨테이너의 이런 기능 덕분에 싱글톤 패턴의 모든 단점을 해결 & 객체를 싱글톤 관리
  - 싱글톤 패턴을 위한 지저분한 코드가 들어가지 않아도 된다.
  - DIP, OCP, 테스트, private 생성자로 부터 자유롭게 싱글톤을 사용할 수 있다.

#### 스프링 컨테이너를 사용하는 테스트 코드

~~~java
package hello.core.singleton;

import hello.core.AppConfig;
import hello.core.member.MemberService;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class SingletonTest {

    @Test
    @DisplayName("스프링 컨테이너와 싱글톤")
    void springContainer() {

        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService1 = ac.getBean("memberService", MemberService.class);
        MemberService memberService2 = ac.getBean("memberService", MemberService.class);

        // 참조값이 다른 것을 확인
        System.out.println("memberService1 = " + memberService1);
        System.out.println("memberService2 = " + memberService2);

        //memberService1 != memberService2
        Assertions.assertThat(memberService1).isSameAs(memberService2);
    }

}
~~~

> singletonService1 = hello.core.member.MemberServiceImpl@5911e990
> singletonService2 = hello.core.member.MemberServiceImpl@5911e990

### 싱글톤 컨테이너 적용 후

![image](https://user-images.githubusercontent.com/38436013/127980670-d428deed-696f-4427-a091-cc6d7e9b7b95.png)

- 스프링 컨테이너 덕분에 이미 만들어진 객체를 공유해서 효율적으로 재사용할 수 있다.

> 참고: 스프링의 기본 빈 등록 방식은 싱글톤이지만, 싱글톤 방식만 지원하는 것은 아니다. 요청할 때 마다 새로운 객체를 생성해서 반환하는 기능도 제공한다. 자세한 내용은 뒤에 빈 스코프에서 설명하겠다.

# 4. 싱글톤 방식의 주의점

- 싱글톤 패턴이든, 스프링 같은 싱글톤 컨테이너를 사용하든, `객체 인스턴스를 하나만 생성해서 공유하는 싱글톤 방식`은` 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유`하기 때문에 싱글톤 객체는 **상태를 유지(stateful)하게 설계하면 안된다.**
- **무상태(stateless)로 설계해야 한다!**
  - 특정 클라이언트에 의존적인 필드가 있으면 안된다.
  - 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다!
  - 가급적 읽기만 가능해야 한다.
  - 필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다.

#### 상태를 유지할 경우 발생하는 문제점 예시

~~~java
package hello.core.singleton;

public class StatefulService {
    private int price; // 상태를 유지하는 필드
    public void order(String name, int price){
        System.out.println("name = " + name + "price = " + price);
        this.price = price; // 여기가 문제

    }
    public int getPrice() {
        return price;
    }
}

~~~

#### 상태를 유지할 경우 발생하는 문제점 예시

~~~java
package hello.core.singleton;

import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.*;

class StatefulServiceTest {
    //  컨테이너 생성 ,  getBean() : 조회
    @Test
    void statefulServiceSingleton() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
    StatefulService statefulService1 = ac.getBean(StatefulService.class);
    StatefulService statefulService2 = ac.getBean(StatefulService.class);

    //ThreadA: A 사용자 10000원 주문
        statefulService1.order("userA", 10000);
    //ThreadB: B 사용자 20000원 주문
        statefulService2.order("userB", 20000);

    //ThreadA: 사용자A 주문 금액 조회
    int price = statefulService1.getPrice();
    //ThreadA: 사용자A는 10000원을 기대했지만, 20000원을 출력 -> 상태 유지 때문
        System.out.println("price = " + price);

    assertThat(statefulService1.getPrice()).isEqualTo(20000);
}

    static class TestConfig {

        @Bean
        public StatefulService statefulService() {
            return new StatefulService();
        }
    }
}
~~~

- 최대한 단순히 설명하기 위해, 실제 쓰레드는 사용하지 않았다.
- `StatefulService` 의 price 필드는 공유되는 필드인데, 특정 클라이언트가 값을 변경한다.
- 사용자A의 주문금액은 10000원이 되어야 하는데, 20000원이라는 결과가 나왔다.
- 실무에서 이런 경우를 종종 보는데, 이로인해 정말 해결하기 어려운 큰 문제들이 터진다.
- 진짜 **공유필드는 조심해야 한다!** 스프링 빈은 항상 `무상태(stateless)로 설계`하자.

# 5. @Configuration과 싱글톤

#### AppConfig 

~~~java
package hello.core;

import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.discount.RateDiscountPolicy;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import hello.core.member.MemoryMemberRepository;
import hello.core.order.OrderService;
import hello.core.order.OrderServiceImpl;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemoryMemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }
   ...
}
~~~

> // @Bean memberService -> new MemoryMemberRepository  메서드 호출
>
> // @Bean orderService -> new MemoryMemberRepository  메서드 호출 
>
> // 이런식이라면 싱글톤 깨지는 것이 아닌가? 서로 다른 객체가 아닌가?

- memberService 빈을 만드는 코드를 보면 memberRepository() 를 호출한다.
  - 이 메서드를 호출하면 new MemoryMemberRepository() 를 호출한다.
- orderService 빈을 만드는 코드도 동일하게 memberRepository() 를 호출한다.
  - 이 메서드를 호출하면 new MemoryMemberRepository() 를 호출한다.

결과적으로 각각 다른 2개의 MemoryMemberRepository 가 생성되면서 싱글톤이 깨지는 것 처럼 보인다. 스프링 컨테이너는 이 문제를 어떻게 해결할까?

#### 검증 용도의 코드 추가

~~~java
package hello.core.member;

public class MemberServiceImpl implements MemberService {

    private final MemberRepository memberRepository;

    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Override
    public void join(Member member) {
        memberRepository.save(member);
    }

    @Override
    public Member findMember(Long memberId) {
        return memberRepository.findById(memberId);
    }
    
    // 테스트 용도 추가
    public MemberRepository getMemberRepository() {
        return memberRepository;
    }
}
~~~

~~~java
public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository ;
    private final DiscountPolicy discountPolicy ;

    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }

    @Override
    public Order createOrder(Long memberId, String itemName, int itemPrice) {
        // 회원 정보 조회하고, 할인 정책에 회원 넘김
        Member member = memberRepository.findById(memberId);
        int discountPrice = discountPolicy.discount(member, itemPrice);

        return new Order(memberId, itemName, itemPrice, discountPrice);
        
    }
    //테스트 용도 추가
    public MemberRepository getMemberRepository(){
        return memberRepository;
    }
~~~



#### 테스트 코드

~~~java
package hello.core.singleton;

import hello.core.AppConfig;
import hello.core.member.MemberRepository;
import hello.core.member.MemberServiceImpl;
import hello.core.order.OrderServiceImpl;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class ConfigurationSingletonTest {

    @Test
    void configurationTest() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
        OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);
        MemberRepository memberRepository = ac.getBean("memberRepository", MemberRepository.class);

        MemberRepository memberRepository1 = memberService.getMemberRepository();
        MemberRepository memberRepository2 = orderService.getMemberRepository();

        System.out.println("memberRepository1 = " + memberRepository1);
        System.out.println("memberRepository2 = " + memberRepository2);
        System.out.println("memberRepository = " + memberRepository);

        Assertions.assertThat(memberService.getMemberRepository()).isSameAs(memberRepository);
        Assertions.assertThat(orderService.getMemberRepository()).isSameAs(memberRepository);
    }
}

~~~

>memberRepository1 = hello.core.member.MemoryMemberRepository@3700ec9c
>memberRepository2 = hello.core.member.MemoryMemberRepository@3700ec9c
>memberRepository = hello.core.member.MemoryMemberRepository@3700ec9c

- 확인해보면 memberRepository 인스턴스는 모두 같은 인스턴스가 공유되어 사용된다.
- AppConfig의 자바 코드를 보면 분명히 각각 2번 new MemoryMemberRepository 호출해서 다른 인스턴스가 생성되어야 하는데?
- 어떻게 된 일일까? 혹시 두 번 호출이 안되는 것일까? 실험을 통해 알아보자.

#### AppConfig에 호출 로그 남김

~~~java
package hello.core;

import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.discount.RateDiscountPolicy;
import hello.core.member.MemberRepository;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import hello.core.member.MemoryMemberRepository;
import hello.core.order.OrderService;
import hello.core.order.OrderServiceImpl;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Controller;



@Configuration
public class AppConfig {
    @Bean
    public MemberService memberService() {
        System.out.println("call AppConfig.memberService");
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemoryMemberRepository memberRepository() {
        System.out.println("call AppConfig.memberRepository");
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        System.out.println("call AppConfig.orderService");
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy() {
//        return new FixDiscountPolicy();
        return new RateDiscountPolicy();
    }
}

~~~

~~~java
package hello.core.singleton;

import hello.core.AppConfig;
import hello.core.member.MemberRepository;
import hello.core.member.MemberServiceImpl;
import hello.core.order.OrderServiceImpl;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

// 예상 호출 순서
// call AppConfig.memberService
// call AppConfig.memberRepository
// call AppConfig.memberRepository
// call AppConfig.orderService
// call AppConfig.memberRepository

// 실제 호출 순서
// call AppConfig.memberService
// call AppConfig.memberRepository
// call AppConfig.orderService

public class ConfigurationSingletonTest {

    @Test
    void configurationTest() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
        OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);
        MemberRepository memberRepository = ac.getBean("memberRepository", MemberRepository.class);

        MemberRepository memberRepository1 = memberService.getMemberRepository();
        MemberRepository memberRepository2 = orderService.getMemberRepository();

        System.out.println("memberRepository1 = " + memberRepository1);
        System.out.println("memberRepository2 = " + memberRepository2);
        System.out.println("memberRepository = " + memberRepository);

        Assertions.assertThat(memberService.getMemberRepository()).isSameAs(memberRepository);
        Assertions.assertThat(orderService.getMemberRepository()).isSameAs(memberRepository);
    }
}

~~~

> // 예상 호출 순서
> // call AppConfig.memberService
> // call AppConfig.memberRepository
> // call AppConfig.memberRepository
> // call AppConfig.orderService
> // call AppConfig.memberRepository
>
> // 실제 호출 순서
> // call AppConfig.memberService
> // call AppConfig.memberRepository
> // call AppConfig.orderService

- 예상과 다르게 스프링 컨테이너는 모두 1번 호출해준다
- why?

# 6. Configuration과 바이트코드 조작의 마법

스프링 컨테이너는 싱글톤 레지스트리다.스프링은 싱글톤을 보장해주기 위해 클래스의 바이트코드를 조작하는 라이브러리를 사용한다.
모든 비밀은 `@Configuration 을 적용한 AppConfig` 에 있다.

다음 코드를 보자.

```java
    @Test
    void configurationDeep() {
        //AppConfig도 스프링 빈으로 등록된다
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        AppConfig bean = ac.getBean(AppConfig.class);
        //출력: bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$bd479d70

        System.out.println("bean = " + bean.getClass());
    }
```

- 사실 AnnotationConfigApplicationContext 에 파라미터로 넘긴 값은 스프링 빈으로 등록된다. 그래서 AppConfig 도 스프링 빈이 된다.
- AppConfig 스프링 빈을 조회해서 클래스 정보를 출력해보자.

> bean = class hello.core.AppConfig*E**n**h**a**n**c**e**r**B**y**S**p**r**i**n**g**C**G**L**I**B*bd479d70

'*n**h**a**n**c**e**r**B**y**S**p**r**i**n**g**C**G**L**I**B*bd479d70' 이것은  스프링이 CGLIB라는 바이트코드 조작 라이브러리를 사용해서 AppConfig 클래스를 상속받은 임의의 다른 클래스다. 임의의 다른 클래스를 스프링 빈으로 등록한 것이다.

![image](https://user-images.githubusercontent.com/38436013/128005027-99b7fc41-fa1a-4fc7-9049-b2eac1855e26.png)

> 임의의 클래스가 싱글톤이 보장되도록 해준다, 

#### AppConfig@CGLIB 예상 코드

~~~java
@Bean
public MemberRepository memberRepository() {
    if (memoryMemberRepository가 이미 스프링 컨테이너에 등록되어 있으면?){
        return 스프링 컨테이너에서 찾아서 반환;
    } else { //스프링 컨테이너에 없으면
        기존 로직을 호출해서 MemoryMemberRepository를 생성하고 스프링 컨테이너에 등록
        return 반환
    }
}
~~~

- **@Bean이 붙은이 붙은 메서드마다 스프링 빈이 존재하면, 빈 반환하고, 없으면 생성하고 빈으로 등록하여 반환해준다.**

- @Configuration 을 적용하지 않고, @Bean 만 적용하면 어떻게 될까?
  - @Configuration 을 붙이면 바이트코드를 조작하는 CGLIB 기술을 사용해서 싱글톤을 보장하지만, @Bean은 스프링 빈으로 등록만 해주고, 싱글톤 보장하지 않는다. 
  - 결론은 스프링설정정보는 항상 @Configuration 사용해야 한다.

### 

