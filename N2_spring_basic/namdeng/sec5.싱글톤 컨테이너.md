# 내용

---

## 웹 애플리케이션과 싱글톤

- 스프링은 태생이 기업용 온라인 서비스 기술을 지원하기 위해 탄생했다
- 대부분의 스프링 애플리케이션은 웹 애플리케이션이다. 물론 웹이 아닌 애플리케이션 개발도 얼마든지 개발할 수 있다.
- 웹 애플리케이션은 보통 여러 고객이 동시에 요청을 한다.

### 스프링 없는 순수한 DI 컨테이너

1. **테스트 코드 ( 결과 : 실패 , 객체 인스턴스가 다르다. )**

```java
AppConfig appConfig = new AppConfig();
MemberService memberService1 = appConfig.memberService();
MemberService memberService2 = appConfig.memberService();

assertThat(memberService1).isSameAs(memberService2);
```

- 순수한 DI 컨테이너는 요청을 할 때 마다 객체를 새로 생성한다. 이는 메모리 낭비로 이어진다.
- 해결방안은 해당 객체가 딱 1개만 생성되고, 공유하도록 싱글톤 패턴을 적용하자.

### 싱글톤 패턴

1. **싱글톤 패턴 구현 코드**

```java
class SingletonService {
    private static final SingletonService instance = new SingletonService();

    public static SingletonService getInstance() {
        return instance;
    }

    private SingletonService() { 
        
    }
}
```

2. **테스트 코드 ( 결과 : 성공, 객체 인스턴스가 동일하다. )**

```java
SingletonService singletonService1 = SingletonService.getInstance();
SingletonService singletonService2 = SingletonService.getInstance();

assertThat(singletonService1).isSameAs(singletonService2);
```

싱글톤 패턴을 적용하면 만들어진 객체를 공유해서 효율적으로 사용할 수 있다. 하지만 싱글톤 패턴은 다음과 같은 많은 문제점들을 가지고 있다.

3. **싱글톤 패턴 문제점**
    - 싱글톤 패턴을 구현하는 코드 자체가 많이 들어간다.
    - 의존관계상 클라이언트가 구체 클래스에 의존한다. → DIP를 위반한다.
    - 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다.
    - 결론적으로 유연성이 떨어진다


### 스프링 컨테이너

1. **테스트 코드 ( 결과 : 성공, 객체 인스턴스가 동일하다. )**

```java
ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
MemberService memberService1 = ac.getBean("memberService", MemberService.class);
MemberService memberService2 = ac.getBean("memberService", MemberService.class);

assertThat(memberService1).isSameAs(memberService2);
```

2. **싱글톤 컨테이너**
    - 스프링 컨테이너는 싱글턴 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리한다.
    - 스프링 컨테이너는 싱글톤 컨테이너 역할을 한다. 이렇게 싱글톤 객체를 생성하고 관리하는 기능을 싱글톤 레지스트리라 한다.
    - 스프링 컨테이너의 이런 기능 덕분에 싱글턴 패턴의 모든 단점을 해결하면서 객체를 싱글톤으로 유지할 수 있다.


### 싱글톤 방식의 주의사항

- 객체 인스턴스를 하나만 생성해서 공유하는 싱글톤 방식은 여러 클라이언트가 하나의 같은 인스턴스를 공유하기 때문에 싱글톤 객체는 상태를 유지(stateful)하게 설계하면 안된다.
- 무상태(stateless)로 설계해야 한다!
    - 특정 클라이언트에 의존적인 필드가 있으면 안된다.
    - 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다!
    - 가급적 읽기만 가능해야 한다.
    - 필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용하자.