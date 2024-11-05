Effective Java의 스물여섯 번째 아이템 "배열보다는 리스트를 사용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 배열과 제네릭 타입의 차이

배열과 제네릭 타입에는 중요한 차이가 두 가지 있다.

### 1) 배열은 공변(covariant)이다.

Sub가 Super의 하위 타입이라면 배열 Sub[]는 배열 Super[]의 하위 타입이 된다. 반면, 제네릭은 불공변이다. 즉, 서로 다른 타입 Type1과 Type2가 있을 때, List\<Type1\>은 List\<Type2\>의 하위 타입도 아니고 상위 타입도 아니다. 

이것만 보면 제네릭에 문제가 있다고 생각할 수도 있지만, 사실 문제가 있는 건 배열 쪽이다. 

```java
// 런타임에 실패한다.
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다."; // ArrayStoreException을 던진다.
```

```java
// 컴파일되지 않는다.
List<Object> ol = new ArrayList<Long>();
ol.add("타입이 달라 넣을 수 없다.");
```

어느 쪽이든 Long용 저장소에 String을 넣을 수는 없다. 다만 배열에서는 그 실수를 런타임에야 알게 되지만, 리스트를 사용하면 컴파일할 때 바로 알 수 있다. 

<br>

### 2) 배열은 실체화가 된다.

배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다. 그래서 위 예제처럼 Long 배열에 String을 넣으려 하면 ArrayStoreException이 발생한다. 반면, 앞서 이야기했듯 제네릭은 타입 정보가 런타임에는 소거(erasure)된다. 원소 타입을 컴파일타임에만 검사하며 런타임에는 알 수조차 없다는 뜻이다. 

참고로 소거는 제네릭이 지원되기 전의 레거시 코드와 제네릭 타입을 함께 사용할 수 있게 해주는 매커니즘으로, Java 5가 제네릭으로 순조롭게 전환될 수 있도록 해줬다.

<br>

이상의 두 가지 주요 차이로 인해 배열과 제네릭은 잘 어우러지지 못한다. 예를 들어 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다. 즉, 코드를 new List\<E\>[], new List\<String\>[], new E[] 식으로 작성하면 컴파일할 때 제네릭 배열 생성 오류를 일으킨다. 

<br>

## 2. 실체화 불가 타입

E, List\<E\>, List\<String\> 같은 타입을 실체화 불가 타입이라 한다. 쉽게 말해, 실체화되지 않아서 런타임에는 컴파일타임보다 타입 정보를 적게 가지는 타입이다.

소거 메커니즘 때문에 매개변수화 타입 가운데 실체화될 수 있는 타입은 List\<?\>와 Map\<?, ?\> 같은 비한정적 와일드카드 타입뿐이다. 배열을 비한정적 와일드카드 타입으로 만들 수는 있지만, 유용하게 쓰일 일은 거의 없다.

<br>

## 3. 배열로의 형변환

배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우, 대부분은 배열인 E[] 대신 컬렉션인 List\<E\>를 사용하면 해결된다. 코드가 조금 복잡해지고 성능이 살짝 나빠질 수도 있지만, 그 대신 타입 안전성과 상호운용성은 좋아진다.

생성자에서 컬렉션을 받는 Chooser 클래스를 예로 살펴보자. 이 클래스는 컬렉션 안의 원소 중 하나를 무작위로 선택해 반환하는 choose 메서드를 제공한다.

```java
public class Chooser {
  private final Object[] choiceArray;
  
  public Chooser(Collection choices) {
    choiceArray = choices.toArray();
  }
  
  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
```

이 클래스를 사용하려면 choose 메서드를 호출할 때마다 반환된 Object를 원하는 타입으로 형변환해야 한다. 혹시나 타입이 다른 원소가 들어 있었다면 런타임에 형변환 오류가 날 것이다.

<br>

클래스를 제네릭으로 만들어보자.

```java
public class Chooser<T> {
  private final T[] choiceArray;
  
  public Chooser(Collection<T> choices) {
    choiceArray = (T[]) choices.toArray();
  }
  
  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
```

이 클래스를 컴파일하면 비검사 형변환 경고가 발생한다. T가 무슨 타입인지 알 수 없으니 컴파일러는 이 형변환이 런타임에도 안전한지 보장할 수 없다는 경고다. 프로그램은 문제없이 동작한다. 단지 컴파일러가 안전을 보장하지 못할 뿐이다. 안전하다고 확신한다면 주석을 남기고 어노테이션을 달아도 되지만, 애초에 경고의 원인을 제거하는 편이 훨씬 낫다.

<br>

비검사 형변환 경고를 제거하려면 배열 대신 리스트를 쓰면 된다.

```java
public class Chooser<T> {
  private final List<T> choiceList;
  
  public Chooser(Collection<T> choices) {
    choiceList = new ArrayList<>(choices);
  }
  
  public T choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceList.get(rnd.nextInt(choiceList.size()));
  }
}
```

위의 Chooser는 오류나 경고 없이 컴파일된다.

<br>

## 4. 핵심 정리

- 배열과 제네릭에는 매우 다른 타입 규칙이 적용된다. 
- 배열은 공변이고 실체화되는 반면, 제네릭은 불공변이고 타입 정보가 소거된다. 그 결과 런타임에는 타입 안전하지만 컴파일타임에는 그렇지 않다. 제네릭은 반대다.
- 그래서 배열과 제네릭을 섞어 쓰기란 쉽지 않다. 컴파일 오류나 경고를 만나면, 가장 먼저 배열을 리스트로 대체하는 방법을 적용해보자.

<br>

## 5. Related Posts

- [제네릭 (Item 26)](https://heung27.github.io/posts/item-26-%EB%A1%9C-raw-%ED%83%80%EC%9E%85%EC%9D%80-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EB%A7%90%EB%9D%BC/)
- [비검사 경고 (Item 27)](https://heung27.github.io/posts/item-27-%EB%B9%84%EA%B2%80%EC%82%AC-%EA%B2%BD%EA%B3%A0%EB%A5%BC-%EC%A0%9C%EA%B1%B0%ED%95%98%EB%9D%BC/)
