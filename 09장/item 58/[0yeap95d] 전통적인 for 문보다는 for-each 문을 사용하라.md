### 📝 전통적인 for 문보다는 for-each 문을 사용하라

<br>

**✏ #01 예제소스 | 컬렉션 순회하기 - 더 나은 방법이 있다**

```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ... // e로 무언가를 한다
}
```

<br>

**✏ #02 예제소스 | 배열 순회하기 - 더 나은 방법이 있다**

```java
for (int i = 0; i < a.length; i++) {
    ... // a[i]로 무언가를 한다
}
```

>실제 필요한건 원소이지만, 반복자와 인덱스 변수는 코드를 지저분하게 함
>
>오류의 가능성이 높아짐
>
>잘못된 변수를 사용했을 때 컴파일러가 잡아주리라는 보장이 없음
>
>컬렉션이나 배열이냐에 따라 코드 형태가 달라지므로 주의가 필요

<br>

✔ **for-each문으로 해결 가능하다!**

<br>

**✏ #03 예제소스 | 컬렉션과 배열을 순회하는 올바른 관용구**

```java
for (Element e : elements) {
	... // e로 무언가를 한다
}
```

> 반복대상이 컬렉션이든 배열이든 속도는 변하지 않음

<br>

**✏ #04 예제소스 | 중첩해 순회하는 경우 이점이 커짐**

``` java
enum Suit {CLUB, DIAMOND, HEART, SPADE}
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, ...}

static Collection<Suit> Arrays.asList(Suit.values());
static Collection<Rank> Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
        deck.add(new Card(i.next(), j.next()));
```

>바깥 컬렉션 `suits` 의 반복자에서 `next` 메서드가 너무 많이 불림
>
>마지막 줄의 `i.next()` 는 '숫자 `Suit` 하나당' 한 번씩만 불려야 하지만, 안쪽 반복문에서 호출되어 '카드 `Rank` 하나당' 한번씩 호출됨
>
>숫자가 바닥나면 `NoSuchElementException`을 던짐

<br>

**✏ #05 예제소스 | 같은 버그, 다른 증상**

```java
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }
...
Collection<Face> faces = EnumSet.allOf(Face.class);

for (Iterator<Face> i = faces.iterator(); i.hasNext(); )
    for (Iterator<Face> j = faces.iterator(); j.hasNext(); )
        System.out.println(i.next() + " " + j.next());
```

>예외를 던지지 않지만, 가능한 조합을 ("ONE ONE"부터 "SIX SIX"까지) 여섯 쌍만 출력

<br>

**✏ #06 예제소스 | 문제는 고쳤지만 보기 좋진 않음. 더 나은 방법이 있다**

```java
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
    Suit suit = i.next();
    for (Iterator<Rank> j = ranks.itertor(); j.hasNext(); )
        deck.add(new Card(suit, j.next()));
}
```

<br>

**✏ #07 예제소스 | 컬렉션이나 배열의 중첩 반복을 위한 권장 관용구**

```java
for (Suit suit : suits)
    for (Rank rank : ranks)
        deck.add(new Card(suit, rank));
```

<br>

---

<br>

### ❌ for-each를 사용할 수 없는 상황

**파괴적인 필터링**

- 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야함
- 자바 8부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있음

<br>

**변형**

- 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 함

<br>

**병렬 반복**

- 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 함

<br>

**✔ 세 가지 상황 중 하나에 속할 때는 일반적인 for 문을 사용하되 위의 문제들을 주의할 것**

<br>

**✏ #08 예제소스 | for-each 문은 Iterable 인터페이스를 구현 시 사용가능**

```java
public interface Iterable<E> {
    // 이 객체의 원소를 순회하는 반복자를 반환한다
    Iterator<E> iterator();
}
```

<br>

---

<br>

### 📌 핵심정리

**전통적인 for 문과 비교했을 때 for-each 문은 명료하고, 유연하고, 버그를 예방해준다**

**성능 저하도 없다**

**가능한 모든 곳에서 for 문이 아닌 for-each 문을 사용하자**

<br>

