## 1. 들어가기

모든 열거 타입은 자연스럽게 하나의 정숫값에 대응되고,

그 열거 타입에서 몇 번째 위치인지 반환하는 ordinal 메서드를 제공합니다.

그렇기 때문에 개발자는 조금 더 편하게 개발하기 위해 ordinal 메서드를 사용할 수도 있습니다.

하지만, ordinal 메서드를 사용하면 몇 가지 문제점이 있는데 한번 알아보겠습니다.

## 2. ordinal 메서드의 문제점

다음은 합주단 종류를 연주자가 1명인 솔로부터 10명인 디텍트까지 정의한 열거 타입 예시입니다.

```java
  public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTET

    public int numberOfMusicians() {
      return ordinal() + 1;
    }
  }
```

제대로 동작은 하지만, 유지보수 측면에서 다음의 문제점이 있습니다.

1. 상수 선언 순서를 바꾸는 순간 오작동한다.

   만약, 실수로 SOLO와 DUET의 순서를 바꾸면 SOLO는 2, DUET은 1의 연주자 수를 반환하게 됩니다.

   <br>

2. 이미 사용 중인 정수와 같은 값을 가진 상수는 추가할 수 없다.

   8중주(octet) 상수가 있으니, 똑같이 8명이 연주하는 복4중주 상수는 추가할 수 없습니다.

   <br>

3. 중간에 값을 비워둘 수 없다.

   12명이 연주하는 3중 4중주 상수를 추가하고 싶다면,
   
   11명으로 구성된 더미 상수를 추가해야 사용할 수 있습니다.

그럼 이런 문제점을 어떻게 해결할 수 있을까요?

## 3. ordinal 메서드의 해결

해결법은 매우 간단합니다.

열거 타입 상수에 연결된 값을 ordinal 메서드가 아닌 인스턴스 필드로 가져오는 것입니다.

```java
  public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);
    
    private final int numberOfMusicians;

    Ensemble(int numberOfMusicians) {
      this.numberOfMusicians = numberOfMusicians;
    }

    public int numberOfMusicians() { return this.numberOfMusicians; }
  }
```

## 4. 정리

이번 포스트는 열거 타입에서 제공하는 ordinal 메서드의 문제점에 대해 알아보았습니다.

Enum API 문서를 보면 ordinal 메서드에 대해 다음과 같이 명시해 놓았습니다.

> It is designed for use by sophisticated enum-based data structures, such as EnumSet and EnumMap.

따라서, 이런 용도가 아니라면 ordinal 메서드는 절대 사용하지 맙시다.