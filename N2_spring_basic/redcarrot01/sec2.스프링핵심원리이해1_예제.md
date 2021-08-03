# 1. 프로젝트 생성

### 사전 준비물

- Java 11 설치

### 프로젝트 생성

https://start.spring.io/

### 프로젝트 선택

- Project: Gradle Project 
  - Spring Boot: 2.5.x 
  - Language: Java 
  - Packaging: Jar 
  - Java: 11 
- Project Metadata 
  - groupId: hello 
  - artifactId: core 
- Dependencies: 선택하지 않는다.

# 2. 비즈니스 요구사항과 설계

- 회원
  - 회원을 가입하고 조회할 수 있다.
  - 회원은 일반과 VIP 두가지 등급이 있다.
  - 회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다. (미확정)
- 주문과 할일 정책
  - 회원은 상품을 주문할 수 있다.
  - 회원 등급에 따라 할인 정책을 적용할 수 있다.
  - 할인 정책은 모든 VIP는 1000원을 할인해주는 고정 금액 할인을 적용해달라. (나중에 변경 될 수 있다.)
  - 할인 정책은 변경 가능성이 높다. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 고민을 미루고 싶다. 최악의 경우 할인을 적용하지 않을 수 도 있다. (미확정)

요구 사항을 보면 현재 결정하기 어려운 부분들이 있지만, 인터페이스를 만들고 구현체를 언제든 갈아끼울 수 있게 설계하면 문제를 해결할 수 있다.

# 3. 회원 도메인 설계

- 회원 도메인 요구사항
  - 회원을 가입하고 조회할 수 있다. 
  - 회원은 일반과 VIP 두 가지 등급이 있다. 
  - 회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다. (미확정)

#### 회원 도메인 협력 관계 : 기획자가 참고

![image](https://user-images.githubusercontent.com/38436013/126977572-61f0f92c-45b0-426f-a524-c9de8b1a693e.png)

#### 회원 클래스 다이어그램 : 개발자가 참고

![image](https://user-images.githubusercontent.com/38436013/126977595-2fc47749-b5cd-4936-b16c-f2ee9a54f6bc.png)

#### 회원 객체 다이어그램 : 서버 떴을 때 클라이언트가 실제로 사용하는 참조

![image](https://user-images.githubusercontent.com/38436013/126973706-356b5fcf-8431-4148-b322-e3c44ff2f804.png)

# 4. 회원 도메인 개발

getter and setter, constructor.. 자동 생성 단축키 윈도우 버전 : alt + ins

자동 생성 + ;  : ctrl + shift + enter



*회원 클래스 다이어 그램을 기반으로 구현*

### 회원 엔티티

#### 회원 등급

~~~java
package hello.core.member;

public enum Grade {
    BASIC,
    VIP
}
~~~



#### 회원 엔티티

~~~java
package hello.core.member;

public class Member {

    private Long id;
    private String name;
    private Grade grade;

    public Member(Long id, String name, Grade grade) {
        this.id = id;
        this.name = name;
        this.grade = grade;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Grade getGrade() {
        return grade;
    }

    public void setGrade(Grade grade) {
        this.grade = grade;
    }
}

~~~



### 회원 저장소

#### 회원 저장소 인터페이스

~~~java
package hello.core.member;

public interface MemberRepository {

    void save(Member member);
    Member findById(Long memberId);
}
~~~



#### 메모리 회원 저장소 구현체

~~~java
package hello.core.member;

import java.util.HashMap;
import java.util.Map;

public class MemoryMemberRepository implements MemberRepository{

    private static Map<Long, Member> store = new HashMap<>();
    @Override
    public void save(Member member) {
        store.put(member.getId(), member);
    }

    @Override
    public Member findById(Long memberId) {
        return store.get(memberId);
    }
}

~~~



### 회원 서비스

#### 회원 서비스 인터페이스

~~~java
package hello.core.member;

public interface MemberService {
    void join(Member memeber);
    Member findMember(Long memberId);
}

~~~



#### 회원 서비스 구현체

~~~java
package hello.core.member;

public class MemberServiceImpl implements MemberService {

    private final MemberRepository memberRepository = new MemoryMemberRepository();
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



# 5. 회원 도메인 실행과 테스트

*회원 객체 다이어그램 기반으로 구현*

### 회원 도메인 실행과 테스트

#### 회원 도메인 - 회원 가입 main

~~~java
package hello.core;

import hello.core.member.Grade;
import hello.core.member.Member;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;

public class MemberApp {

    public static void main(String[] args){
        MemberService memberService = new MemberServiceImpl();
        Member member = new Member(1L, "memberA", Grade.VIP);
        memberService.join(member);

        // 등록한 사람이 가입되는지 확인하기
        Member findMember = memberService.findMember(1L);
        System.out.println("new member =" + member.getName());
        System.out.println("find member =" + findMember.getName());
    }
}

~~~

순수 자바로 조인이 되었는지 확인했으나, 애플리케이션 로직보다 test를 이용하는 것이 좋다.

JUnit테스트 이용하자!

#### 회원 도메인 - 회원 가입 테스트

~~~java
package hello.core.member;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;

public class MemberServiceTest {

    MemberService memberService = new MemberServiceImpl();

    @Test
    void join() {
        // given
        Member member = new Member(1L, "memberA", Grade.VIP);

        // when
        memberService.join(member);
        Member findMember = memberService.findMember(1L);

        // then
        Assertions.assertThat(member).isEqualTo(findMember);
    }
}

~~~



#### 회원 도메인 설계의 문제점

- 이 코드의 설계상 문제점은 무엇일까?
- 다른 저장소 변경할 때 OCP 원칙 잘 지키고 있는가?
- DIP는 지키고 있는가?
- 의존관계가 인터페이스 뿐 아니라 구현까지 모두 의존하는 문제점이 있다.

~~~java
public class MemberServiceImpl implements MemberService {

    private final MemberRepository memberRepository = new MemoryMemberRepository();
    }
~~~

 MemberServiceImpl는 MemberRepository, MemoryMemberRepository 모두 의존하고 있다. -> DIP 위반



# 6. 주문과 할인 도메인 설계

- 주문과 할인 정책
  - 회원은 상품 주문 가능
  - 회원 등급에 따라 할인 정책을 적용할 수 있다.
    - 할인 정책은 모든 VIP는 1000원을 할인해주는 고정 금액 할인을 적용하자.(변경 가능)
    - 할인 정책은 변경 가능성이 높다. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 고민을 미루고 싶다. (미확정) 



### 주문 도메인 협력, 역할, 책임

역할만 그림으로 표현

![image](https://user-images.githubusercontent.com/38436013/127806729-8677c1ad-596d-492e-a8ad-d8f8cc156e60.png)

1. **주문 생성**: 클라이언트는 주문 생성을 요청한다.
2. **회원 조회**: 할인을 위해서는 회원 등급이 필요하고, 주문 서비스는 회원저장소에서 회원을 조회한다.
3. **할인 적용**: 주문 서비스는 회원 등급에 따른 할인 여부를 할인 정책에 위임한다. 
4. **주문 결과 반환**: 주문 서비스는 할인 결과를 포함한 주문 결과를 반환한다.



### 주문 도메인 전체

역할과 구현 그림으로 표현

정액 할인 : vip는 1000원 할인,  정률 확인 : 금액에 대한 10%, 20% 할인 

![image](https://user-images.githubusercontent.com/38436013/127806739-dc421bbd-7cc3-47d0-871f-136b22ac3c82.png)

**역할과 구현을 분리**해서 자유롭게 구현 객체를 조립할 수 있게 설계

회원저장소나 할인 정책 변경 가능

### 주문 도메인 클래스 다이어그램

![image](https://user-images.githubusercontent.com/38436013/127806753-93218780-2792-4369-baf4-8dcb14a97fd1.png)

### 주문 도메인 객체 다이어그램1

![image](https://user-images.githubusercontent.com/38436013/127806771-96267a68-f4e2-484a-8739-29f72f8bc570.png)

![image](https://user-images.githubusercontent.com/38436013/127806785-b834f0cf-150d-48b9-9801-8bc0628434b1.png)

회원을 메모리에서 조회하고, 정액 할인 정책을 지원해도 주문 서비스를 변경하지 않아도 된다. 역할들의 협력 관계를 그대로 재사용 할 수 있다.

협력 관계를 그대로 재사용 가능하다.

# 7. 주문과 할인 도메인 개발

#### 할인 정책 인터페이스

~~~java
package hello.core.discount;

import hello.core.member.Member;

public interface DiscountPolicy {
    /**
     * @return 할인 대상 금액
     */
    int discount(Member member, int price) ;
}

~~~



#### 정책 할인 정책 구현체

~~~java
package hello.core.discount;

import hello.core.member.Grade;
import hello.core.member.Member;

public class FixDiscountPolicy implements DiscountPolicy{
    private int discountFixAmount = 1000; // 1000원 할인

    // VIP 1000원 할인, 아니면 할인 없음
    @Override
    public int discount(Member member, int price) {
        if(member.getGrade()== Grade.VIP){
            return discountFixAmount;
        }
        else {
            return 0;
        }
    }
}

~~~



#### 주문 엔티티

~~~java
package hello.core.order;

public class Order {

    private Long memberId;
    private String itemName;
    private int itemPrice;
    private int discountPrice;

    public Order(Long memberId, String itemName, int itemPrice, int discountPrice) {
        this.memberId = memberId;
        this.itemName = itemName;
        this.itemPrice = itemPrice;
        this.discountPrice = discountPrice;
    }
    public int calculatePrice() {
        return itemPrice - discountPrice;
    }

    public Long getMemberId() {
        return memberId;
    }

    public void setMemberId(Long memberId) {
        this.memberId = memberId;
    }

    public String getItemName() {
        return itemName;
    }

    public void setItemName(String itemName) {
        this.itemName = itemName;
    }

    public int getItemPrice() {
        return itemPrice;
    }

    public void setItemPrice(int itemPrice) {
        this.itemPrice = itemPrice;
    }

    public int getDiscountPrice() {
        return discountPrice;
    }

    public void setDiscountPrice(int discountPrice) {
        this.discountPrice = discountPrice;
    }

    @Override
    public String toString() { // 객체 출력하면 toString 출력하기
        return "Order{" +
                "memberId=" + memberId +
                ", itemName='" + itemName + '\'' +
                ", itemPrice=" + itemPrice +
                ", discountPrice=" + discountPrice +
                '}';
    }
}

~~~



#### 주문 서비스 인터페이스

~~~java
package hello.core.order;

public interface OrderService {
    Order createOrder (Long memberId, String itemName,int itemPrice );
    // 최종 주문 결과를 반환
}

~~~



#### 주문 서비스 구현체

~~~java
package hello.core.order;

import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.member.Member;
import hello.core.member.MemberRepository;
import hello.core.member.MemoryMemberRepository;



public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository = new MemoryMemberRepository();
    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();


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

주문 생성 요청오면, 회원 조회하고 회원 정보 할인 정책에 넘겨, 주문 객체를 생성하고, 반환한다



# 8. 주문과 할인 도메인 실행과 테스트

인텔리제이 - psvm 치면 아래가 자동생성 - 꿀팁

```
public static void main(String[] args) {
    
}
```



#### 주문과 할인 정책 실행 - main

~~~java
package hello.core;

import hello.core.member.Grade;
import hello.core.member.Member;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import hello.core.order.Order;
import hello.core.order.OrderService;
import hello.core.order.OrderServiceImpl;

public class OrderApp {
    public static void main(String[] args) {

        MemberService memberService = new MemberServiceImpl();
        OrderService orderService = new OrderServiceImpl();

        Long memberId = 1L;
        Member member = new Member(memberId, "memberA", Grade.VIP);
        memberService.join(member);

        Order order = orderService.createOrder(memberId, "itemA", 10000);

        System.out.println("order = "+ order);
        System.out.println("order.calculatePrice = "+ order.calculatePrice());
    }

}

~~~

> 결과
>
> order = Order{memberId=1, itemName='itemA', itemPrice=10000, discountPrice=1000}
> order.calculatePrice = 9000



#### 주문과 할인 정책 테스트

~~~java
package hello.core.order;

import hello.core.member.Grade;
import hello.core.member.Member;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;

public class OrderServiceTest {
    MemberService memberService = new MemberServiceImpl();
    OrderService orderService = new OrderServiceImpl();

    @Test
    void createOrder() {
        long memberId = 1L;
        Member member = new Member(memberId, "memberA", Grade.VIP);
        memberService.join(member);

        Order order = orderService.createOrder(memberId, "ItemA", 10000);
        Assertions.assertThat(order.getDiscountPrice()).isEqualTo(1000);
    }
}

~~~



## Feedback

지금까지는 다형성을 잘 적용했지만,  (할인 정책을 바꾼다면)정률 할인 정책으로 바꾼다면, 지금처럼 스프링 기본 원칙들을 지켜나갈 수 있는지에 초점을 맞추고 다음 시리즈에서 개발해보자.

