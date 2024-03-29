Effective Java의  44번째 아이템 "표준 함수형 인터페이스를 사용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. API 작성 모범 사례

자바가 람다를 지원하면서 API를 작성하는 모범 사례도 크게 바뀌었다. 

예를 들어 상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는 템플릿 메서드 패턴의 매력이 크게 줄었다. 이를 대체하는 현대적인 해법은 같은 효과의 **함수 객체**를 받는 정적 팩터리나 생성자를 제공하는 것이다. 이때 함수형 매개변수 타입을 올바르게 선택해야 한다.

<br>

LinkedHashMap을 생각해보자. 이 클래스의 protected 메서드인 removeEldestEntry를 재정의하면 캐시로 사용할 수 있다. 맵에 새로운 키를 추가하는 put 메서드는 이 메서드를 호출하여 true가 반환되면 맵에서 가장 오래된 원소를 제거한다. 

예를 들어 아래처럼 재정의하면 맵에 원소가 100개가 될 때까지 커지다가, 그 이상이 되면 새로운 키가 더해질 때마다 가장 오래된 원소를 하나씩 제거한다. 즉, 가장 최근 원소 100개를 유지한다.

```java
protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
  return size() > 100;
}
```

잘 동작하지만 람다를 사용하면 훨씬 잘 해낼 수 있다. LinkedHashMap을 오늘날 다시 구현한다면 함수 객체를 받는 정적 팩터리나 생성자를 제공했을 것이다. 함수형 인터페이스는 다음 처럼 선언할 수 있다.

```java
@FunctionalInterface
interface EldestEntryRemovalFunction<K, V> {
  boolean remove(Map<K, V> map, Map.Entry<K, V> eldest);
}
```

이 인터페이스도 잘 동작하기는 하지만, 굳이 사용할 이유는 없다. 자바 표준 라이브러리에 이미 같은 모양의 인터페이스가 준비되어 있기 때문이다. 이 경우에는 `BiPredicate<Map<K, V>, Map.Entry<K, V>>`를 사용할 수 있다.

<br>

## 2. 표준 함수형 인터페이스

`java.util.function` 패키지를 보면 다양한 용도의 표준 함수형 인터페이스가 담겨 있다. **필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라.** 그러면 API가 다루는 개념의 수가 줄어들어 익히기 더 쉬워진다. 또한, 표준 함수형 인터페이스들은 유용한 디폴트 메서드를 많이 제공하므로 다른 코드와의 상호운용성도 크게 좋아질 것이다. 

패키지에는 총 43개의 인터페이스가 담겨 있다. 전부 기억하긴 어렵겠지만, 기본 인터페이스 6개만 기억하면 나머지를 충분히 유추해 낼 수 있다. 이 기본 인터페이스들은 모두 참조 타입용이다.

* Operator
  * UnaryOperator - `T apply(T t)`
  * BinaryOperator - `T apply(T t1, T t2)`
* Predicate - `boolean test(T t)`
* Function - `R apply(T t)`
* Supplier - `T get()`
* Consumer - `void accept(T t)`

<br>

표준 함수형 인터페이스는 대부분 기본 타입만 지원한다. 그렇다고 **기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자.** 동작은 하지만 계산량이 많을 때는 성능이 처참히 느려질 수 있다.

<br>

## 3. 함수형 인터페이스를 직접 작성해야 하는 경우

이제 대부분 상황에서는 직접 작성하는 것보다 표준 함수형 인터페이스를 사용하는 편이 나음을 알았을 것이다. 그러나 표준 인터페이스 중 필요한 용도에 맞는 것이 없다면 직접 작성해야 한다. 예를 들어 매개변수 3개를 받는 Predicate라든가 검사 예외를 던지는 경우가 있을 수 있다. 

그런데 구조적으로 똑같은 표준 함수형 인터페이스가 있더라도 직접 작성해야만 할 때가 있다. 자주 보아온 `Comparator<T> `인터페이스를 떠올려보자. 구조적으로는 `ToIntBiFunction<T, U>`와 동일하다. 심지어 자바 라이브러리에 `Compatator<T>`를 추가할 당시 `ToIntBiFunction<T, U>`가 이미 존재했더라도 `ToIntBiFunction<T, U>`를 사용하면 안 됐다.

Comparator가 독자적인 인터페이스로 살아남아야 하는 이유가 몇 개 있다.

* API에서 굉장히 자주 사용되는데, 지금의 이름이 그 용도를 아주 훌륭히 설명해준다.
* 구현하는 족에서 반드시 지켜야 할 규약을 담고 있다. 
* 비교자들을 변환하고 조합해주는 유용한 디폴트 메서드들을 듬뿍 담고 있다.

<br>Comparator 특성을 정리하면 다음의 세 가지인데, **이 중 하나 이상을 만족한다면 전용 함수형 인터페이스를 구현해야 하는 건 아닌지 진중히 고민해야 한다.**

* 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.
* 반드시 다라야 하는 규약이 있다.
* 유용한 디폴트 메서드를 제공할 수 있다.

전용 함수형 인터페이스를 작성하기로 했다면, 자신이 작성하는 것이 다른 것도 아닌 '인터페이스'임을 명심해야 한다. 아주 주의해서 설계해야 한다는 뜻이다. 

<br>

앞의 EldestEntryRemovalFunction 인터페이스에 **@FunctionalInterface** 애너테이션이 달려 있음에 주목하자. 이 애너테이션을 사용하는 이유는 @Override를 사용하는 이유와 비슷하다. 프로그래머의 의도를 명시하는 것으로, 크게 세 가지 목적이 있다.

* 해당 클래스의 코드나 설명 문서를 읽을 이에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다.
* 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다.
* 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.

그러니 **직접 만든 함수형 인터페이스에는 항상 @FunctionalInterface 애너테이션을 사용하라.**

<br>

## 4. 함수형 인터페이스를 사용할 때 주의할 점

함수형 인터페이스를 API에서 사용할 때 주의해야 할 점이 있다. **서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중정의해서는 안된다.** 클라이언트에게 불필요한 모호함을 안겨줄 뿐이며, 이 모호함으로 인해 실제로 문제가 일어나기도 한다. 

예를 들어 ExcecutorService의 submit 메서드는 Callable\<T>를 받는 것과 Runnable을 받는 것을 다중정의했다. 그래서 올바른 메서드를 알려주기 위해 형변환해야 할 때가 왕왕 생긴다. 

<br>

## 5. 핵심 정리

* 자바도 람다를 지원하니, API를 설계할 때 람다도 염두에 두어야 한다.
* 입력값과 반환값에 함수형 인터페이스 타입을 활용하라.
* 보통은 `java.util.function` 패키지의 표준 함수형 인터페이스를 사용하는 것이 가장 좋은 선택이다.
* 단, 흔치는 않지만 직접 새로운 함수형 인터페이스를 만들어 쓰는 편이 나을 수도 있다.

<br>

## 6. Related Posts

* 박싱된 기본 타입 (Item 61)
* 인터페이스 (Item 21)
* 다중정의 (Item 52)
