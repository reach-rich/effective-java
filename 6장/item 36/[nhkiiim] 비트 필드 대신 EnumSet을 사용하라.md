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

- 열거하는 값들이 주로 집합으로 사용되는 경우, 예전에느 정수 열거 패턴을 사용했다.
