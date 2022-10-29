## 1. 들어가기

Enum을 사용한 배열이나 리스트에서 원소를 꺼낼 때, ordinal 메서드를 사용하는 경우가 있습니다.

예시를 보겠습니다.

```java
  class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
      this.name = name;
      this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
      return name;
    }
  }
```

```java
  Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

  for(int i=0; i<plantsByLifeCycle.length; i++)
    plantsByLifeCycle[i] = new HashSet<>();

  for(Plant p : garden)
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);

  for (int i=0; i<plantsByLifeCycle.length; i++)
    System.out.printf("%s : %s%n", LifeCycle.values()[i], plantsByLifeCycle[i]);
```

정원에 심은 식물들을 생애주기 별로 관리하는 코드입니다.

위의 코드는 동작은 하지만 다음의 문제가 있습니다.

## 2. ordinal 메서드를 인덱스로 사용한 코드의 문제점

1. 비검사 형변환을 수행해야한다.

   🔹배열은 제네릭과 호환되지 않기 때문에 비검사 형변환을 수행해야 합니다.

   🔹`Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];`

   <br>

2. 출력 결과에 직접 레이블을 달아야 한다.

   🔹`System.out.printf("%s : %s%n", LifeCycle.values()[i], plantsByLifeCycle[i]);`

   <br>

3. 잘못된 값을 사용할 위험이 있다.

   🔹잘못된 동작을 수행하거나, ArrayIndexOutOfBoundsException을 발생시킬 수 있습니다.

## 3. 해결법

위의 문제를 해결할 수 있는 좋은 방법이 있습니다.

예시를 천천히 살펴보면 실질적으로 배열은 열거 타입 상수를 값으로 매핑하는 역할을 합니다.

그렇기 때문에 Map을 사용할 수 있고 특히, 열거 타입을 키로 사용하는 EnumMap을 사용할 수 있습니다.

위의 예시를 EnumMap으로 대체하면 다음과 같습니다.

```java
  Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
                  
  for (Plant.LifeCycle lc : Plant.LifeCycle.values())
    plantsByLifeCycle.put(lc, new HashSet<>());

  for (Plant p : garden)
    plantsByLifeCycle.get(p.lifeCycle).add(p);

  System.out.println(plantsByLifeCycle);
```

EnumMap을 사용하면 다음의 장점이 있습니다.

1. EnumMap 내부적으로 배열을 사용하기 때문에 배열을 사용한 코드와 성능이 비슷하다.

2. EnumMap의 키 타입은 한정적 타입 토큰이므로 안전하지 않는 형변환을 사용할 필요가 없다.

3. 열거 타입을 키로 사용하기 때문에 출력 결과에 직접 레이블을 달 필요가 없다.

4. 배열 인덱스를 계산하는 과정에서 오류가 날 가능성이 없다.

## 4. Stream을 이용한 EnumMap

Java 8에서 추가된 Stream을 이용하면 코드를 더 줄일 수 있습니다.

```java
  Arrays.stream(garden)
        .collect(groupingBy(p -> p.lifeCycle, () -> new EnumMap<>(LifeCycle.class), toSet()));
```

이는 최적화를 이뤄낼 수 있다는 장점도 있습니다.

Stream을 이용하지 않는 예시는 무조건 식물의 생애주기 당 하나씩의 중첩 Map을 만들지만,

Stream을 이용하면 해당 생애주기에 속하는 식물이 있을 때만 중첩 Map을 만듭니다.

## 5. Stream을 이용한 EnumMap 심화

이번에는 ordinal을 두 번 사용한 예시를 살펴보겠습니다.

```java
  public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
      MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

      private static final Transition[][] TRANSITIONS = {
        {null, MELT, SUBLIME},
        {FREEZE, null, BOIL},
        {DEPOSIT, CONDENSE, null}
      };

      public static Transition from(Phase from, Phase to) {
        return TRANSITIONS[from.ordinal()][to.ordinal()];
      }
    }
}
```

위의 예시는 두 가지 상태(Phase)를 전이(transition)와 매핑하도록 구현한 코드입니다.

예를 들면, 액체(LIQUID)에서 고체(SOLID)로의 전이는 응고(FREEZE)입니다.

이 예시도 EnumMap으로 변환할 수 있는데 다음과 같습니다.

```java
  public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
      MELT(SOLID, LIQUID),
      FREEZE(LIQUID, SOLID),
      BOIL(LIQUID, GAS),
      CONDENSE(GAS, LIQUID),
      SUBLIME(SOLID, GAS),
      DEPOSIT(GAS, SOLID);

      private final Phase from;
      private final Phase to;

      Transition(Phase from, Phase to) {
        this.from = from;
        this.to = to;
      }

      private static final Map<Phase, Map<Phase, Transition>> transitionMap = 
        Stream.of(values())
              .collect(Collectors.groupingBy(
                t -> t.from,                          // 바깥 Map의 Key
                () -> new EnumMap<>(Phase.class),     // 바깥 Map의 구현체
                Collectors.toMap(
                  t -> t.to,                          // 바깥 Map의 Value(Map으로), 안쪽 Map의 Key
                  t -> t,                             // 안쪽 Map의 Value
                  (x, y) -> y,                        // 충돌 처리 방법
                  () -> new EnumMap<>(Phase.class)    // 안쪽 Map의 구현체
                )
              ));

      public static Transition from(Phase from, Phase to) {
          return transitionMap.get(from).get(to);
      }
    }
  }
```

여기서 만약, `PLASMA`를 추가한다면 EnumMap 예시는 Phase 1개, Transition 2개만 추가해주면 되지만,

배열 예시에서는 Phase 1개, Transition 2개와 더불어 TRANSITIONS 상태표도 함께 수정해줘야 하기에

유지 보수 측면에서도 EnumMap을 사용하는 것이 좋습니다.

## 6. 정리

배열의 인덱스를 얻기 위해 ordinal 메서드를 사용하는 것은 일반적으로 좋지 않기 때문에

더 짧고 명료하고 안전하며 유지보수 측면에서도 좋은 EnumMap을 사용합시다.