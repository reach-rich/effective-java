Effective Java의 서른두 번째 아이템 "제네릭과 가변인수를 함께 쓸 때는 신중하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 제네릭과 가변인수

가변인수(varargs) 메서드와 제네릭은 자바 5 때 함께 추가되었지만, 잘 어울리지는 않는다. 가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 해주는데, 구현 방식에 허점이 있다. **가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어진다. 이 varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 알기 어려운 컴파일 경고가 발생한다.** 

실체화 불가 타입은 런타임에는 컴파일타임보다 타입 관련 정보를 적게 담고 있다. 그리고 거의 모든 제네릭과 매개변수화 타입은 실체화되지 않는다. 메서드를 선언할 때 실체화 불가 타입으로 varargs 매개변수를 선언하면 컴파일러가 경고를 보낸다. 가변인수 메서드를 호출할 때도 varargs 매개변수가 실체화 불가 타입으로 추론되면, 그 호출에 대해서도 경고를 낸다. 

```java
warning: [unchecked] Possible heap pollution from
  parameterized vararg type List<String>
```

**매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다.** 이렇게 다른 타입 객체를 참조하는 상황에서는 컴파일러가 자동 생성한 형변환이 실패할 수 있으니, 제네릭 타입 시스템이 약속한 타입 안정성의 근간이 흔들려버린다.

다음 메서드를 예로 생각해보자.

```java
static void dangerous(List<String>... stringLists) {
  List<Integer> intList = List.of(42);
  Object[] objects = stringLists;
  objects[0] = intList; // 힙 오염 발생
  String s = stringLists[0].get[0]; // ClassCastException
}
```

이 메서드에서는 형변환하는 곳이 보이지 않는데도 인수를 건네 호출하면 ClassCastException을 던진다. 마지막 줄에 컴파일러가 생성한 (보이지 않는) 형변환이 숨어 있기 때문이다. 이처럼 타입 안전성이 깨지니 **제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.** 

>제네릭 배열을 프로그래머가 직접 생성하는 건 허용하지 않으면서 제네릭 varargs 매개변수를 받는 메서드를 선언할 수 있게 한 이유는 무엇일까?
>
>제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 메서드가 실무에서 매우 유용하기 때문이다. 자바 라이브러리도 이런 메서드를 여럿 제공한다. 대표적으로 아래와 같은 메서드가 있다.
>
>- Arrays.asList(T... a)
>- ßCollections.addAll(Collection<? super T> c, T... elements)
>- EnumSet.of(E first, E... rest)
>
>이 메서드들은 앞서 보여준 메서드와 달리 타입 안전하다.

<br>

## 2. @SafeVarargs

자바 7 전에는 제네릭 가변인수 메서드의 작성자가 호출자 쪽에서 발생하는 경고에 대해서 해줄 수 있는 일이 없었다. 사용자는 이 경고들을 그냥 두거나 호출하는 곳마다 @SuppressWarnings("unchecked") 어노테이션을 달아 경고를 숨겨야 했다.

자바 7에서는 @SafeVarargs 어노테이션이 추가되어 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있게 되었다. **@SafeVarargs 어노테이션은 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치다.** 컴파일러는 이 약속을 믿고 그 메서드가 안전하지 않을 수 있다는 경고를 더 이상 하지 않는다.

<br>

### 주의할 점

**메서드가 안전한 게 확실하지 않다면 절대 @SafeVarargs 어노테이션을 달아서는 안 된다.**

가변인수 메서드를 호출할 때 varargs 매개변수를 담는 제네릭 배열이 만들어진다는 사실을 기억하자. **메서드가 이 배열에 아무것도 저장하지 않고 그 배열의 참조가 밖으로 노출되지 않는다면 타입 안전하다.** 달리 말하면, 이 varargs 매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수들을 전달하는 일만 한다면(varargs 목적대로만 쓰인다면) 그 메서드는 안전하다.

다음은 varargs 매개변수 배열의 참조가 밖으로 노출되어 타입 안전성을 깨진 예제다.

```java
static <T> T[] toArray(T... args) {
  return args;
}
```

이 메서드가 반환하는 배열의 타입은 이 메서드에 인수를 넘기는 컴파일타임에 결정되는데, 그 시점에는 컴파일러에게 충분한 정보가 주어지지 않아 타입을 잘못 판단할 수 있다. 따라서 자신의 varargs 매개변수 배열을 그대로 반환하면 힙 오염을 이 메서드를 호출한 쪽의 콜스택으로까지 전이하는 결과를 낳을 수 있다.

```java
static <T> T[] pickTwo(T a, T b, T c) {
  switch (ThreadLocalRandom.current().nextInt(3)) {
    case 0: return toArray(a, b);
    case 1: return toArray(a, c);
    case 2: return toArray(b, c);
  }
  throw new AssertionError();
}
```

이 메서드를 본 컴파일러는 toArray에 넘길 T 인스턴스 2개를 담을 varargs 매개변수 배열을 만드는 코드를 생성한다. 이때 생성된 배열의 타입은 Object[] 이다. 그리고 toArray 메서드가 돌려준 이 배열이 그대로 pickTwo를 호출한 클라이언트까지 전달된다. 즉, pickTwo는 항상 Object[] 타입 배열을 반환한다.

다음은 pickTwo를 사용하는 main 메서드다.

```java
public static void main(String[] args) {
  String[] attributes = pickTwo("좋은", "빠른", "저렴한");
}
```

아무런 문제가 없는 메서드이니 별다른 경고 없이 컴파일된다. 하지만 실행하려 들면 ClassCastException을 던진다. pickTwo의 반환값을 attributes에 저장하기 위해 String[]로 형변환하는 코드를 컴파일러가 자동으로 생성하기 때문이다. 

<br>

위의 예는 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다는 점을 상기시킨다. 단, 예외가 두 가지 있다.

- @SafeVarargs로 제대로 어노테이트된 또 다른 varargs 메서드에 넘기는 것은 안전하다.
- 그저 이 배열 내용의 일부 함수를 호출만 하는 (varargs를 받지 않는) 일반 메서드에 넘기는 것도 안전하다.

다음은 제네릭 varargs 매개변수를 안전하게 사용하는 전형적인 예다.

```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
  List<T> result = new ArrayList<>();
  for (List<? extends T> list : lists)
    result.addAll(list);
  return result;
}
```

<br>

### 규칙

@SafeVarargs 어노테이션을 사용해야 할 때를 정하는 규칙은 간단하다.

**제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 @SafeVarargs를 달라.**

그래야 사용자를 헷갈리게 하는 컴파일러 경고를 없앨 수 있다. 이 말은 안전하지 않은 varargs 메서드는 절대 작성해서는 안 된다는 뜻이기도 하다. 다음 두 조건을 만족하는 제네릭 varargs 메서드는 안전하다. 둘 중 하나라도 어겼다면 수정하라.

- varargs 매개변수 배열에 아무것도 저장하지 않는다.
- 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다.

> @SafeVarargs 어노테이션은 재정의할 수 없는 메서드에만 달아야 한다. 재정의한 메서드도 안전할지는 보장할 수 없기 때문이다. 자바 8에서 이 어노테이션은 오직 정적 메서드와 final 인스턴스 메서드에만 붙일 수 있고, 자바 9부터는 private 인스턴스 메서드에도 허용된다.

<br>

## 3. 핵심 정리

- 가변인수 기능은 배열을 노출하여 추상화가 완벽하지 못하고 배열과 제네릭의 타입 규칙이 서로 다르기 때문에, 가변인수와 제네릭은 궁합이 좋지 않다.
- 제네릭 varargs 매개변수는 타입 안전하지는 않지만 허용된다.
- 메서드에 제네릭  (혹은 매개변수화된) varargs 매개변수를 사용하고자 한다면, 먼저 그 메서드가 타입 안전한지 확인한 다음 @SafeVarargs 어노테이션을 달아 사용하는 데 불편함이 없게끔 하자.

<br>

## 4. Related Posts

- 가변인수 (Item 53)
- [실체화 불가 타입 (Item 28)](https://heung27.github.io/posts/item-28-%EB%B0%B0%EC%97%B4%EB%B3%B4%EB%8B%A4%EB%8A%94-%EB%A6%AC%EC%8A%A4%ED%8A%B8%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC/)
- [@SuppressWarnings (Item 27)](https://heung27.github.io/posts/item-27-%EB%B9%84%EA%B2%80%EC%82%AC-%EA%B2%BD%EA%B3%A0%EB%A5%BC-%EC%A0%9C%EA%B1%B0%ED%95%98%EB%9D%BC/)
