## 비지니스 요구사항과 설계

---

### ⭐ 요구사항

- 회원
    - 회원 가입, 조회 기능이 있다.
    - 회원 등급은 VIP, 일반 등급이 있다.
    - 회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다 **( 미확정 )**

- 주문과 할인 정책
    - 상품 주문 기능이 있다.
    - 회원 등급에 따라 할인 정책을 적용할 수 있다.
    - VIP 등급은 1000원을 할인해주는 고정 금액 할인을 적용. **( 추후에 변경 될 수 있다. )**
    - 할인 정책은 변경 가능성이 높다. 회사의 기본 할인 정책을 정하지 못했고, 오픈 직전까지 고민을 미루고 싶다. 최악의 경우 할인을 적용하지 않을 수 있다.

### ⭐ 설계

**회원 도메인 협력 관계**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/33f55985-5303-4c8d-ab58-48833f02764f/도메인.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/33f55985-5303-4c8d-ab58-48833f02764f/도메인.png)

**회원 클래스 다이어그램**

 클래스 다이어그램은 모델링 된 시스템 구조의 전체 또는 부분을 보여주기 위한 정적 구조 다이어그램 유형이다. 클래스 다이어그램에서 어떤 구현체가 동작할 지는 알 수 없다

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/366e595f-5620-465e-9f47-8b8fe0f54b1d/회원클래스.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/366e595f-5620-465e-9f47-8b8fe0f54b1d/회원클래스.png)

**회원 객체 다이어그램**

객체 다이어그램은 특정 순간에 객체와 객체 간의 관계를 나타낸다. 클라이언트가 사용하는 구현체를 알 수 있다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e9316ca2-2496-434e-8da3-024506f69e8b/객체_다이어그램.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e9316ca2-2496-434e-8da3-024506f69e8b/객체_다이어그램.png)

## 주요 코드

---

### 🙋Q&A

---

### Q : 저장소는 DIP를 이용해 손쉬운 교체를 하기 위함은 충분히 이해가 됐습니다. 하지만 현재 교체 가능성이 없는 Service클래스를 추상화하고 구현하는 이유가 있을까요? 어떤 기준으로 추상화하고 구현하시는지 궁금합니다.

A: [제가 객체 지향 설계와 스프링 마지막](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8/lecture/55331?tab=curriculum)에서 말씀드렸던 내용이 기억나실꺼에요.

- 이상적으로는 모든 설계에 인터페이스를 부여하자

**실무 고민**

- 하지만 인터페이스를 도입하면 추상화라는 비용이 발생한다.