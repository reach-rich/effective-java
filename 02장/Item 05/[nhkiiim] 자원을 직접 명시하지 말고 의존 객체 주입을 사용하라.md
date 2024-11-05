# 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

### 1. 객체 하나를 가지는 클래스

- 예를 들어 맞춤법 검사기는 사전에 의존

<br>

(1) 이런 클래스를 정적 유틸리티 클래스(아이템 4)로 구현하기도 함
```java
public class SpellChecker {
  private static final Ganada dictionary = ...;
  
  private SpellChecker() {} // 객체 생성 방지
  ...
}
```

<br>

(2) 싱글턴으로 구현하기도 함 (아이템 3)
```java
public class SpellChecker {
  private final Ganada dictionary = ...;
  
  private SpellChecker(...) {}
  public static SpellChecker INSTANCE = new SpellChecker(...);
  
  ...
}
```

<br>

- 두 방식 모두 사전을 단 하나만 사용하게 됨 (안훌륭함 ㅡ3ㅡ)
  
#

### 2. 의존 객체 주입 사용

- 실전에서는 가나다 사전, 어머나 사전 등등의 많은 사전들을 참고 할 것임
 - 여러 사전을 사용할 수 있도록 바꿔보자!
  
(1) 필드에서 final 제거하고 사전 교체 메서드 추가하기
- 어색한 방법
- 오류를 내기 쉬움
- 멀티스레드 환경에서는 쓸 수 없음
  
- 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않음

<br>
 
(2) 인스턴스 생성 시 생성자에 필요한 자원을 넘겨주는 방식 사용 `(정답)`
- 클래스가 여러 자원 인스턴스를 지원하고 클라이언트가 원하는 자원을 사용함
- 의존 객체 주입의 한 형태

```java
public class SpellChecker{
 private final Ganada dictionary;
 
 public SpellChecker(Ganada dictionary) {
  this.dictionary = Object.requireNonNull(dictionary);
 }
 
 ...
}
```

- 단순해서 많은 프로그래머가 사용하는 방식 (이름도 모름서)
- 자원이 몇개든 의존관계가 뭐든 잘 작동함
- 불변을 보장해 여러 클라이언트가 의존 객체들을 안심하고 사용 가능
- 생성자, 정적팩터리, 빌더 모두에 똑같이 적용 가능


<br>

- 이 패턴의 쓸만한 변형 : 생성자에 자원 팩터리 넘겨주기
- 팩터리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체
- 즉 팩터리 메서드 패턴을 구현한 것
- Supplier<T>가 완벽한 예시
- Supplier<T>를 입력으로 받는 메서드는 일반적으로 한정적 와일드카드 타입을 사용해 팩터리 타입 매개변수를 제한해야 함
- 이 방식으로 클라이언트는 자신이 명시한 타입의 하위 타입이라면 무었이든 생성할 수 있는 팩터리를 넘길 수 있음

  <br>
  
- 클라이언트가 제공한 팩터라가 생성한 타일들로 구성된 모자이크 만드는 메서드

  ```java Mosaic create(Supplier<? extends Tile> tileFactory) {...}```

#
  
### 3. 주목
- 의존 객체 주입이 유연성과 테스트 용이성은 개선해줌
- 의존성이 왕많은 큰 프로젝트에서는 코드를 어지럽게 만들기도 함

- Dagger, Guice, Spring과 같은 의존 객체 주입 프레임워크를 사용하면 이런 어질어질 해결 가능

