Effective Java의  54번째 아이템 "null이 아닌, 빈 컬렉션이나 배열을 반환하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. null 반환 vs 빈 컬렉션 반환

먼저 null을 반환하도록 메서드를 경우를 보자.

```java
private final List<Cheese> cheesesInStock = ...;

public List<Cheese> getCheeses() {
  return cheesesInStock.isEmpty() ? null
    : new ArrayList<>(cheesesInStock);
}
```

위 처럼 null을 반환하는 코드를 작성하면, 이 null 상황을 처리하는 코드를 추가로 작성해야 한다.

```java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
  System.out.println("좋았어, 바로 그거야.");
```

클라이언트에서 이러한 방어 코드를 빼먹으면 오류가 발생할 수 있다. 실제로 객체가 0개일 가능성이 거의 없는 상황에서는 수년 뒤에야 오류가 발생하기도 한다.

한편, null을 반환하려면 반환하는 쪽에서도 이 상황을 특별히 취급해줘야 해서 코드가 더 복잡해진다. 

<br>

그러면 코드를 어떻게 작성해야 할까? 다음은 빈 컬렉션을 반환하는 전형적인 코드다. 

```java
public List<Cheese> getCheeses() {
  return new ArrayList<>(cheesesInStock);
}
```

하지만 여기도 약간의 문제가 있다. 가능성은 작지만, 사용 패턴에 따라 빈 컬렉션 할당이 성능을 눈에 띄게 떨어뜨릴 수 있다. 

다행히 해법은 간단한데, 매번 똑같은 빈 '불변' 컬렉션을 반환하도록 하는 것이다. 

```java
public List<Cheese> getCheeses() {
  return cheesesInStock.isEmpty() ? Collections.emptyList() // 불변 컬렉션 반환
    : new ArrayList<>(cheesesInStock);
}
```

<br>

## 2.null 반환 vs 빈 배열 반환

배열을 쓸 때도 컬렉션과 마찬가지다. 절대 null을 반환하지 말고 길이가 0인 배열을 반환하자. 참고로 여기서 toArray 메서드에 넘긴 길이 0짜리 배열(`new Cheese[0]`)은 원하는 반환 타입을 알려주는 역할을 한다.

```java
public Cheese[] getCheeses() {
  return cheesesInStock.toArray(new Cheese[0]);
}
```

이 방식이 성능을 떨어뜨릴 것 같다면 길이 0짜리 배열을 미리 선언해두고 매번 그 배열을 반환하면 된다. 길이 0배열은 모두 불변이다.

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
  return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

이 최적화 버전의 getCheeses는 항상 EMPTY_CHEESE_ARRAY를 인수로 넘겨 toArray를 호출한다. 따라서 cheesesInStock이 비었을 때면 언제나 EMPTY_CHEESE_ARRAY를 반환하게 된다. 

<br>

단순히 성능을 개선할 목적이라면 toArray에 넘기는 배열을 미리 할당하는 건 추천하지 않는다. 오히려 성능이 떨어진다는 연구 결과도 있다.

```java
public Cheese[] getCheeses() {
  return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
}
```

<br>

## 3. 핵심 정리

* null이 아닌, 빈 배열이나 컬렉션을 반환하라. 
* null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다. 그렇다고 성능이 좋은 것도 아니다.
