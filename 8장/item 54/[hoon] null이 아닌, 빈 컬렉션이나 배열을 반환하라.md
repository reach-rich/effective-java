## 1. 들어가기

대부분 다음과 같이 반환할 값이 없다면 null을 반환할 것입니다.

```java
  private final List<Cheese> cheesesInStock = ...;

  /**
   *  매장 안의 모든 치즈 목록을 반환한다.
   *  단, 재고가 없다면 null을 반환한다. 
   */
  public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
  }
```

잘못된 코드는 아니지만, null을 반환한다면 클라이언트 입장에서는 null을 처리하는 코드가 필요할 것입니다.

```java
  List<Cheese> cheeses = shop.getCheeses();
  if (cheeses != null && cheeses.contains(Cheese.STILTON)) {
    ...
  }
```

그러므로 컬렉션이나 배열과 같은 컨테이너가 비었을 때 null을 반환하는 메서드를 사용한다면

null 대신 빈 컬렉션이나 배열을 반환합시다.

## 2. 성능 면에서 null을 반환해야 한다는 주장에 대한 반론

그럼 이런 생각이 들 수 있습니다.

> 빈 컨테이너 할당하는데도 비용이 드니깐 차라리 null을 반환하는게 낫지 않아?

이 주장은 두 가지 면에서 틀린 주장입니다.

1. 빈 컬렉션 할당이 성능 저하의 주범이라고 확인되지 않는 한 이 정도 성능 차이는 신경 쓸 수준이 못된다.

2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.

## 3. 빈 컬렉션을 반환하는 방법

그럼 빈 컬렉션이나 배열을 어떻게 반환할 수 있을까요?

가장 전형적인 방법으로 대부분의 상황에서는 아래와 같이 사용할 수 있습니다.

```java
/* 빈 컬렉션 반환 */
  public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
  }

/* 빈 배열 반환 */
  public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
  }
```

하지만 이는 가능성은 작지만, 사용 패턴에 따라 빈 컬렉션 할당이 성능을 눈에 띄게 떨어뜨릴 수도 있습니다.

이럴 때는 매번 똑같은 빈 '불변' 컬렉션을 반환하는 방법을 사용할 수 있습니다.

불변 객체는 자유롭게 공유해도 안전하기 때문에 가능합니다.

```java
/* 빈 컬렉션 반환 */
  public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
  }

/* 빈 배열 반환 */
  private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

  public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
  }
```

## 4. 주의사항

단순히 성능을 개선할 목적이라면 toArray에 넘기는 배열을 미리 할당하지는 말아야 합니다.

미리 할당한다면 오히려 성능이 떨어진다는 연구 결과가 있습니다.

```java
  return cheesesInStock.toArray(new Cheese[cheeseInStock.size()]);
```

## 5. 정리

이번 포스트는 빈 컬렉션과 배열을 반환하는 방법에 대해 알아보았습니다.

만약, 컬렉션이나 배열과 같은 컨테이너를 반환하는 메서드를 사용하는데

비어있을 가능성이 있다면 null 대신 빈 컬렉션이나 배열을 반환합시다.