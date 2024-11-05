# ordinal 인덱싱 대신 EnumMap을 사용하라

### 1. ordinal 메서드로 인덱스 얻기 (No 권장)

정원에 심은 식물들을 하나의 배열로 관리하고 생애주기별로 묶기 예시

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

<br>

동작은 하지만 문제가 많다

<br>

(1) 배열은 제네릭과 호환되지 않아 비검사 형변환을 수행해야 함

`Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[LifeCycle.values().length];`

(2) 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야함

`System.out.printf("%s : %s%n", LifeCycle.values()[i], plantsByLifeCycle[i]);`

(3) __정확한 정수값을 사용한다는 것을 직접 보증해야한다__

정수는 열거 타입과 달리 타입 안전하지 않음 - 잘못된 값을 사용할 수도 있기 때문 (ArrayIndexOutOfBoundException 운이 좋으면 발생)

#
### 2. EnumMap으로 대체하기

위의 문제를 해결할 수 있는 아주 좋은 해결책

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
                  
for (Plant.LifeCycle lc : Plant.LifeCycle.values()){
  plantsByLifeCycle.put(lc, new HashSet<>());
}

for (Plant p : garden){
  plantsByLifeCycle.get(p.lifeCycle).add(p);
}

System.out.println(plantsByLifeCycle);
```

- 더 짧고 명료, 성능도 ordinal() 메서드 사용과 비등
- 안전하지 않은 형변환은 빠지고, 맵의 키인 열거타입이 그 자체로 출력용 문자열도 제공
- 배열 인덱스 계산 과정에서 오류 가능성도 원청봉쇄 (라이프사이클 자체를 키로 쓰기 때문)


#
### 3. 스트림 사용으로 코드 더 줄이기

```java
System.out.println(Arrays.stream(garden)
                      .collect(groupingBy(
                                            p -> p.lifeCycle, 
                                            () -> new EnumMap<>(LifeCycle.class), 
                                            toSet()
                                          )
                              )    
                   );
}
```
- collect()는 원하는 자료형으로 변환해주는 Stream 메서드
- groupingBy()는 Map 형태로 return
- 결과적으로 .. 점 헷갈림


// 액체 고체 예시는 같은 내용 같으므로 정리 생략

#
### 4. 결론

__ordinal 메서드 쓰지 마라!__




