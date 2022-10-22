# 비트 필드 대신 EnumSet을 사용하라

### 1. 구닥다리 비트 필드

```java
public class Text {
   public static final int STYLE_BOLD      = 1 << 0; // 1
   public static final int STYLE_ITALIC    = 1 << 1; // 2
   public static final int STYLE_UNDERLINE = 1 << 2; // 4
   public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8
   
   // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
   public void applyStyles(int styles) { ... }
}
```

- 열거하는 값들이 주로 집합으로 사용되는 경우, 예전에느 정수 열거 패턴을 사용했다
- 비트별 OR를 사용해 여러 상수를 하나의 집합으로 모을 수 있다
- 이러한 집합을 비트 필드라고 한다

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```
- 비트 필드를 사용하면 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있다

<br>

### 2. 비트 필드의 단점
- 정수 열거 상수의 단점을 그대로 지닌다
- 비트 필드 값이 그대로 출력되면 단순한 상수 출력보다 해석이 어렵다
- 비트 필드 하나에 있는 모든 원소를 순회하기도 까다롭다
- 비트 수가 늘어나면 자료형을 바꿔야하기 때문에 (int - long)  API 작성 시 미리 예측해 수정을 방지해야한다

### 3. EnumSet 클래스
- EnumSet 클래스는 열거 타입 상수의 값으로 구현된 집합을 효과적으로 표현
- Set 인터페이스 완벽 구현
- 타입 안전
- 어떤 Set 구현체와도 같이 사용 가능
- 
