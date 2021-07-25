# 내용

---

## 1. 할인 정책 확장

**정률 할인 정책 추가**

![할인정책추가-클래스다이어그램](https://user-images.githubusercontent.com/17094674/126908076-b289e01c-11d6-4a6e-8c94-abea4f34eec1.PNG)


**역할**과 **구현**을 분리해서 자유롭게 할인 정책 구현 객체를 조립할 수 있다. 변경 가능성이 있다면 인터페이스를 사용하자.

## 2. **새로운 할인 정책 적용과 문제점**

할인 정책을 변경한다면 클라이언트인 OrderServiceImpl 코드를 수정해야 한다.

```java
public class OrderServiceImpl implements OrderService {
// private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
 private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
}
```

위 코드는 DCP, DIP를 위반했다.

1) 클라이언트(OrderServiceImpl) 는 인터페이스에 의존해야 하나 구현체(new RateDiscountPolicy)에도 의존하고 있어 DIP를 위반한다.

2) 위 코드는 할인 정책을 확장해서 변경할 경우 클라이언트(OrderServiceImpl) 코드에 영향을 주므로 OCP를 위반한다.

## 3. 해결방안

**1) 관심사의 분리**

- 배우는 **본인의 역할**인 배역을 수행하는 것에만 집중해야 한다.
- 디카프리오는 어떤 여자 주인공이 선택되더라도 똑같이 공연을 할 수 있어야 한다.
- 공연을 구성하고, 담당 배우를 섭외하고, 역할에 맞는 배우를 지정하는 **책임을 담당**하는 별도의 공연 기획자가 나올시점이다.
- 공연 기획자를 만들고, 배우와 공연 기획자의 **책임**을 확실히 **분리**하자

**2) AppConfig 등장**

- 애플리케이션의 전체 동작 방식을 구성(config)하기 위해, 구현 객체를 생성하고, 연결하는 책임을 가지는 별도의 설정 클래스를 만들자

```java
package spring.demo;

import spring.demo.discount.DiscountPolicy;
import spring.demo.discount.FixDiscountPolicy;
import spring.demo.member.MemberRepository;
import spring.demo.member.MemberService;
import spring.demo.member.MemberServiceImpl;
import spring.demo.member.MemoryMemberRepository;
import spring.demo.order.OrderService;
import spring.demo.order.OrderServiceImpl;

public class AppConfig {

    public MemberService memberService() {
        return new MemberServiceImpl(getMemberRepository());
    }

    private MemberRepository getMemberRepository() {
        return new MemoryMemberRepository();
    }

    public OrderService orderService() {
        return new OrderServiceImpl(getMemberRepository(), getDiscountPolicy());
    }

    private DiscountPolicy getDiscountPolicy() {
        return new FixDiscountPolicy();
    }
}
```

2-1) AppConfig는 애플리케이션의 실제 동작에 필요한 구현 객체를 생성한다.

- MemberServiceImpl
- MemoryMemberRepository
- OrderServiceImpl
- FixDiscountPolicy

2-2) AppConfig는 생성한 객체 인스턴스의 참조(레퍼런스)를 생성자를 통해서 주입(연결)해준다.

클라이언트(memberServiceImpl) 입장에서 보면 의존관계를 마치 외부에서 주입해주는 것 같다고 해서 DI(Dependency Injection) 우리말로 **의존관계 주입** 또는 **의존성 주입**이라 한다

- MemberServiceImpl **→** MemoryMemberRepository
- OrderServiceImpl **→** MemoryMemberRepository , FixDiscountPolic

```java
package spring.demo.member;

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
}
```

- 설계 변경으로 클라이언트(MemberServiceImpl)는 구현체(MemoryMemberRepository)를 의존하지 않는다. 단지 인터페이스(MemberRepository)만 의존한다.
- 클라이언트(MemberServiceImpl) 입장에서 어떤 구현 객체가 들어올지(주입될지)는 알 수 없으며, 어떤 구현 객체가 들어올지(주입될지)는 오직 외부(AppConfig)에서 결정된다.
- 클라이언트(MemberServiceImpl)는 **의존관계에 대한 고민은 외부**에 맡기고 **실행에만 집중**하면 된다.

## 4. IoC, DI 그리고 컨테이너

**1) 제어의 역전 IoC(Inversion of Control)**

- 기존 프로그램은 클라이언트 구현 객체가 스스로 필요한 서버 구현 객체를 생성하고, 연결하고, 실행했다. 한마디로 구현 객체가 프로그램의 제어 흐름을 스스로 조종했다.
- AppConfig가 등장한 이후에 구현 객체는 자신의 로직을 실행하는 역할만 담당한다. 프로그램의 제어 흐름은 AppConfig가 가져간다. 예를 들어서 OrderServiceImpl 은 필요한 인터페이스들을 호출하지만 어떤 구현 객체들이 실행될지 모른다.
- 프로그램에 대한 제어 흐름에 대한 권한은 모두 AppConfig가 가지고 있다. 클라이언트(OrderServiceImpl)는 어떤 구현 객체가 주입될지는 알 수 없다. 이렇듯 프로그램의 제어 흐름을 외부(AppConfig)에서 관리하는 것을 제어의 역전(IoC)이라 한다

**2) 프레임워크 vs 라이브러리**

프로그램 제어 흐름이 어디에 있느냐에 따라서 프레임워크, 라이브러리가 구분됨.

- 내가 작성한 코드를 **프레임워크가 제어**하고, 대신 실행하면 그것은 **프레임워크**다. (JUnit)
- 내가 작성한 코드가 **직접 제어의 흐름을 담당**한다면 그것은 **라이브러리**다

**3) DI(Dependency Injection)**

- 애플리케이션 실행 시점(런타임)에 **외부에서 실제 구현 객체를 생성**하고 **클라이언트에 전달**해서 **클라이언트와 서버의 실제 의존관계가 연결 되는 것을 의존관계 주입**이라 한다.
- 의존관계 주입을 사용하면 클라이언트 코드를 변경하지 않고, 클라이언트가 호출하는 대상의 타입 인스턴스를 변경할 수 있다
- 의존관계 주입을 사용하면 정적인 클래스 의존관계를 변경하지 않고, 동적인 객체 인스턴스 의존관계를 쉽게 변경할 수 있다

**4) IoC 컨테이너, DI 컨테이너**

- AppConfig 처럼 객체를 생성하고 관리하면서 의존관계를 연결해 주는 것을 IoC 컨테이너 또는 DI 컨테이너라 한다.
- 의존관계 주입에 초점을 맞추어 최근에는 주로 DI 컨테이너라 한다

## 5.  스프링으로 전환하기

순수 자바로 작성된 코드를 스프링으로 전환하자

**1) AppConfig**

```java
package spring.demo;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import spring.demo.discount.DiscountPolicy;
import spring.demo.discount.RateDiscountPolicy;
import spring.demo.member.MemberRepository;
import spring.demo.member.MemberService;
import spring.demo.member.MemberServiceImpl;
import spring.demo.member.MemoryMemberRepository;
import spring.demo.order.OrderService;
import spring.demo.order.OrderServiceImpl;

@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }
}
```

**2) OrderApp**

```java
package spring.demo;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import spring.demo.member.Grade;
import spring.demo.member.Member;
import spring.demo.member.MemberService;
import spring.demo.member.MemberServiceImpl;
import spring.demo.order.Order;
import spring.demo.order.OrderService;
import spring.demo.order.OrderServiceImpl;

public class OrderApp {

    public static void main(String[] args) {
//      final AppConfig appConfig = new AppConfig();
//      final MemberService memberService = appConfig.memberService();
//      final OrderService orderService = appConfig.orderService();

        final ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        final MemberService memberService = ac.getBean("memberService", MemberService.class);
        final OrderService orderService = ac.getBean("orderService", OrderService.class);

        final Long memberId = 1L;
        final Member member = new Member(memberId, "namDeng", Grade.VIP);
        memberService.join(member);

        final Order order = orderService.createOrder(memberId, "jpa", 10000);

        System.out.println(order);
    }
}
```

**스프링 컨테이너**

- ApplicationContext 를 스프링 컨테이너라 한다
- **스프링 컨테이너**는 **@Configuration이 붙은 AppConfig 를 설정(구성) 정보로 사용**한다. 여기서 @Bean이라 적힌 메서드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록한다. 이렇게 스프링 컨테이너에 등록된 객체를 **스프링 빈**이라 한다
- 기존에는 개발자가 직접 자바코드로 모든 것을 했다면 이제부터는 스프링 컨테이너에 객체를 스프링 빈으로 등록하고, 스프링 컨테이너에서 스프링 빈을 찾아서 사용하도록 변경되었다.

# Q&A

---

### **Q: MemberServiceImple은 하나의 인터페이스를 따르게 되어 Impl을 넣으신다 하셨는데, 고정할인정책이나 비율할인정책 클래스는 왜 없을까요 ?! 어떤 관례라고 들었는데 궁금하네요.**

A: 인터페이스가 하나이고, 구현 클래스도 딱 하나인 경우에는 관례상 Impl을 많이 붙입니다.

할인 정책 같은 경우에는 구현 클래스가 하나가 아니니까요^^

### Q: 정액할인정책이 정률할인정책으로 바뀌면서 AppConfig에서는 와 같이 FixDiscountPolicy대신 RateDiscountPolicy로 바꿔주고 있습니다.  그러면 OCP를 위배하는것은 똑같은것 아닌가요?

```
private DiscountPolicy DiscountPolicy() {
    return new RateDiscountPolicy();
}
```

A: 사용 영역과 구성 영역을 설명드렸는데요. AppConfig는 구성 영역입니다. 사용영역의 코드 변경없이 구성 영역의 코드가 변경된 것이지요. 추가로 **OCP의 핵심은 사용하는 클라이언트 객체의 코드에 변경이 없다는 것**입니다.

여기서 OrderServiceImpl 코드의 변경 없이 FixDiscountPolicy에서 RateDiscountPolicy로 바꿀 수 있었기 때문에 OCP를 지키게 됩니다.

### Q: AppConfig에 어노테이션으로 @Configuration을 사용하게 되면 그 안에 @Bean 어노테이션으로 등록된 객체들을 스프링 구동 시에 스프링컨테이너가 @Bean 안의 new를 이용해서 인스턴스를 생성 후에 그 인스턴스의 참조값들을 메소드명:참조값 형식으로 컨테이너에 Map처럼 차곡차곡 집어넣어놓는건가요?이게 맞다면 이러한 원리로 같은 참조값만 반환하니까 결과적으로 싱글톤이 되는거구요?

A: 네 맞습니다^^!

# 그외

---

**1) 생성자 주입을 사용해야 하는 이유**

[https://yaboong.github.io/spring/2019/08/29/why-field-injection-is-bad/](https://yaboong.github.io/spring/2019/08/29/why-field-injection-is-bad/)