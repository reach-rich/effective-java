# null이 아닌, 빈 컬렉션이나 배열을 반환하라

> 주변에서 흔히 볼 수 있는 메서드 이지만, 재고가 없다고 해서 특별히 취급할 이우는 없다.
```java
public List<Cheese> getCheeses() {
  return cheeseInStock.isEmpty() ? null : new ArrayList<>(cheeseInStock);
}
```

> 그럼에도 위의 코드처럼 null을 반환한다면, 클라이언드는 null 상황을 처리하는 코드를 추가로 작성해야 한다.

```java
List<Cheese> getCheese = shop.getCheese();
if (cheeses ! null && cheese.contains(Cheese.STILTON)) {
  System.out.println("조아조아");
}
```
- 컬렉션이나 배열 같은 컨테이너가 비었을 때 null을 반환하는 메서드를 사용하면 방어 코드 추가 필여
- 클라이언트에서 방어 코드를 뺴먹으면 오류 발생 가능

#
### 주의사항

> 빈 컨테이너를 할당하는 데 비용이 들기 때문에 null을 반환하는 쪽이 낫다는 주장도 있지만 틀린 주장이다!

#### 1) 빈 컨테이너의 할당이 성능 저하의 주범이라고 확인되지 않는 한 성능차이 크지 않다!

#### 2) 빈 컬렉션과 배열은 굳이 새로 할당하지 않고 반환 가능하다

> 빈 컬렉션을 반환하는 전형적인 코드

```java
public List<Cheese> getCheese() {
  return new ArrayList<>(cheeseInStock);
}
```

#
### 빈 컬렉션 반환 방법
> 가능성은 작지만, 사용 패턴에 따라 빈 컬레션 할당이 성능에 영향을 미칠 수 있으나, 해법은 간단

#### 1) 매번 똑같은 빈 불변 컬렉션 반환
- 불변 객체는 자유롭게 공유해도 안전
```java
public List<Cheese> getCheese() {
  return cheeseInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheeseInStock);
}
```

#### 2) 배열인 경우 길이가 0인 배열 반환
```java
public Cheese[] getCheeses() {
  return cheeseInStock.toArray(new Cheese[0]);
}
- 이 방식이 성능을 떨어뜨릴 것 같으면 0짜리 배열을 미리 선언해두고 배먼 그 배열을 반환하면 OK
- 길이가 0인 배열은 모두 불변

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
  return CheeseInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```
- 이 최적화 버전의 getCheeses는 항상 EMPTY_CHEESE_ARRAY를 인수로 넘겨 toArray 호충
- CheeseInStock이 비었을 경우 항상 EMPTY_CHEESE_ARRAY를 qksghks
- 단순히 성능 개선의 목적으로는 toArrayd에 넘기는 배열을 미리 할당하는 것을 추천하지 않음 (성능이 떨어진다는 연구 경과도 있음 ㅠㅁㅜ)
