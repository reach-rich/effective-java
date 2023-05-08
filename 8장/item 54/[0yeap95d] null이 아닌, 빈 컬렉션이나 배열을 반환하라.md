### 💡 null이 아닌, 빈 컬렉션이나 배열을 반환하라

<br>

**✏ #01 예제소스 | 컬렉션이 비었으면 null을 반환한다 - 따라 하지 말 것!**

```java
private final List<Cheese> cheeseInStock = ...;

/**
 * @return 매장 안의 모든 치즈 목록을 반환한다
 *     단, 재고가 하나도 없다면 null을 반환한다
 */
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null
        : new ArrayList<>(cheesesInStock);
}
```

>`null`을 반환한다면, 클라이언트는 이 `null `상황을 처리하는 코드를 추가로 작성해야 함
>
>```java
>List<Cheese> cheeses = shop.getCheeses();
>if (cheeses != null && cheeses.contains(CHeese.STILTON))
>    System.out.println("좋았어, 바로 그거야.");
>```

<br>

- 컬렉션이나 배열 같은 컨테이너가 비었을 때 `null`을 반환하는 메서드를 사용할 때면 항시 이와 같은 방어 코드 필요

- 클라이언트에서 방어 코드를 빼먹으면 오류가 발생할 수 있음

- 실제로 객체가 0개일 가능성이 거의 없는 상황에서는 수년 뒤에야 오류가 발생하기도 함

- 한편, `null`을 반환하려면 반환하는 쪽에서도 이 상황을 특별히 취급해줘야 해서 코드가 더 복잡해짐

<br>

**🔎 때로는 빈 컨테이너를 할당하는 데도 비용이 드니 null을 반환하는 쪽이 낫다는 주장도 있음**

이는 두 가지 면에서 틀린 주장

1. 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한, 이 정도의 성능 차이는 신경 쓸 수준이 못 된다
2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다

<br>

**✏ #02 예제소스 | 빈 컬렉션을 반환하는 올바른 예**

```java
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}
```

>사용 패턴에 따라 빈 컬렉션 할당이 성능을 눈에 띄게 떨어뜨릴 수도 있지만, 매번 똑같은 빈 '불변' 컬렉션을 반환하는 것으로 해결 가능
>
>불변객체는 자유롭게 공유해도 안전

<br>

- 다음 코드에서 사용하는 `Collections.emptyList` 메서드가 그러한 예시
- 집합이 필요하면 `Collections.emptySet`을, 맵이 필요하면 `Collections.emptyMap`을 사용 

<br>

**✔ 단, 이 역시 최적화에 해당하니 꼭 필요할 때만 사용하자**

<br>

**✏ #03 예제소스 | 최적화-빈컬렉션을 매번 새로 할당하지 않도록 했다**

```java
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList()
        : new ArrayList<>(cheesesInStock);
}
```

> 배열을 쓸 때도 마찬가지로 절대 `null`을 반환하지 말고 길이가 0인 배열을 반환
>
> 보통은 단순히 정확한 길이의 배열을 반환(그 길이가 0일 수도 있을 뿐)

<br>

**✏ #04 예제소스 | 길이가 0일 수도 있는 배열을 반환하는 올바른 방법**

```java
public Cheese[] getCheeses() {
	return cheesesInStock.toArray(new Cheese[0]);
}
```

> `toArray` 메서드에 건넨 길이 0짜리 배열은 우리가 원하는 반환 타입(이 경우엔 `Cheese[]`)을 알려주는 역할
>
> 이 방식이 성능을 떨어뜨릴 것 같다면, 아래의 방법을 참고

<br>

**✏ #05 예제소스 | 최적화-빈 배열을 매번 새로 할당하지 않도록 했다**

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
	return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

>길이 0인  배열을 미리 선언해두고 매번 그 배열을 반환 (길이 0인 배열은 모두 불변)
>
>`getCheeses`는 항상 `EMPTY_CHEESE_ARRAY`를 인수로 넘겨 `toArray`를 호출
>
>`cheesesInStock`이 비어있을 때면 언제나 `EMPTY_CHEESE_ARRAY`를 반환
>
>단순 성능 개선을 위한 목적이라면 `toArray`에 넘기는 배열을 미리 할당하는 건 추천하지 않음 (오히려 성능이 떨어진다는 연구 결과)

<br>

**✏ #06 예제소스 | 나쁜 예 - 배열을 미리 할당하면 성능이 나빠진다**

```java
return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
```

<br>

---

### 📌 핵심정리

**null이 아닌, 빈 배열이나 컬렉션을 반환하라**

**null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다**

**그렇다고 성능이 좋은 것도 아니다**



**끝.**
