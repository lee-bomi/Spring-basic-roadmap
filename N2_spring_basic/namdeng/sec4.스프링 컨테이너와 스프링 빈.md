# 내용

---

## 스프링 컨테이너

- 스프링 컨테이너의 종류에는 BeanFactory, ApplicationContext 가 있다.
- BeanFactory는 **스프링 빈 객체를 관리하고 조회하는 역할을 담당한다.**
- ApplicationContext 는 BeanFactory를 상속 받아 빈 객체를 관리하고 조회하는 기능을 가지고 있으며, 추가적으로 트랜잭션 관리, 메시지 기반의 다국어 처리, 편리한 리소스 조회 등의  기능도 가지고 있다.
- **ApplicationContext**는 **컨테이너가 구동되는 시점에 객체를 생성하는 Pre-Loading 방식**이나, **BeanFactory**는 컨테이너가 구동될때 빈 객체가 생성되지 않고 **클라이언트의 요청에 의해서 빈 이 사용되는 시점(Lazy Loading)**에 빈 객체를 생성하는 방식을 사용하고 있다.
- BeanFactory 를 직접 사용하는 경우는 거의 없으므로 일반적으로 ApplicationContext 를 스프링 컨테이너라 한다.
- 스프링 컨테이너는 XML을 기반으로 만들 수 있고, 애노테이션 기반의 자바 설정 클래스로 만들 수 있다.

![beanFactory](https://user-images.githubusercontent.com/17094674/127888082-68a2fd91-7511-4c0c-812a-1f3e1030affc.png)

## 스프링 컨테이너 생성

### **1) 애노테이션**

- **구성 정보**

```java
@Configuration
public class AppConfig {
		@Bean
		public MemberService memberService() {
        System.out.println("AppConfig.memberService");
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        System.out.println("AppConfig.memberRepository");
        return new MemoryMemberRepository();
    }
}
```

- **스프링컨테이너 생성**

```java
ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
MemberRepository memberRepository = ac.getBean("memberRepository", MemberRepository.class);

assertThat(memberService).isInstanceOf(MemberService.class);
```

### **2) XML**

- **구성 정보**

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="memberService" class="spring.demo.member.MemberServiceImpl">
        <constructor-arg name="memberRepository" ref="memberRepository" />
    </bean>

    <bean id="memberRepository" class="spring.demo.member.MemoryMemberRepository">
    </bean>
</beans>
```

- **스프링 컨테이너 생성**

```java
ApplicationContext ac = new GenericXmlApplicationContext("appConfig.xml");
MemberService memberService = ac.getBean("memberService", MemberService.class);

assertThat(memberService).isInstanceOf(MemberService.class);
```

## 스프링 컨테이너의 장점

- 스프링 빈을 등록하면 스프링 컨테이너가 구성 정보를 참조해 의존성 주입(DI)를 해준다. 이로 인해 클라이언트는 의존 관계를 신경쓰지 않아도 되고 실행에만 집중할 수 있게 되는것 같다.
- 스프링 컨테이너가 스프링 빈을 싱글톤으로 관리 해준다.
