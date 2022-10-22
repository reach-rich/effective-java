---
layout: post
title: Item 36. 비트 필드 대신 EnumSet을 사용하라
date: 2022-10-22 16:27 +0900
categories: [Book, Effective Java]
tags: [Java, Effective Java, Enum]
---



Effective Java의 서른여섯 번째 아이템 "비트 필드 대신 EnumSet을 사용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 비트 필드 열거 상수

열거한 값들이 주로 집합으로 사용될 경우, 예전에는 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해왔다.

```java
public class Text {
  public static final int STYLE_BOLD = 1 << 0;
  public static final int STYLE_ITALIC = 1 << 1;
  public static final int STYLE_UNDERLINE = 1 << 2;
  public static final int STYLE_STRIKETHROUGH = 1 << 3;
  
  // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
  public void applyStyles(int styles) { ... }
}
```

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

다음과 같은 식으로 비트별 OR를 사용해 여러 상수를 하나의 집합으로 모을 수 있으며, 이렇게 만들어진 집합을 비트 필드라 한다.

* 비트별 연산을 사용해 합집합과 교집합 같은 집합을 효율적으로 수행할 수 있다.
* 정수 열거 상수의 단점을 그대로 지닌다.
* 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기가 훨씬 어렵다.
* 비트 필드 하나에 녹아 있는 모든 원소를 순회하기에 까다롭다.
* 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입(보통은 int나 long)을 선택해야 한다.

<br>

## 2. EnumSet

정수 상수보다 열거 타입을 선호하는 프로그래머 중에도 상수 집합을 주고 받아야 할 때는 여전히 비트 필드를 사용하기도 한다. 하지만 이제 더 나은 대안이 있다. java.util 패키지의 EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.

* Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 Set 구현체와도 함께 사용할 수 있다.
* EnumSet의 내부는 비트 벡터로 구현되었다. 원소가 64개 이하라면, 즉 대부분의 경우에 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.
* removeAll과 retainAll 같은 대량 작업은 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현했다. 그러면서도 비트를 직접 다룰 때 흔히 겪는 오류들에서 해방된다.

다음은 앞의 예를 EnumSet을 활용해 수정한 코드다. 보다시피 짧고 깔끔하고 안전하다. 

```java
public class Text {
  public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH };
  
  // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
  public void applyStyles(Set<Style> styles) { ... }
}
```

```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

<br>

## 3. 핵심 정리

* 열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다.
* EnumSet 클래스는 비트 필드 수준의 명료함과 성능을 제공하고 열거 타입의 장점까지 선사한다.
* EnumSet의 유일한 단점이라면 불변 EnumSet을 만들 수 없다는 것이다.

<br>

## 4. Related Posts

* 정수 열거 패턴 (Item 34)
