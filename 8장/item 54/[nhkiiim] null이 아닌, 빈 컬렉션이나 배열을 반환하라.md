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


> 빈 컨테이너를 할당하는 데 비용이 들기 때문에 null을 반환하는 쪽이 낫다는 주장도 있지만 틀린 주장이다!
