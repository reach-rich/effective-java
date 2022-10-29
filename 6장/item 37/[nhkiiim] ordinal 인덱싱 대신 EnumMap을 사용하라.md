# ordinal 인덱싱 대신 EnumMap을 사용하라

### 1. ordinal 메서드로 인덱스 얻기 (No 권장)

- 정원에 심은 식물들을 하나의 배열로 관리하고 생애주기별로 묶기 예시

```java
public class Plant {
  enum LifeCycle { ANNUAL, PERNNIAL, BIENNIAL}

  final String name;
  final LifeCycle lifeCycle;

  public Plant(String name, LifeCycle lifeCycle) {
    this.name = name;
    this.lifeCycle = lifeCycle;
  }

  @Override
  public String toString() {
    return name;
  }
}
```

<br>

__ordinal()을 배열 인덱스로 사용__ `(절대 따라하지 말 것)`

```java
public static void usingOrdinalArray(List<Plant> garden) {
  Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[LifeCycle.values().length];
  
  for(int i = 0 ; i < plantsByLifeCycle.length ; i++) {
    plantsByLifeCycle[i] = new HashSet<>();
  }

  for(Plant plant : garden) {
    plantsByLifeCycle[plant.lifeCycle.ordinal()].add(plant);
  }

  for (int i = 0 ; i < plantsByLifeCycle.length ; i++) {
    System.out.printf("%s : %s%n", LifeCycle.values()[i], plantsByLifeCycle[i]);
  }
}
```

동작은 하지만 문제가 많다

(1) 배열은 제네릭과 호환되지 않아 비검사 형변환을 수행해야 함
`Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[LifeCycle.values().length];`

(2) 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야함
`System.out.printf("%s : %s%n", LifeCycle.values()[i], plantsByLifeCycle[i]);`

(3) __정확한 정수값을 사용한다는 것을 직접 보증해야한다__
