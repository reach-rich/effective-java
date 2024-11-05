## 1. 들어가기

Java는 C, C++ 언어와 다르게 안전한 언어입니다.

Java로 작성한 클래스는 시스템의 다른 부분에서 무슨 짓을 하든 그 불변식을 지켜지기 때문인데요.

하지만 아무리 Java라고해도 다른 클래스로부터의 침범을 아무런 노력 없이 다 막을 수 있는 것은 아닙니다.

그럼 어떤 경우에 취약한 부분이 생길까요?

## 2. 불변식을 지키지 못한 예시

다음 코드는 기간을 표현하는 클래스입니다.

```java
/* 불변식이 깨질 수 있는 Period 클래스 */
   class Period {
      private final Date start;
      private final Date end;

      public Period(Date start, Date end) {
         if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦습니다.");
         }

         this.start = start;
         this.end = end;
      }

      public Date start() { return start; }
      public Date end() { return end; }

      // ... 코드 생략
}
```

얼핏 보기엔 불변처럼 보이고, 시작 시간이 종료 시각보다 늦을 수 없다는 불변식이 지켜질 것으로 보입니다.

하지만 Date는 가변이므로 어렵지 않게 그 불변식을 깨뜨릴 수 있습니다.

```java
/* Client */
   Date start = new Date();
   Date end = new Date();
   Period p = new Period(start, end);
   end.setYear(78);

   System.out.println(p.end);
```

```
   Sat Mar 25 16:26:18 KST 1978
```

다행히 Java 8 이후로는 간단한 해결 방법이 있습니다.

Date 대신 불변인 Instant(또는 LocalDateTime, ZonedDateTime)를 사용하면 됩니다.

그럼 Java 8 이전은 불변식으로 만드는 방법이 없는걸까요?

## 3. 불변식으로 만드는 방법

외부 공격으로부터 Period 인스턴스 내부를 보호하기 위해서는 매개변수를 방어적으로 복사해야 합니다.

그 후, Period 인스턴스 내부에서는 원본이 아닌 복사본을 사용합니다.

Period 인스턴스를 보호하기 위해서는 두 가지 방법이 있습니다.

1. 가변 매개변수 방어적 복사

   ```java
   /* 불변식을 지키기 위한 Period 클래스 생성자 수정 */
      public Period(Date start, Date end) {
         this.start = new Date(start.getTime());
         this.end = new Date(end.getTime());

         if (this.start.compareTo(this.end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦습니다.");
         }
      }
   ```

   순서가 부자연스러워 보이겠지만 멀티스레딩 환경을 위해 반드시 이렇게 작성해야 합니다.

   복사본을 만드는 찰나의 취약 순간에 다른 스레드가 원본 객체를 수정할 위험이 있기 때문입니다.

   추가로 방어적 복사에 Date의 clone 메서드를 사용하지 않은 점에도 주목해야 합니다.

   그 이유는 Date가 가변이므로 Date의 clone이 아닐 수 있기 때문입니다.

   즉, clone이 악의를 가진 하위 클래스 인스턴스를 반환할 수도 있다는 의미입니다.

   그렇기 때문에 매개변수가 제 3자에 의해 확장될 수 있다면 clone을 사용하지 않는 것이 좋습니다.

   <br>

2. 가변 필드 방어적 복사

   ```java
   /* 불변식을 지키기 위한 Period 클래스 접근자 메서드 수정 */
      public Date start() {
         return new Date(start.getTime());
      }

      public Date end() {
         return new Date(end.getTime());
      }
   ```

   생성자와는 달리 접근자 메서드에서는 방어적 복사를 위해 clone 메서드를 사용할 수 있습니다.

   Period가 가지고 있는 Date 객체는 java.util.Date임이 확실하기 때문입니다.

## 4. 방어적 복사의 다른 목적

매개변수를 방어적으로 복사하는 목적이 불변 객체를 만들기 위해서만은 아닙니다.

클라이언트가 제공한 객체 참조를 내부 자료구조에 보관할 때면 항상 잠재적으로 변경될 수 있는지 확인하고

변경될 수 있는 객체라면 임의로 변경 되어도 클래스가 문제없이 동작할지를 따져보아야 합니다.

만약, 확신할 수 없다면 복사본을 만들어 저장해야 합니다.

가변 내부 객체를 클라이언트에 건네줄 때도 안심할 수 없다면 마찬가지로 방어적 복사를 수행해야 합니다.

## 5. 방어적 복사의 예외

방어적 복사는 성능 저하가 따르고, 또 항상 쓸 수 있는 것은 아닙니다.

그래서 두 가지의 경우 방어적 복사를 생략할 수 있습니다.

1. 클라이언트가 컴포넌트 내부를 수정하지 않으리라 확신(신뢰)하는 경우

   생략을 하더라도 클라이언트가 매개변수나 반환값을 수정하지 말아야 함을
   
   명확히 문서화하는 게 좋습니다.

   <br>

2. 불변식이 깨지더라도 그 영향이 오직 클라이언트로 국한되는 경우

   한 예시로 래퍼 클래스를 들 수 있습니다.

   래퍼 클래스의 특성상 클라이언트는 래퍼에 넘긴 객체에 직접 접근할 수 있지만,

   그 영향을 오직 클라이언트 자신만 받게 됩니다.

## 6. 정리

   이번 포스트는 악의적인 클라이언트의 공격을 막기 위해 방어적 복사하는 법을 알아보았습니다.

   클라이언트로부터 받거나 클라이언트로 반환하는 구성 요소가 가변이라면 반드시 방어적 복사를 해야합니다.

   하지만 복사 비용이 크거나 클라이언트를 신뢰한다면 방어적 복사를 생략할 수 있습니다.

   이런 경우에는 반드시 문서도 함께 남기도록 합시다.