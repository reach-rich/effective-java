### 🔎 한정적 와일드카드 타입

**✏ #01 예제소스 | 와일드카드 타입을 사용하지 않은 pushAll 메서드 (결함)**

```java
public void pushAll(Iterable<E> src) {
	for (E e : src)
        push(e);
}
```

>`Iterable src`의 원소 타입이 스택의 원소 타입과 일치하면 정상 작동
>
>`Stack<Number>` 선언 후 `pushAll(intVal)` 호출 시 오류 메시지 발생
>
>매개변수 타입이 불공변이라서 오류 발생

**✏ #02 예제소스 | E 생산자(producer) 매개변수에 와일드카드 타입 적용**

```java
public void pushAll(Interable<? extends E> src) {
    for (E e : src)
        push(e);
}
```

>`Iterable<? extends E>`가 의미하는 것은 `E`의 `Iterable`이 아니라 `E`의 하위 타입의 `Iterable`이어야 한다는 뜻

<br>

**✏ #03 예제소스 | 와일드카드 타입을 사용하지 않은 popAll 메서드 (결함)**

```java
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

>컬렉션의 원소 타입이 스택의 원소타입과 일치한다면 문제 없음
>
>`Stack<Number>`의 원소를 `Object1`용 컬렉션으로 옮기려 할때 오류 발생
>
>와일드카드 타입으로 해결 가능

<br>

**✏ #04 예제소스 | E 소비자(consumer) 매개변수에 와일드카드 타입 적용**

```java
public void popAll(Collection<? super E> dst) {
	while (!isEmpty())
		dst.add(pop());
}
```

>`E`의 `Collection`이 아니라 `E`의 상위 타입의 `Collection`

<br>

**✔ 유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라**

<br>

---

<br>

### ❓ 언제 와일드카드 타입을 사용할까 

**쓰지 말아야하는 경우**

- 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 써도 좋을게 없음
- 타입을 정확히 지정해야하는 상황

<br>

**사용해야 하는 경우**

- 펙스(PECS) : producer-extends, consumer-super

  **매개변수화 타입 `T`가 생산자: `<? extends T>`, 매개변수화 타입 `T`가 소비자: `<? spuber T>`**

<br>

**✏ #05 예제소스 | 생산자 매개변수에 와일드카드 타입 적용**

```java
public Chooser(Collection<T> choices)

public Chooser(Collection<? extends T> choices)
```

>T를 확장하는 와일드카드 타입을 사용해 선언해야한다

<br>

**✏ #06 예제소스 | 생산자 매개변수에 와일드카드 타입 적용**

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2)
    
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
```

>s1과 s1 모두 E의 생산자이므로 공식에 따라 변경
>
>반환 타입에는 한정적 와일드카드 타입을 사용하면 안 된다

<br>

받아들여야할 매개변수를 받고 거절해야 할 매개변수는 거절하는 작업이 알아서 이루어짐

클래스 사용자가 와일드카드 타입을 신경 써야 한다면 그 API에 무슨 문제가 있을 가능성이 큼

<br>

**❗ 주의해야할 점**

자바 7까지는 타입 추론 능력이 강력하지 못해 문맥에 맞는 반환 타입을 명시해야 했음

```tex
Union.java:14: error: imcompatible types
		Set<Number> numbers = union(integers, doubles);
```

```java
Set<Number> numbers = Union.<Number>union(integers, doubles);
```

>컴파일러가 올바른 타입을 추론하지 못할 때면 언제든 명시적 타입 인수(explicit type argument)를 사용해서 타입을 알려주면 된다

<br>

---

<br>

### 📑 max 메서드 

**✏ #07 예제소스 | PECS공식 두 번 사용**

```java
public static <E extends Comparable<E>> E max(List<E> list)
```

```java
public static <E extends Comparable<? super E>> E max(List<? extends E> list)
```

>입력 매개변수에서는 `E` 인스턴스를 생산하므로 원래의 `List<E>`를 `List<? extends E>`로 수정
>
>타입 매개변수 `E`가 `Comparable<E>`를 확장 한다고 정의했는데, 이때 `Comparable<E>`는 `E` 인스턴스를 소비하므로 `Comparable<E>`를 한정적 와일드카드 타입 `Comparable<? spuber E>`로 대체
>
>`Comparable`은 언제나 소비자이므로, 일반적으로 `Comparable<? super E>`를 사용하는편이 나음
>
>`Comprator`도 마찬가지로 `Comprator<? super E>`를 사용하는 편이 나음

<br>

**🤔 이렇게 복잡하게까지 만들어야 하는 이유**

```java
List<ScheduledFuture<?>> scheduledFutures = ...;
```

>오직 수정된 `max`로만 처리할 수 있다
>
>`ScheduledFuture`가 `Comparable<ScheduledFuture>`를 구현하지 않았기 때문이다
>
>`ScheduledFuture`는 `Delayed`의 하위 인터페이스이고, `Delayed`는 `Comparable<Delayed>`를 확장했다
>
>`ScheduledFuture`의 인스턴스는 다른 `ScheduledFuture` 인스턴스뿐 아니라 `Delayed` 인스턴스와도 비교할 수 있다

**✔ Comparable(혹은 Comparator)을 직접 구현하지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원하기 위해 와일드카드가 필요하다**

<br>

---

<br>

### 📑 sawp 메서드 

타입 매개변수와 와일드카드에는 공통되는 부분이 있어서, 메서드를 정의할 때 둘 중 어느 것을 사용해도 괜찮은 경우

<br>

**✏ #08 예제소스 | swap 메서드의 두 가지 선언**

```java
public static <E> void swap(List<E> list, int i, int j);
```

```java
public static void swap(List<?> list, int i, int j);
```

>public API라면 간단한 두 번째가 낫다
>
>**메서드 선언에 타입 매개변수가 한 번만 나오면 와일드 카드로 대체하라**
>
>비한정적 타입 매개변수라면 비한정적 와일드카드로 바꾸고, 한정적 타입 매개변수라면 한정적 와일드카드로 바꾸면 된다

<br>

**✏ #09 예제소스**

```java
public static void swap(List<?> list, int i, int j) {
	list.set(i, list.set(j, list.get(i)));
}
```

```tex
Swap.java:5: error: incompatible types: Object cannot be converted to CAP#1
		list.set(i, list.set(j, list.get(i)));
										^
where CAP#1 is a fresh type-variable:
	CAP#1 extends Object from capture of ?
```

>리스트의 타입이 `List<?>`인데, `List<?>`에는 `null` 외에는 어떤 값도 넣을 수 없다
>
>와일드카드 타입의 실제 타입을 알려주는 메서드를 `private` 도우미 메서드로 따로 작성하여 활용하는 방법으로 해결가능

<br>

**✏ #10 예제소스**

```java
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

>와일드카드 타입을 실제 타입으로 바꿔주는 `private` 도우미 메서드
>
>`swapHelper` 메서드는 리스트가 `List<E>`임을 알고 있음
>
>리스트에서 꺼낸 값의 타입은 항상 `E`이고, `E` 타입의 값이라면 이 리스트에 넣어도 안전함을 알고 있음

<br>

---

<br>

### 💡 매개변수(parameter)와 인수(argument)의 차이

**: 매개변수는 매서드 선언에 정의한 변수이고, 인수는 메서드 호출 시 넘기는 '실제값'이다**

<br>

```java
void add(int value) { ... }
add(10)
```

>value는 매개변수이고 10은 인수이다

<br>

```java
class Set<T> {...}
Set<Integer> = ...;
```

>T는 매개변수가 되고, Integer는 타입 인수가 된다
>
>보통은 이 둘을 명확히 구분하지 않으나, 자바 언어 명세에서는 구분한다

<br>

---

<br>

### 📌 핵심정리

**조금 복잡하더라도 와일드카드 타입을 적용하면 API가 훨씬 유연해진다**

**그러니 널리 쓰일 라이브러리를 작성한다면 반드시 와일드카드 타입을 적절히 사용해줘야 한다**

**PECS공식을 기억하자**

**즉, 생산자(producer)는 extends를 소비자(consumer)는 super를 사용한다**

**Comparable과 Comparator는 모두 소비자라는 사실도 잊지 말자**

<br>
