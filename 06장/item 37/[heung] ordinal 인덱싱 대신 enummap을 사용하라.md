Effective Java의 서른일곱 번째 아이템 "ordinal 인덱싱 대신 EnumMap을 사용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 예제

이따금 배열이나 리스트에서 원소를 꺼낼 때 oridinal 메서드로 인덱스를 얻는 코드가 있다. 다음은 식물을 간단한 나타낸 예다.

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

정원에 심은 식물들을 배열 하나로 관리하고, 이들을 생애주기(한해살이, 여러해살이, 두해살이)별로 묶어보자. 

<br>

### 1) ordinal

어떤 프로그래머는 집합들을 배열 하나에 넣고 생애주기의 ordinal 값을 그 배열의 인덱스로 사용하려 할 것이다.

```java
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for (int i=0; i<plantsByLifeCycle.length; i++) {
  plantsByLifeCycle[i] = new HashSet<>();
}

for (Plant p : garden) {
  plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
}

for (int i=0; i<plantsByLifeCycle.length; i++) {
  System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

동작은 하지만 문제가 많다. 

* 배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야 하고 깔끔히 컴파일되지 않는다.
* 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다.
* 정수는 타입 안전하지 않기 때문에 정확한 정숫값을 사용한다는 것을 직접 보증해야 한다.

<br>

### 2) EnumMap

위에서 배열은 실질적으로 열거 타입 상수를 값으로 매핑하는 일을 한다. 그러니 Map을 사용할 수도 있을 것이다. 사실 열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현체가 존재하는데, 바로 EnumMap이다.

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
  plantsByLifeCycle.put(lc, new HashSet<>());
}
for (Plant p : garden) {
  plantsByLifeCycle.get(p.lifeCycle).add(p);
}
System.out.println(plantsByLifeCycle);
```

* 더 짧고 명료하고 안전하고 성능도 원래 버전과 비등하다.
* 안전하지 않은 형변은 쓰지않는다.
* 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력 결과에 직접 레이블을 달 필요가 없다.
* 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 원천봉쇄된다.

여기서 EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로, 런타임 제네릭 타입 정보를 제공한다.

<br>

### 3) Stream 활용

 스트림을 이용해 맵을 관리하면 코드를 더 줄일 수 있다. 다음은 위 예의 동작을 거의 모방한 단순한 스트림 기반 코드다.

```java
System.out.println(Arrays.stream(garden)
                   .collect(groupingBy(p -> p.lifeCycle)));
```

이 코드는 EnumMap이 아닌 고유한 맵 구현체를 사용했기 때문에 EnumMap을 써서 얻은 공간과 성능 이점이 사라진다는 문제가 있다. 이 문제는 매개변수 3개짜리 Collectors.groupingBy 메서드를 사용하여 해결할 수 있다. 이 메서드는 mapFactory 매개변수에 원하는 맵 구현체를 명시해 호출할 수 있다.  

```java
System.out.println(Arrays.stream(garden)
                   .collect(groupingBy(p -> p.lifeCycle, 
                                       () -> new EnumMap<>(LifeCycle.class), toSet())));
```

단순한 프로그램에서는 최적화가 굳이 필요 없지만, 맵을 빈번히 사용하는 프로그램에서는 꼭 필요할 것이다.

<br>

Stream을 사용하면 EnumMap만 사용했을 때와는 살짝 다르게 동작한다. EnumMap 버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, Stream 버전에서는 해당 생애주기에 속하는 식물이 있을 때만 만든다.

<br>

## 2. 핵심 정리

* 배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니, 대신 EnumMap을 사용하라.
* 다차원 관계는 EnumMap<..., EnumMap<...>>으로 표현하라.

<br>

## 3. Related Posts

* ordinal 메서드 (Item 35)
* 한정적 타입 토큰 (Item 33)
* 제네릭 (Item 28)
* Stream (Item 45)
