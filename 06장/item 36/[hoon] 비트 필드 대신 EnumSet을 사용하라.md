## 1. 들어가기

열거한 값들을 집합으로 사용될 경우, 기존에는 정수 열거 패턴을 이용해왔습니다.

```java
  public class Text {
    public static final int STYLE_BOLD            = 1 << 0; // 1
    public static final int STYLE_ITALIC          = 1 << 1; // 2
    public static final int STYLE_UNDERLINE       = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH   = 1 << 3; // 8

    public void applyStyles(int styles) {...}
  }
```

```java
  text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

이렇게 비트별 OR를 사용해 여러 상수를 하나의 집합으로 만든 것을 비트 필드라고 합니다.

하지만, 비트 필드는 정수 열거 상수의 단점을 그대로 지니며, 다음과 같은 문제를 가집니다.

## 2. 비트 필드의 문제점

1. 비트 필드 값이 그대로 출력되는 경우, 단순 정수 열거 상수보다 해석하기 어렵다.

   🔹위의 예시에서 특정 값이 반환됐을 경우, 해당 값만 보고는 무엇을 뜻하는지 해석하기 어렵습니다.

   <br>

2. 비트 필드 하나에 녹아 있는 모든 원소를 순회하기 까다롭다.

   🔹비트 필드 하나의 값을 알기 위해서는 모든 원소를 순회해야 합니다.

   <br>

3. 최대 몇 비트가 필요한지를 API작성 시 미리 예측해 적절한 타입을 선택해야 한다.

   🔹API를 수정하지 않고는 비트 수를 더 늘릴 수 없기 때문입니다.

## 3. 비트 필드의 해결법

다행히 비트 필드보다 더 나은 해결법이 있습니다.

바로, java.util 패키지의 EnumSet 클래스를 사용하는 것인데요.

EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해줍니다.

EnumSet 클래스로 위의 비트 필드 예시를 대체하면 다음과 같습니다.

```java
  public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    // 어떤 Set을 넘겨도 좋으나 EnumSet 추천
    public void applyStyles(Set<Style> styles) { ... }
  }
```

```java
  text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

## 4. 정리

이번 포스트는 열거한 값들을 집합으로 사용될 경우, 기존에 사용했던 비트 필드와

비트 필드의 문제점을 해결할 수 있는 EnumSet에 대해서 알아보았습니다.

EnumSet 클래스의 유일한 단점은 불변 EnumSet을 만들 수 없다는 점이지만,

비트 필드 수준의 명료함과 성능, 열거 타입의 장점을 제공하기 때문에 EnumSet 클래스를 사용해봅시다.