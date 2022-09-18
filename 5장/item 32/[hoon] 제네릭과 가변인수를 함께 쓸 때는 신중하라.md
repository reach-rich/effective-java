## 1. 들어가기

자바 5에서 제네릭과 함께 가변인수도 함께 추가되었습니다.

가변인수란 무엇일까요?

가변인수에 대해 간단히 알아보겠습니다.

## 2. 가변인수 (varargs)

자바는 오버로딩이라는 기능이 구현되어 있기 때문에

매개변수의 개수와 타입이 달라도 동일한 메소드명으로 기능을 구현할 수 있습니다.

하지만 매개변수의 개수가 무한히 늘어난다면 일일이 매개변수를 하나하나 오버로딩 해줘야할까요?

```java
  String divide(String s1, String s2){...}
  String divide(String s1, String s2, String s3){...}
  String divide(String s1, String s2, String s3, String s4){...}
  String divide(String s1, String s2, String s3, String s4, String s5){...}
```

<div class="div-post-img" style="text-align:center">
  <img src="https://ss-hoon.github.io/assets/img/effective_java/item2/wtf.png" width="30%" height="20%" />
</div>

개발자들은 단순 반복을 매우 싫어합니다.

그래서 JDK 1.5부터 매개변수의 개수를 동적으로 지정해줄 수 있게 되었는데 이것이 바로 **가변인수**입니다.

가변인수를 통해 위의 예시를 바꿔보면 다음과 같습니다.

```java
  String divide(String... s){...}
```

## 3. 제네릭-가변인수의 문제점

다시 돌아와서 자바 5때 함께 추가되었던 제네릭과 가변인수는 서로 잘 어우러지리라 기대했지만,

아쉽게도 그렇지는 않습니다.

런타임 시에 정보를 적게 담고 있는 제네릭과 매개변수화 타입은 실체화되지 않기 때문에

메서드 선언 시 실체화 불가 타입으로 varargs 매개변수를 선언하면 컴파일러가 경고를 보냅니다.

```java
  static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringList;
    objects[0] = intList;               // 힙 오염 발생
    String s = stringLists[0].get(0);   // ClassCastException 발생
  }
```

```
<경고>
  warning: [unchecked] Possible heap pollution from
      parameterized vararg type List<String>

<에러>
  Exception in thread "main" java.lang.ClassCastException
```

이처럼 타입 안전성이 깨지므로 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않습니다.

> 💡 힙 오염(heap pollution)
>
>  보통 선언된 매개변수 타입의 변수로 다른 매개변수 타입의 변수를 가리키는 경우에 발생
>
>  주로 unchecked 경고를 발생시키는 부분에서 발생하고, ClassCastException을 발생시킨다.
>
>  [http://kimalam.blogspot.com/2019/09/java-heap-pollution.html](http://kimalam.blogspot.com/2019/09/java-heap-pollution.html)

> 💡 SafeVarargs 어노테이션
>
>  자바 7부터 추가된 어노테이션으로 사용자 쪽에서 발생하는 힙 오염 경고를 숨길 수 있다.
>
>  메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치

## 4. 제네릭-가변인수의 조화

앞서 알아보았듯이 제네릭과 가변인수를 함께 사용하면 힙 오염을 발생할 가능성이 있었습니다.

그러나 제네릭과 가변인수를 함께 사용하는 것은 실무에서 매우 유용합니다.

자바 라이브러리도 가변인수를 사용한 메서드를 여럿 제공합니다.

`Arrays.asList(T... a)`, `Collections.addAll(Collection<? super T> c, T... elements)`

다행인 점은 해당 메서드들은 타입 안전하다는 점입니다.

어떻게 타입을 안전하게 만들 수 있을까요?

먼저, 앞선 예시에서 힙 오염이 발생한 이유를 알아보겠습니다.

```java
  static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringList;      // 1. stringLists의 참조가 외부로 노출
    objects[0] = intList;               // 2. 외부로 노출된 stringLists 배열에 데이터가 변경 
    String s = stringLists[0].get(0);   
  }
```

즉, 힙 오염이 발생한 이유 두 가지 모두 지킨다면 타입을 안전하게 만들 수 있습니다.

둘 중 하나만 지킨다고 하더라도 힙 오염에 대해 자유로울 수 없습니다.

아래는 가변인수 배열에 데이터를 아무것도 저장하지 않지만, 참조가 외부로 노출된 예시입니다.

```java
  static <T> T[] toArray(T... args) {
    return args;    // 가변인수 배열을 외부로 노출
  }

  static <T> T[] pickTwo(T a, T b, T c) {
    switch(ThreadLocalRandom.current().nextInt(3)) {
      case 0: return toArray(a, b);
      case 1: return toArray(a, c);
      case 2: return toArray(b, c);
    }

    throw new AssertionError();
  }

  public static void main(String[] args) {
    String[] attributes = pickTwo("좋은", "빠른", "저렴한");
  }
```

해당 예시는 아무런 경고 없이 컴파일되지만, 실행하면 ClassCastException이 발생합니다.

그 이유는 pickTwo의 반환 값을 attributes에 저장할 때,

String[]로 형변환하는 코드를 컴파일러가 자동생성하기 때문입니다.

컴파일러는 toArray에 넘길 T 인스턴스 2개를 담을 가변인수 배열을 만드는 코드를 생성하는데

pickTwo에 어떤 타입의 객체를 넘기더라도 담을 수 있는 가장 구체적인 타입인 Object[]을 반환합니다.

> Q1. 가변인수 배열이 생성되면 무조건 Object[] 타입으로 생성되는가?

그래서 이 배열이 호출한 클라이언트까지 전해지고 ClassCastException이 발생하게 됩니다.

그러므로 힙 오염이 발생한 이유 두 가지를 모두 지켜야 **타입 안전한 가변인수 메서드**를 만들 수 있습니다.

```java
  @SafeVarargs
  static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for(List<? extends T> list : lists) {
      result.addAll(list);
    }

    return result;
  }
```

## 5. 힙 오염을 발생시키지 않는 예외

힙 오염은 가변인수 배열에 데이터를 저장하거나, 참조가 외부로 노출된 코드에서 발생했습니다.

하지만, 이것도 예외가 있는데 다음과 같습니다.

1. @SafeVarargs로 제대로 어노테이트된 또 다른 가변인수 메서드에 넘기는 것은 안전하다.

2. 배열의 내용을 일부 함수를 호출만 하는(varargs를 받지 않는) 일반 메서드에 넘기는 것도 안전하다.

> Q2. "**배열의 내용을 일부 함수를 호출만 하는**" 이게 무슨말?

## 6. 제네릭-가변인수의 회피

타입 안전하지 않은 제네릭-가변인수를 List로 대체해 회피하는 방법도 있습니다.

위의 가변인수 배열을 외부로 노출한 예시를 변경해보겠습니다.

```java
  static <T> List<T> pickTwo(T a, T b, T c) {
    switch(ThreadLocalRandom.current().nextInt(3)) {
      case 0: return List.of(a, b);
      case 1: return List.of(a, c);
      case 2: return List.of(b, c);
    }

    throw new AssertionError();
  }

  public static void main(String[] args) {
    List<String> attributes = pickTwo("좋은", "빠른", "저렴한");
  }
```

이 방법은 배열 없이 제네릭만 사용하므로 타입 안전합니다.

## 7. 정리

이번 포스트는 가변인수와 제네릭을 함께 사용할 때, 타입을 안전하게 만드는 방법에 대해 알아보았습니다.

가변인수는 배열을 노출해 추상화가 완벽하지 못하기 때문에 제네릭과 궁합이 좋지 않습니다.

그래서 타입이 안전하지 않아 힙 오염의 가능성이 존재하므로 주의해서 사용해야 합니다.

둘을 함께 사용할 때는 @SafeVarargs 어노테이션을 통해 사용에 불편함이 없도록 합시다.

## 8. 참조

* [https://sleepyeyes.tistory.com/29](https://sleepyeyes.tistory.com/29)