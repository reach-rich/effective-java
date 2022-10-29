### 🤔 ordinal 인덱싱

**✏ ex) 식물을 간단히 나타낸 클래스**

```java
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }
    
    final String name;
    final LifeCycle lifeCycle;
    
    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }
    
    @Override public String toString() {
        return name;
    }
}
```

>배열이나 리스트에서 원소를 꺼낼 때 `ordinal`메서드로 인덱스를 얻는 코드

<br>

**✏ #01 예제소스 | 비트 필드 열거 상수 - 구닥다리 기법**

```java
Set<Plant>[] plantsByCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

for (int i = 0; i < plantsByLifeCycle.length; i++) 
    plantByLifeCycle[i] = new HashSet<>();

for (Plant p : garden)
    palntsByLifeCycle[p.lifeCycle.ordinal()].add(p);

// 결과 출력
for (int i = 0; i < plantsByLifeCycle.length; i++) {
    System.out.println("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

>배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야 하고 깔끔히 컴파일되지 않음
>
>배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 함
>
>정수는 열거 타입과 다릴 타입 안전하지 않기 때문에 정확한 정숫값을 사용한다는 것을 직접 보증해야 함
>
>잘못된 값을 사용하면 잘못된 동작을 묵묵히 수행하거나 `ArrayIndexOutOfBoundsException` 발생

<br>

---

<br>

### 💡 EnumMap

**: 열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현체**

배열은 실질적으로 열거 타입 상수를 값으로 매핑하는 일을 하므로 `Map`을 사용할 수도 있음

<br>

**✏ #02 예제소스 | EnumMap을 사용해 데이터와 열거 타입을 매핑한다**

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.valuese())
    plantsByLifeCycle.put(lc, new HashSet<>());

for (Plant p : garden)
    palntsByLifeCycle.get(P.lifeCycle).add(p);

System.out.println(plantsByLifeCycle);
```

>더 짧고 명료하지만 안전하고 성능도 원래 버전과 비등
>
>안전하지 않은 형변환은 쓰지 않고, 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하여 레이블 필요 없음
>
>배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 원천봉쇄된다

<br>

**EnumMap의 성능이 ordinal을 쓴 배열에 비견되는 이유**

- 내부에서 배열을 사용하기 때문
- 내부 구현 방식을 안으로 숨겨서 `Map`의 타입 안전성과 배열의 성능을 모두 얻어낸 것
- `EnumMap`의 생성자가 받는 키 타입의 `Class` 객체는 한정적 타입 토큰으로 런타임 제네릭 타입 정보를 제공

<br>

---

<br>

### 🔍 스트림을 이용한 방식

**✏ #03 예제소스 | 스트림을 사용한 코드1 - EnumMap을 사용하지 않는다**

```java
System.out.println(Arrays.stream(garden)
        .collect(groupingBy(p -> p.lifeCycle)));
```

>`EnumMap`이 아닌 고유한 맵 구현체를 사용했기 때문에 `EnumMap`을 써서 얻은 공간과 성능 이점이 사라짐

<br>

**✏ #04 예제소스 | 스트림을 사용한 코드2 - EnumMap을 이용해 데이터와 열거 타입 매핑**

```java
System.out.println(Arrays.stream(garden)
        .collect(groupingBy(p -> p.lifeCycle, 
             ()-> new EnumMap<>(LifeCycle.class), toSet())));
```

>단순한 프로그램에서는 최적화 필요 없지만, 맵을 빈번히 사용하는 프로그램에서는 꼭 필요함

<br>

**스트림과 EnumMap의 차이**

- `EnumMap`버전은 언제나 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 생애주기에 속하는 식물이 있을때만 만듦

<br>

---

<br>

### 📚 두 열거 타입 값을 매핑하느라 `ordinal`을 두 번이나 쓴 배열들의 배열

**✏ #04 예제소스 | 배열들의 배열의 인덱스에 ordinal()을 사용 - 따라 하지 말 것**

```java
public enum Phase {
    SOLID, LIQUID, GAS;
    
    public enum Transition {
		MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
        
        // 행은 from의 ordinal을, 열은 to의 ordinal을 인덱스로 쓴다.
        private static final Transition[][] TRANSITIONS = {
            { null, MELT, SUBLIME },
            { FREEZE, null, BOIL },
            { DEPOSIT, CONDENSE, null }
        };
        
        // 한 상태에서 다른 상태로의 전이를 반환한다.
        public static Transition from(Phase from, Phase to) {
			return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```

>두 가지 `Phase`를 `Transition`와 매핑하도록 구현한 프로그램
>
>컴파일러는 `ordinal`과 배열 인덱스의 관계를 알 수 없음
>즉, `Phase`나 `Phase.Transition` 열거 타입을 수정하면서 상전이 표 `TRANSITIONS`를 함께 수정하지 않으면 런타임 오류 발생
>
>`ArrayIndexOutOfBoundsException` 또는 `NullPointerException`을 던지거나, 예외를 던지지 않고 이상하게 동작할 수 있음
>
>상전이 표의 크기는 상태의 가짓수가 늘어나면 제곱해서 커지며 `null`로 채워지는 칸도 증가

<br>

**✏ #05 예제소스 | 중첩 EnumMap으로 데이터와 열거 타입 쌍을 연결**

```java
public enum Phase {
    SOLID, LIQUID, GAS;
    
    public enum Transition {
		MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID), 
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);
        
        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }
        
        // 상전이 맵을 초기화한다.
        private static final Map<Phase, Map<Phase, Transition>>
            m = Stream.of(values()).collect(groupingBy(t -> t.from,
                   () -> new EnumMap<>(Phase.class),
                   toMap(t -> t.to, t -> t,
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));
        
        public static Transition from(Phase from, Phase to) {
			return m.get(frome).get(to);
        }
    }
}
```

>`Map<Phase, Map<Phase, Transition>>`은 "이전 상태에서 \`이후 상태에서 전이로의 맵\`에 대응시키는 맵"이라는 뜻
>
>맵의 맵을 초기화하기 위해 `Collector` 2개를 차례로 사용함

<br>

**✏ #06 예제소스 | EnumMap 버전에 새로운 상태 추가하기**

```java
public enum Phase {
    SOLID, LIQUID, GAS, PLASM;
    
    public enum Transition {
		MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID), 
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
        IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);
```

>기존 로직에서 잘 처리해주어 잘못 수정할 가능성이 극히 작음
>
>실제 내부에서는 맵들의 맵이 배열들의 배열로 구현되므로 낭비되는 공간과 시간도 거의 없으므로 안전하고 유지보수에 좋음

<br>

---

<br>

### 📌 핵심정리

**배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니, 대신 EnumMap을 사용하라**

**다차원 관계는 EnumMap<..., EnumMap<...>>으로 표현하라**

**"애플리케이션 프로그래머는 Enum.ordinal을 (웬만해서는) 사요하지 말아야 한다" 는 일반 원칙의 특수한 사례다**

<br>
