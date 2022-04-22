## 1. 들어가기

의존 객체 주입(Dependency Injection) 패턴에 대해 들어본 적 있나요?

저는 느낌 상으로는 알겠는데 자세히는 잘 모르겠네요...

그럼 먼저, 예시를 통해 어떤 이유로 생성되었는지 배경부터 차근차근 알아봅시다.

```java
  // 할인 정책 인터페이스
  public interface DiscountPolicy {}

  // 정액 할인, 정률 할인 구체 클래스
  public class FixDiscountPolicy implements DiscountPolicy() {}
  public class RateDiscountPolicy implements DiscountPolicy() {}

  // 할인 클래스
  public class Discount {
    private static final DiscountPolicy discountPolicy = new FixDiscountPolicy();
  }
```

E-Commerce 할인 정책(DiscountPolicy)은 보통

정해진 금액만큼 할인해주는 정액 할인(FixDiscountPolicy)과

일정 비율만큼 할인해주는 정률 할인(RateDiscountPolicy)로 나뉩니다.

만약, 할인 정책을 정액 할인에서 정률 할인으로 변경하고 싶을 때,

할인 클래스는 인터페이스와 구체 클래스를 모두 의존하고 있기 때문에

할인 클래스를 변경해줘야하고 이는 5가지 객체지향 설계 원칙 중 OCP 원칙을 위배하게 됩니다.

> * OCP(Open-Closed Principle)
>
> 소프트웨어 개체는 확장에는 열려있고, 변경에는 닫혀 있어야 한다.

그럼 해당 코드를 어떻게 수정해야 OCP 원칙을 지킬 수 있을까요?

이럴 때, 사용하는 패턴이 '의존 객체 주입'입니다.

## 2. 의존 객체 주입(Dependency Injection)

의존 객체 주입이 무엇일까요?

이번에도 예시를 통해 알아봅시다.

```java
  // 할인 정책 인터페이스
  public interface DiscountPolicy {}

  // 정액 할인, 정률 할인 구체 클래스
  public class FixDiscountPolicy implements DiscountPolicy() {}
  public class RateDiscountPolicy implements DiscountPolicy() {}

  // 할인 클래스
  public class Discount {
    private static final DiscountPolicy discountPolicy;

    public Discount(DiscountPolicy discountPolicy) {
      this.discountPolicy = Objects.requireNonNull(discountPolicy);
    }
  }
```

첫 번째 예시의 할인 클래스만 수정을 해보았습니다.

기존 코드와 비교해서 살펴보면 필드에 값을 바로 넣어주는 것이 아닌

생성자를 통해 필드에 값을 넣어주는 것을 볼 수 있습니다.

기존 코드를 해당 코드로 수정하면서 얻는 장점은

할인 정책 클래스(정액 할인, 정률 할인) 의존성을 제거하고,

할인 정책 인터페이스만 의존하고 있기 때문에

만약, 정액 할인을 정률 할인으로 변경했을 경우, 할인 클래스 자체를 수정하지 않아도 됩니다.

즉, 확장에는 열려있고, 변경에는 닫혀있는 객체 지향 설계 원칙의 OCP를 지키게 됩니다.

OCP를 지키기 위해 필요한 이 패턴은

외부에서 객체를 주입하는 것처럼 보인다고 해서 '의존 객체 주입' 이라고 합니다.

## 3. 변형

의존 객체 주입의 변형으로 생성자에 자원 팩터리를 넘겨주는 방식이 있습니다.

팩터리란, 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체로

대표적인 예는 자바 8의 Supplier<T> 인터페이스가 있습니다.

자원 팩터리인 Supplier<T>를 입력으로 받는 메서드로 만드는데

일반적으로 한정적 와일드카드 타입을 사용해

팩터리의 타입 매개변수를 제한하는 방법으로 사용됩니다.

이 방식을 사용해 클라이언트는 자신이 명시한 타입의 하위 타입이라면

무엇이든 생성할 수 있는 팩터리를 넘길 수 있습니다.

```java
  Discount create(Supplier<? extends DiscountPolicy> discountFactory) { ... }
```

## 4. 정리

이번 포스트에서는 의존 객체 주입에 대해서 알아보았습니다.

그리고 예시를 통해 의존 객체 주입을 사용하면 클래스의 의존성을 줄임으로써

좋은 객체 지향 설계를 할 수 있음을 확인했습니다.

유연성, 재사용성, 테스트 용이성을 개선해주는 의존 객체 주입을 자주 활용해봅시다.