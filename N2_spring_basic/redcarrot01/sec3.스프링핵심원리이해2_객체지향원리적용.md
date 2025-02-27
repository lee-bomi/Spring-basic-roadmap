# 1. 새로운 할인 정책 개발

### 새로운 할인 정책을 확장해보자

- 기획자 : 주문 금액 당 할인하는 정률% 할인으로 변경하고 싶어요. 예를 들어서 기존 정책은 VIP가 10000원을 주문하든 20000원을 주문하든 항상 1000원을 할인했는데, 이번에 새로 나온 정책은 10%로 지정해두면 고객이 10000원 주문시 1000원을 할인해주고, 20000원 주문시에 2000원을 할인해주는 거에요.
- 개발자 : 애자일 정신으로 유연하고 객체지향적인 설계를 해볼게요.

> 애자일 소프트웨어 개발 선언
>
> 우리는 소프트웨어를 개발하고, 또 다른 사람의 개발을
> 도와주면서 소프트웨어 개발의 더 나은 방법들을 찾아가고
> 있다. 이 작업을 통해 우리는 다음을 가치 있게 여기게 되었다:
>
> 공정과 도구보다 개인과 상호작용을
> 포괄적인 문서보다 작동하는 소프트웨어를
> 계약 협상보다 고객과의 협력을
> 계획을 따르기보다 변화에 대응하기를
>
> 가치 있게 여긴다. 이 말은, 왼쪽에 있는 것들도 가치가 있지만,
> 우리는 오른쪽에 있는 것들에 더 높은 가치를 둔다는 것이다.

#### RateDiscountPolicy 추가

#### RateDiscountPolicy 코드 추가

~~~java
package hello.core.discount;

import hello.core.member.Grade;
import hello.core.member.Member;

public class RateDiscountPolicy implements DiscountPolicy{

    private int discountPercent = 10; // 10% 할인
    @Override
    public int discount(Member member, int price) {

        if (member.getGrade() == Grade.VIP) {
            return price*discountPercent / 100;
        }
        else {
            return 0;
        }
    }
}

~~~



#### 테스트 작성

~~~java
package hello.core.discount;

import hello.core.member.Grade;
import hello.core.member.Member;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;


import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.*;

class RateDiscountPolicyTest {
    RateDiscountPolicy discountPolicy = new RateDiscountPolicy();

    @Test
    @DisplayName("VIP는 10% 할인이 적용되어야 한다")
    void vip_o(){
        // given
        Member member = new Member(1L, "memberVIP", Grade.VIP);

        // when
        int discount = discountPolicy.discount(member, 10000);

        // then
        assertThat(discount).isEqualTo(1000);
    }

    @Test
    @DisplayName("VIP는 10% 할인이 적용되어야 한다")
    void vip_x(){
        // given
        Member member = new Member(2L, "memberBASIC", Grade.BASIC);

        // when
        int discount = discountPolicy.discount(member, 10000);

        // then
        assertThat(discount).isEqualTo(0);
    }
}
~~~



# 2. 새로운 할인 정책 적용과 문제점

### 할인 정책을 애플리케이션에 적용해보자.

~~~java
public class OrderServiceImpl implements OrderService {
// private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
 private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
}
~~~

#### 문제점 발견

- 역할고 구현 분리 -> O

- 다형성 활용하고, 인터페이스와 구현 객체 분리 -> O

- OCP, DIP 같은 객체지향 설계 원칙을 충실히 준수 -> 그렇지 않다.

- **DIP** : 주문서비스 클라이언트(`OrderServiceImpl` )는 `DiscountPolicy` 인터페이스에 의존하면서 DIP를 지킨 것 같은데? 

  - **클래스 의존관계를 분석해 보자. `추상인터페이스`뿐만 아니라 `구체클래스`에도 의존**하고 있다. 
  - 추상(인터페이스) 의존: `DiscountPolicy` 
  - 구체(구현) 클래스: `FixDiscountPolicy` , `RateDiscountPolicy `

- **OCP**: 변경하지 않고 확장할 수 있다고 했는데! 

  - 지금 코드는 기능을 확장해서 변경하면, 클라이언트 코드에 영향을 준다! **따라서 OCP를 위반한다.**

    ~~~java
    // 클라이언트 코드
    // private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
     private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
    ~~~

#### 기대했던 의존관계

<img src="https://user-images.githubusercontent.com/38436013/127828001-3d92c35c-08cb-43b2-a3e3-a07e1da4908d.png" alt="image" style="zoom:80%;" />

지금까지 단순히 DiscountPolicy 인터페이스만 의존한다고 생각했다

#### 실제 의존관계

<img src="https://user-images.githubusercontent.com/38436013/127828043-3d0f7775-5bb7-45ca-bb1c-089acd545efb.png" alt="image" style="zoom:80%;" />

잘보면 클라이언트인 OrderServiceImpl 이 DiscountPolicy 인터페이스 뿐만 아니라 FixDiscountPolicy 인 구체 클래스도 함께 의존하고 있다. 실제 코드를 보면 의존하고 있다! **DIP 위반**

#### 정책 변경

<img src="https://user-images.githubusercontent.com/38436013/127828101-7130408b-7eb3-4f3a-8269-412a1e288444.png" alt="image" style="zoom:80%;" />

 그래서 FixDiscountPolicy 를 RateDiscountPolicy 로 변경하는 순간 OrderServiceImpl 의 소스 코드도 함께 변경해야 한다! **OCP 위반**

### 어떻게 문제를 해결할 수 있을까?

#### 인터페이스에만 의존하도록 설계를 변경하자

DIP를 위반하지 않도록 인터페이스에만 의존하도록 의존관계를 변경하면 된다.

<img src="https://user-images.githubusercontent.com/38436013/127828596-5544b276-5707-4cd4-aa28-9acfae877289.png" alt="image" style="zoom:80%;" />

#### 인터페이스에만 의존하도록 코드 변경

~~~java
public class OrderServiceImpl implements OrderService {
 //private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
 private DiscountPolicy discountPolicy;
}
~~~

- 인터페이스에만 의존하도록 설계와 코드를 변경했다. 
- 구현체 없어서 실제 실행을 해보면 NPE(null pointer exception)가 발생한다.

#### 해결방안

 누군가가 클라이언트인 OrderServiceImpl 에 DiscountPolicy 의 구현 객체를 대신 생성하고 주입해주어야 한다.

# 3. 관심사의 분리

### 관심사를 분리하자

구현 객체를 대신 생성해주고 주입해 줄 수 있는 AppConfig 등장

### AppConfig 등장

이전에는 구현체에서 객체를 생성했지만, appconfig에서 **구현 객체 생**성하고, 생성한 객체를 **생성자를 통해 주입**한다.

#### AppConfig

~~~java
package hello.core;

import hello.core.discount.FixDiscountPolicy;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import hello.core.member.MemoryMemberRepository;
import hello.core.order.OrderService;
import hello.core.order.OrderServiceImpl;

public class AppConfig {
    public MemberService memberService() {
        // 구현 객체 생성 & 생성자 주입
        return new MemberServiceImpl(new MemoryMemberRepository());
    }
    public OrderService orderService() {
        return new OrderServiceImpl(new MemoryMemberRepository(), new FixDiscountPolicy());
    }
}

~~~

- AppConfig는 구현 객체를 생성한다.
  - `MemberServiceImpl `
  - `MemoryMemberRepository `
  - `OrderServiceImpl `
  - `FixDiscountPolicy`
- AppConfig는 생성한 객체 인스턴스의 참조를 생성자를 통해 주입해준다.
  - `MemberServiceImpl` -> `MemoryMemberRepository` 
  - `OrderServiceImpl` -> `MemoryMemberRepository` , `FixDiscountPolicy` 



#### MemberServiceImpl - 생성자 주입

~~~java
package hello.core.member;

public class MemberServiceImpl implements MemberService {

    private final MemberRepository memberRepository;

    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }


    // 다형성에 의해 MemoryMemberRepository의 save, findById 호출
    @Override
    public void join(Member member) {
        memberRepository.save(member);
    }

    @Override
    public Member findMember(Long memberId) {
        return memberRepository.findById(memberId);
    }
}

~~~

- 설계 변경으로 `MemberServiceImpl` 은`MemberRepository`인터페이스만 의존한다.
-  `MemberServiceImpl` 는  생성자를 통해 어떤 구현 객체가 들어올지 알 수 없고, 외부(`AppConfig`)에서 결정된다. 따라서  `MemberServiceImpl`는 **의존관계 고려하지 않고, 실행에만 집중**

#### 그림 - 클래스 다이어그램

<img src="https://user-images.githubusercontent.com/38436013/127845579-7f2122b8-92e1-404c-b5f9-8b484d41aa73.png" alt="image" style="zoom:80%;" />

- 객체의 생성과 연결은 AppConfig가 담당
- **DIP** : MemberServiceImpl 은 MemberRepository 인 추상에만 의존하면 된다. 구체클래스는 몰라도 된다.
- **관심사의 분리** : 객체 생성, 연결의 역할과 실행의 역할이 분리됨

> MemberServiceImpl는 MemberService의 구현체로, MemberRepository 라는 인터페이스를 의존하고 있다. -> DIP 
>
> AppConfig에서 memoryMemberRepository 객체를 생성하고, 이 객체의 참조값을 memberServiceImpl을 생성하면서, 생성자로 전달한다.
>
> 즉, 외부에서 생성과 연결을 해주기 때문에 MemberServiceImpl는 구체 클래스의 변경이 생겨도 OCP를 지킬 수 있다.

#### 그림 - 회원 객체 인스턴스 다이어그램

<img src="C:\Users\yujin\AppData\Roaming\Typora\typora-user-images\image-20210802191506454.png" alt="image-20210802191506454" style="zoom:80%;" />

- `appConfig` 객체는 `memoryMemberRepository` 객체 생성하고, 그 참조값을 `memberServiceImpl`을 생성하면서, 생성자로 전달한다.
- 클라이언트인 `memberServiceImpl` 입장에서 보면 `의존관계를 외부에서 주입해주는 모양새`이므로, **DI(Dependency Injection)** 우리말로 **의존관계 주입**이라고 한다.



#### OrderServiceImpl - 생성자 주입

~~~JAVA
package hello.core.order;

import hello.core.discount.DiscountPolicy;
import hello.core.member.Member;
import hello.core.member.MemberRepository;


public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository ;
    private final DiscountPolicy discountPolicy ;

    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }


    // 좋은 설계 - 단일 체계 원칙이 잘 지켜짐
    // discountPolicy로 인해 할인 정책을 변경하더라도, 주문 서비스(구현체)는 신경x
    // discountPolicy가 없고, 할인 정책 변경이 있으면, 주문 서비스(구현체)도 변경해야 함
    @Override
    public Order createOrder(Long memberId, String itemName, int itemPrice) {
        // 회원 정보 조회하고, 할인 정책에 회원 넘김
        Member member = memberRepository.findById(memberId);
        int discountPrice = discountPolicy.discount(member, itemPrice);

        return new Order(memberId, itemName, itemPrice, discountPrice);
    }
}

~~~

- 설계 변경으로 `OrderServiceImpl` 은 `FixDiscountPolicy` 를 의존하지 않는다!
- `OrderServiceImpl` 에는 `MemoryMemberRepository` , `FixDiscountPolicy` 객체의 의존관계가 주입 된다.

### AppConfig 실행

#### 사용 클래스 - MemberApp 수정

~~~java
 public static void main(String[] args){
        AppConfig appConfig = new AppConfig();
        MemberService memberService = appConfig.memberService();
        //MemberService memberService = new MemberServiceImpl();
~~~

#### 사용 클래스 - OrderApp 수정

~~~java
public class OrderApp {
    public static void main(String[] args) {

        AppConfig appConfig = new AppConfig();
        MemberService memberService = appConfig.memberService();
        OrderService orderService = appConfig.orderService();
~~~

#### 테스트 코드 오류 수정

~~~java
public class MemberServiceTest {

    MemberService memberService ;

    @BeforeEach
    public void beforeEach() {
        AppConfig appConfig = new AppConfig();
        memberService = appConfig.memberService();
    }
~~~

~~~java
public class OrderServiceTest {
    MemberService memberService ;
    OrderService orderService ;

    @BeforeEach
    public void beforeEach() {
        AppConfig appConfig = new AppConfig();
        memberService = appConfig.memberService();
        orderService = appConfig.orderService();
    }
~~~

> 이전에는 `MemberService memberService = new MemberServiceImpl();`  메인에서 직접 객체를 생성했다면, 메인에서 AppConfig 객체를 생성하여, 서비스(memberService를 받아온다. 이때, 서비스의 추상(MemberServiceImpl)을 호출하고, 추상의 구현체를 받아온다.

### 정리

- AppConfig 통해 관심사 분리했고, 객체 생성과 연결을 담당한다.
- OrderServiceImpl 은 기능을 실행하는 책임만 지면 된다.



# 4. AppConfig 리팩토링

현재 AppConfig를 보면 **중복**이 있고, **역할에 따른 구현**이 잘 안보인다.

#### 리팩토링 전

~~~java
package hello.core;
import hello.core.discount.FixDiscountPolicy;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import hello.core.member.MemoryMemberRepository;
import hello.core.order.OrderService;
import hello.core.order.OrderServiceImpl;
public class AppConfig {
 	public MemberService memberService() {
 		return new MemberServiceImpl(new MemoryMemberRepository());
 	}
 	public OrderService orderService() {
 		return new OrderServiceImpl(
 			new MemoryMemberRepository(),
 			new FixDiscountPolicy());
 	}
}
~~~



#### 리팩토링 후

~~~java
package hello.core;

import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.member.MemberRepository;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import hello.core.member.MemoryMemberRepository;
import hello.core.order.OrderService;
import hello.core.order.OrderServiceImpl;

public class AppConfig {
    // 역할이 드러나게 리팩토링하기, 인터페이스와 구체 클래스를 한눈에 확인 가능하다.
    public MemberService memberService() {
        return new MemberServiceImpl(MemberRepository());
    }

    private MemberRepository MemberRepository() {
        return new MemoryMemberRepository();
    }

    public OrderService orderService() {
        return new OrderServiceImpl(MemberRepository(), discountPolicy());
    }
    public DiscountPolicy discountPolicy(){
        return new FixDiscountPolicy();
    }
}

~~~

- `new MemoryMemberRepository()` 이 부분이 중복 제거되었다. 
- 이제 `MemoryMemberRepository` 를 다른 구현체로 변경할 때 한 부분만 변경하면 된다. 
- `AppConfig` 를 보면 역할과 구현 클래스가 한눈에 들어온다. 애플리케이션 전체 구성이 어떻게 되어있는지 빠르게 파악할 수 있다.



# 5. 새로운 구조와 할인 정책 적용

- 정액 할인 정책을 정률 할인 정책으로 변경하자
- 어떤 부분을 변경할까?

<img src="https://user-images.githubusercontent.com/38436013/127893289-1b452acc-a84d-4f54-8c3b-13fc3d084762.png" alt="image" style="zoom:80%;" />

#### 할인 정책 변경 구성 코드

~~~JAVA
public class AppConfig {
    // 역할이 드러나게 리팩토링하기, 인터페이스와 구체 클래스를 한눈에 확인 가능하다.
    
    public DiscountPolicy discountPolicy(){
        //return new FixDiscountPolicy();
        return new RateDiscountPolicy();
    }
}
~~~

- 이제 할인 정책을 변경해도, 애플리케이션의 구성 역할을 담당하는 AppConfig만 변경하면 된다.
- 사용 영역은 변경X, 구성 영역은 당연히 변경된다.



# 6. 전체 흐름 정리

| 새로운 할인 정책 개발        | 정률 할인 정책 코드 추가 개발에는 다형성 덕분에 문제 없음    |
| ---------------------------- | ------------------------------------------------------------ |
| 새로운 할인 정책 개발 문제점 | 1. 클라이언트 코드 변경 <BR>2. 클라이언트 인터페이스가 DiscountPolicy 뿐만 아니라, 구체 클래스인 FixDiscountPolicy 도 함께 의존 -> **DIP 위반** |
| 관심사 분리                  | AppConfig 등장 <BR>AppConfig는 전체 동작 방식 구성을 위해, 구현 객체를 생성하고, 연결 담당<BR>클라이언트 객체는 실행에만 집중 |
| AppConfig 리팩터링           | 역할과 구현을 명확하게 분리(AppConfig는 전체 동작 확인), 중복 제거 |
| 새로운 구조와 할인 정책 적용 | 정액 할인 정책 -> 정률 할인 정책으로 변경<BR>AppConfig의 등장으로 애플리케이션이 크게 **사용 영역**과, 객체를 생성하고 **구성(Configuration)하는 영역**으로 분리 |



# 7. 좋은 객체 지향 설계의 5가지 원칙의 적용

여기서 3가지 SRP, DIP, OCP 적용

### SRP 단일 책임 원칙

**한 클래스는 하나의 책임만 가져야 한다.**

- 클라이언트 객체는 직접 구현 객체를 생성하고, 연결하고, 실행하는 다양한 책임을 가지고 있음
- SRP 단일 책임 원칙을 따르면서 관심사를 분리함
- 구현 객체를 생성하고 연결하는 책임은 AppConfig가 담당
- **클라이언트 객체는 실행하는 책임만 담당**

### DIP 의존관계 역전 원칙

**프로그래머는 “추상화에 의존해야지, 구체화에 의존하면 안된다.” 의존성 주입은 이 원칙을 따르는 방법 중 하나다.**

- 새로운 할인 정책을 개발하고, 적용하려고 하니 클라이언트 코드도 함께 변경해야 했다. 왜냐하면 기존 클라 이언트 코드( OrderServiceImpl )는 DIP를 지키며 DiscountPolicy 추상화 인터페이스에 의존하는 것 같았지만, FixDiscountPolicy 구체화 구현 클래스에도 함께 의존했다.
- **클라이언트 코드가 DiscountPolicy 추상화 인터페이스에만 의존**하도록 코드를 변경했다.
- 하지만 클라이언트 코드는 인터페이스만으로는 아무것도 실행할 수 없다.
- **AppConfig가 FixDiscountPolicy 객체 인스턴스를 생성해서 클라이언트 코드에 의존관계를 주입**했다. 이렇게해서 DIP 원칙을 따르면서 문제도 해결했다.

### OCP

#### **소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀 있어야 한다**

- 다형성 사용하고 클라이언트가 DIP를 지킴
- 애플리케이션을 **사용 영역과 구성 영역으로 나눔**
- AppConfig가 의존관계를 FixDiscountPolicy RateDiscountPolicy 로 변경해서 클라이언트 코드에 주입하므로 `**클라이언트 코드는 변경하지 않아도 됨**` 
- **소프트웨어 요소를 새롭게 확장해도 사용 역영의 변경은 닫혀 있다!**



# 8. IoC, DI, 컨테이너

### 제어의 역전 IoC(Inversion of Control)

- 기존에는 클라이언트 구현 객체가 구현 객체 생성, 연결, 실행했지만, AppConfig 등장으로 실행하는 역할만 수행
- 프로그램에 대한 제어 흐름에 대한 권한은 모두 AppConfig가 가지고 있다.
- **프로그램의 제어 흐름을 외부에서 관리하는 것을 제어의 역전(IoC)**이라 한다.

### 프레임워크 vs 라이브러리

- 프레임워크가 내가 작성한 코드를 제어하고, 대신 실행하면 그것은 프레임워크가 맞다. (JUnit) -> 제어권이 외부에 !
- 반면에 내가 작성한 코드가 직접 제어의 흐름을 담당한다면 그것은 프레임워크가 아니라 라이브러리다.

### 의존관계 주입 DI(Dependency Injection)

- OrderServiceImpl 은 DiscountPolicy 인터페이스에 의존한다. 실제 어떤 구현 객체가 사용될지는 모른다.
- 의존관계는 **정적인 클래스 의존 관계와, 실행 시점에 결정되는 동적인 객체(인스턴스) 의존 관계** 둘을 분리해서 생각해야 한다.

### 정적인 클래스 의존관계

클래스가 사용하는 import 코드만 보고 의존관계를 쉽게 판단할 수 있다. 정적인 의존관계는 애플리케이션을 실행하지 않아도 분석할 수 있다. 클래스 다이어그램을 보자
OrderServiceImpl 은 MemberRepository , DiscountPolicy 에 의존한다는 것을 알 수 있다. 그런데 이러한 클래스 의존관계 만으로는 실제 어떤 객체가 OrderServiceImpl 에 주입 될지 알 수 없다.

**클래스 다이어그램**

<img src="https://user-images.githubusercontent.com/38436013/127897742-83687d49-4889-4e9e-8ff6-446ab9592225.png" alt="image" style="zoom:80%;" />

> OrderServiceImpl 코드를 보면, OrderService를 참조, MemberRepository , DiscountPolicy 를 참조
>
> FixDiscountPolicy를 보면  DiscountPolicy 상속하고 있음 

### 동적인 객체 인스턴스 의존 관계

애플리케이션 실행 시점에 실제 생성된 객체 인스턴스의 참조가 연결된 의존 관계다.

**객체 다이어그램**

![image](https://user-images.githubusercontent.com/38436013/127898806-ddbc991b-8aa1-43de-8568-3c8f3a644512.png)

- 애플리케이션 **실행 시점(런타임)**에 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달해서 클라이언트와 서버의 실제 의존관계가 연결 되는 것을 **의존관계 주입**이라 한다.
- `객체 인스턴스를 생성하고, 그 참조값을 전달해서 연결`된다.
- 의존관계 주입을 사용하면 클라이언트 코드를 변경하지 않고, 클라이언트가 호출하는 대상의 타입 인스턴스를 변경할 수 있다.
- **의존관계 주입을 사용하면 정적인 클래스 의존관계를 변경하지 않고, 동적인 객체 인스턴스 의존관계를 쉽게 변경할 수 있다.**

### IoC 컨테이너, DI 컨테이너

- AppConfig 처럼 객체를 생성하고 관리하면서 의존관계를 연결해 주는 것을
- IoC 컨테이너 또는 **DI 컨테이너**라 한다.
- 의존관계 주입에 초점을 맞추어 최근에는 주로 DI 컨테이너라 한다.
- 또는 어샘블러, 오브젝트 팩토리 등으로 불리기도 한다.

# 9. 스프링으로 전환하기

####  AppConfig 스프링 기반으로 변경

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
    // 역할이 드러나게 리팩토링하기, 인터페이스와 구체 클래스를 한눈에 확인 가능하다.
    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(MemberRepository());
    }

    @Bean
    public MemberRepository MemberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(MemberRepository(), discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy(){
        //return new FixDiscountPolicy();
        return new RateDiscountPolicy();
    }
}
~~~

AppConfig에 설정을 구성한다는 뜻의 @Configuration 을 붙여준다. 

각 메서드에 @Bean 을 붙여준다. 이렇게 하면 스프링 컨테이너에 스프링 빈으로 등록한다

#### MemberApp에 스프링 컨테이너 적용

~~~java
public class MemberApp {

    public static void main(String[] args){
        //AppConfig appConfig = new AppConfig();
        //MemberService memberService = appConfig.memberService();

        // 스프링 코드, 스프링은 AppConfig를 가지고, @Bean이 등록된 객체 인스턴스를 스프링 컨테이너에 등록하고 관리해줌
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
        // AppConfig에서 memberService 객체를 MemberService.class에서 찾을 거야 - 이게 가능한 것은 바로 위 스프링 코드 때문
        MemberService memberService = applicationContext.getBean("memberService", MemberService.class);

        Member member = new Member(1L, "memberA", Grade.VIP);
        memberService.join(member);

        // 등록한 사람이 가입되는지 확인하기
        Member findMember = memberService.findMember(1L);
        System.out.println("new member =" + member.getName());
        System.out.println("find member =" + findMember.getName());
    }
}
~~~



#### OrderApp에 스프링 컨테이너 적용

~~~java
public class OrderApp {
    public static void main(String[] args) {

//        AppConfig appConfig = new AppConfig();
//        MemberService memberService = appConfig.memberService();
//        OrderService orderService = appConfig.orderService();

        ApplicationContext applicationContext = new
                AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService =
                applicationContext.getBean("memberService", MemberService.class);
        OrderService orderService = applicationContext.getBean("orderService",
                OrderService.class);

        Long memberId = 1L;
        Member member = new Member(memberId, "memberA", Grade.VIP);
        memberService.join(member);

        Order order = orderService.createOrder(memberId, "itemA", 10000);

        System.out.println("order = "+ order);
        System.out.println("order.calculatePrice = "+ order.calculatePrice());
    }

~~~



#### 스프링 컨테이너

- ApplicationContext 를 스프링 컨테이너라 한다.
- 기존에는 개발자가 AppConfig 를 사용해서 직접 객체를 생성하고 DI를 했지만, 이제부터는 스프링 컨테이너를 통해서 사용한다.
- 스프링 컨테이너는 @Configuration 이 붙은 AppConfig 를 설정(구성) 정보로 사용한다. 여기서 @Bean이라 적힌 메서드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록한다. 이렇게 스프링 컨테이너에 등록된 객체를 스프링 빈이라 한다.
- 스프링 빈은 @Bean 이 붙은 메서드의 명을 스프링 빈의 이름으로 사용한다. ( memberService ,orderService )
- 이전에는 개발자가 필요한 객체를 AppConfig 를 사용해서 직접 조회했지만, **이제부터는 스프링 컨테이너를 통해서 필요한 스프링 빈(객체)를 찾아야 한다.** 스프링 빈은 applicationContext.getBean() 메서드를 사용해서 찾을 수 있다.
- 기존에는 개발자가 직접 자바코드로 모든 것을 했다면 이제부터는 스프링 컨테이너에 객체를 스프링 빈으로 등록하고, 스프링 컨테이너에서 스프링 빈을 찾아서 사용하도록 변경되었다.