# 내용

---

## 1️⃣ 회원 요구사항과 설계

### 1) 요구사항

- 회원 가입, 조회 기능이 있다.
- 회원 등급은 VIP, 일반 등급이 있다.
- 회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다 **( 미확정 )**

### 2) 설계

**회원 도메인 협력 관계**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/33f55985-5303-4c8d-ab58-48833f02764f/도메인.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/33f55985-5303-4c8d-ab58-48833f02764f/도메인.png)

**회원 클래스 다이어그램**

클래스 다이어그램은 모델링 된 시스템 구조를 보여주기 위한 정적 다이어그램 유형이다. 클래스 다이어그램에서 어떤 구현체가 동작할 지는 알 수 없다

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/366e595f-5620-465e-9f47-8b8fe0f54b1d/회원클래스.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/366e595f-5620-465e-9f47-8b8fe0f54b1d/회원클래스.png)

**회원 객체 다이어그램**

객체 다이어그램은 특정 순간에 객체와 객체 간의 관계를 나타낸다. 클라이언트가 사용하는 구현체를 알 수 있다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e9316ca2-2496-434e-8da3-024506f69e8b/객체_다이어그램.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e9316ca2-2496-434e-8da3-024506f69e8b/객체_다이어그램.png)

## 2️⃣ 주문 요구사항과 설계

### ⭐ 주문과 할인 정책

- 상품 주문 기능이 있다.
- 회원 등급에 따라 할인 정책을 적용할 수 있다.
- VIP 등급은 1000원을 할인해주는 고정 금액 할인을 적용. **( 추후에 변경 될 수 있음 )**
- 할인 정책은 **변경 가능성이 높다.** 회사의 기본 할인 정책을 정하지 못했고, 오픈 직전까지 고민을 미루고 싶다. 최악의 경우 할인을 적용하지 않을 수 있다.

### ⭐ 설계

**주문 도메인 협력 관계**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/833df191-b1c9-4062-9327-8d3ae4842615/주문-도메인.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/833df191-b1c9-4062-9327-8d3ae4842615/주문-도메인.png)

1. **주문 생성** : 클라이언트는 주문 서비스에 주문 생성을 요청한다.
2. **회원 조회** : 할인을 위해서는 회원 등급이 필요하다. 그래서 주문 서비스는 회원 저장소에서 회원을 조회한다.
3. **할인 적용** : 주문 서비스는 회원 등급에 따른 할인 여부를 할인 정책에 위임한다.
4. **주문 결과 반환** : 주문 서비스는 할인 결과를 포함한 주문 결과를 반환한다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e9a0abe5-76fc-4500-8fd2-985d49523448/주문도메인전체.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e9a0abe5-76fc-4500-8fd2-985d49523448/주문도메인전체.png)

**역할(인터페이스)과 구현을 분리**해서 자유롭게 구현 객체를 조립할 수 있게 설계했다. 덕분에 회원 저장소는 물론이고, 할인 정책도 유연하게 변경할 수 있다

**주문 도메인 클래스 다이어 그램**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/805b5e3f-dea4-4f7c-9318-82f12d58187b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/805b5e3f-dea4-4f7c-9318-82f12d58187b/Untitled.png)

# Q&A

---

### Q : 저장소는 DIP를 이용해 손쉬운 교체를 하기 위함은 충분히 이해가 됐습니다. 하지만 현재 교체 가능성이 없는 Service클래스를 추상화하고 구현하는 이유가 있을까요? 어떤 기준으로 추상화하고 구현하시는지 궁금합니다.

A: [제가 객체 지향 설계와 스프링 마지막](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8/lecture/55331?tab=curriculum)에서 말씀드렸던 내용이 기억나실꺼에요.

- 이상적으로는 모든 설계에 인터페이스를 부여하자

**실무 고민**

- 하지만 인터페이스를 도입하면 추상화라는 비용이 발생한다.
- **기능을 확장할 가능성이 없다면, 구체 클래스를 직접 사용하고, 향후 꼭 필요할 때 리팩터링해서 인터페이스를 도입하는 것도 방법이다.**

강의에서는 이상적으로 역할과 구현을 분리한는 것에 초점을 맞추어서 이런 부분들도 분리했습니다. 저도 실무에서는 교체 가능성이 없는 서비스 클래스는 구체 클래스로 바로 만드는 것을 선호합니다^^

### Q: 스프링 예제를 볼 때 마다, Repository에 대해 구현은 여러개 인게 많은데 Service 에 대한 구현 객체는 꼭 Impl 이라는 이름으로 하나 더라구요..ㅜㅜ

현업에서 하나의 서비스 인터페이스에 대해 두 개 이상의  서비스 구현체를 만드는 경우가 있나요? 있다면 어떤 예시가 있을까요?

A: 실무에서도 대부분의 경우 서비스 인터페이스의 구현체는 하나입니다.

그런데, 전략패턴을 사용해서 실시간으로 다른 빈을 선택하는 경우에는 둘 이상 사용하는 경우도 있습니다.

관련해서는 조금 뒤에 의존관계 자동 주입 -> [조회한 빈이 모두 필요할 때, List, Map](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8/lecture/55380?tab=curriculum)에서 예시를 설명드립니다^^

추가로 다음 글도 읽어보시면 도움이 되실꺼에요^^ [https://www.inflearn.com/questions/69278](https://www.inflearn.com/questions/69278)

# 그외

---

1. **ConcurrentHashMap**

```jsx
private final Map<Long, Member> store = new ConcurrentHashMap<>();
```

- 실무에서는 동시성 문제를 해결하기 위해 ConcurrentHashMap를 사용해야 한다고 함.
- API URL : [https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentHashMap.html](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentHashMap.html)
- 참고 URL  : [https://devlog-wjdrbs96.tistory.com/269](https://devlog-wjdrbs96.tistory.com/269)