Effective Java의  52번째 아이템 "다중정의는 신중히 사용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 다중정의 문제 - CollectionClassifier

```java
public class CollectionClassifier {
  public static String classify(Set<?> s) {
    return "집합";
  }
  
  public static String classify<List<?> lst) {
    return "리스트";
  }
  
  public static String classify<Collection<?> c) {
    return "그 외";
  }
  
  public static void main(String[] args) {
    Collection<?>[] collections = {
      new HashSet<String>(),
      new ArrayList<BigInteger>(),
      new HashMap<String, String>().values()
    };
    
    for (Collection<?> c : collections)
      System.out.println(classify(c));
  }
}
```

위 예제는 실제로 "집합", "리스트", "그 외"를 차례로 출력할 것 같지만, 실제로 수행해보면 "그 외"만 세번 연달아 출력한다. 

그 이유는 다중정의(오버로딩)된 세 classify 중 **어느 메서드를 호출할지가 컴파일타임에 정해지기 때문이다.** 컴파일타임에는 for 문 안의 c는 항상 Collection<?> 타입이다. 런타임에는 타입이 매번 달라지지만, 호출할 메서드를 선택하는 데는 영향을 주지 못 한다. 

이렇게 직관과 어긋나는 이유는 **재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택**되기 때문이다. 

<br>

이 문제는 CollectionClassifier의 모든 classify 메서드를 하나로 합친 후 instanceof로 명시적으로 검사하면 말씀히 해결된다. 

```java
public static String classify(Collection<?> c) {
  return c instanceof Set ? "집합" : c instanceof List ? "리스트" : "그 외";
}
```

<br>

**헷갈릴 수 있는 코드는 작성하지 않는 게 좋다.** 특히나 공개 API라면 더욱 신경 써야 하며 다중정의가 혼동을 일으키는 상황을 피해야 한다.

정확히 어떻게 사용했을 때 다중정의가 혼란을 주느냐에 대해서는 논란의 여지가 있다. **안전하고 보수적으로 가려면 매개변수 수가 같은 다중정의는 만들지 말자.** 특히 가변인수를 사용하는 메서드라면 다중정의를 아예 하지말아야 한다. 대신 메서드 이름을 다르게 짓도록 하자. 

<br>

## 2. 다중정의 문제 - SetList

```java
public class SetList {
  public static void main(String[] args) {
    Set<Integer> set = new TreeSet<>();
    List<Integer> list = new ArrayList<>();
    
    for (int i=-3; i<3; i++) {
      set.add(i);
      list.add(i);
    }
    
    for (int i=0; i<3; i++) {
      set.remove(i);
      list.remove(i);
    }
    
    System.out.println(set + " " + list);
  }
}
```

이 프로그램은 "[-3, -2, -1] [-3, -2, -1]"을 출력하리라 예상할 것이다. 하지만 실제로는 "[-3, -2, -1] [-2, 0, 2]"를 출력한다.

어떻게된 일일까? set remove의 시그니처는 remove(Object)고 list remove는 remove(int index)다. list의 remove는 지정한 위치의 원소를 제거하는 기능을 수행한다. 리스트의 0번째, 1번째, 2번째  원소를 제거하게 된다. 

이 문제는 list.remove의 인수를 Integer로 형변환하여 올바른 다중정의 메서드를 선택하게 하면 해결된다. 이렇게 하면 원래 기대한 "[-3, -2, -1] [-3, -2, -1]"를 출력한다. 

이 예제가 혼란스러웠던 이유는 List\<E> 인터페이스가 remove(Object)와 remove(int)를 다중정의했기 때문이다. 자바 언어에 제네릭과 오토박싱을 더한 결과 List 인터페이스가 취약해졌다. 다행히 같은 피해를 입은 API는 거의 없지만, 다중정의 시 주의를 기울여야 할 근거로는 충분하다. 

<br>

## 3. 다중정의 문제 - 메서드 참조

```java
// 1번. Thread의 생성자 호출
new Thread(System.out::println).start();

// 2번. ExecutorService의 submit 메서드 호출
ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println);
```

자바 8에서 도입한 람다와 메서드 참조 역시 다중정의 시의 혼란을 키웠다. 1번과 2번이 모습은 비슷하지만, 2번만 컴파일 오류가 난다. 

넘겨진 인수는 모두 `System.out::println`으로 똑같고, 양쪽 모두 Runnable을 받는 형제 메서드를 다중정의하고 있다. 그런데 한쪽만 실패하는 이유는 submit 다중정의 메서드 중에서는 Callable\<T>를 받는 메서드도 있다는 데 있다. 

다중정의된 메서드들이 함수형 인터페이스를 인수로 받을 때, 비록 서로 다른 함수형 인터페이스라도 인수 위치가 같으면 혼란이 생긴다. 따라서 **메서드를 다중정의 할 때, 서로 다른 함수형 인터페이스도 같은 위치이의 인수로 받아서는 안 된다.**

<br>

## 4. 핵심 정리

* 일반적으로 매개변수 수가 같을 때는 다중정의를 피하는 것이 좋다.
* 상황에 따라, 특히 생성자라면 이 조언을 따르기가 불가능할 수 있다. 그럴 때는 헷갈릴 만한 매개변수는 형변환하여 정확한 다중정의 메서드가 선택되도록 해야 한다.
* 이것이 불가능하면 같은 객체를 입력받는 다중정의 메서드들이 모두 동일하게 동작하도록 만들어야 한다.
* 그렇지 못하면 프로그래머들은 다중정의된 메서드나 생성자를 효과적으로 사용하지 못할 것이고, 의도대로 동작하지 않는 이유를 이해하지도 못할 것이다. 

