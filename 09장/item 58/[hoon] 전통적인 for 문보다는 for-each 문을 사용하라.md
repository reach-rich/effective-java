## 1. 들어가기

어떤 컬렉션이나 배열을 순회하기 위해서는 다음과 같이 반복문을 통해서 구현할 수 있습니다.

```java
   for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
      Element e = i.next();

      ...
   }
```

```java
   for (int i = 0; i < a.length; i++) {
      Element e = a[i];

      ...
   }
```

이 관용구들은 while 문보다 낫지만 가장 좋은 방법은 아닙니다.

그 이유는 필요하지도 않은 반복자와 인덱스 변수를 씀으로써 코드를 지저분하게 만들기 때문입니다.

이런 문제는 어떻게 해결할 수 있을까요?

## 2. for-each 문

위와 같은 문제는 for-each 문을 통해 깔끔하게 해결할 수 있습니다.

```java
   for (Element e : elements) {
      ...
   }
```

for-each 문을 사용하면 반복자 또는 인덱스 변수가 없어 전보다 훨씬 깔끔해진 것을 확인할 수 있습니다.

이러한 for-each 문은 컬렉션을 중첩 순회하는 경우에 진가를 발휘합니다.

```java
   enum Suit {CLUB, DIAMOND, HEART, SPADE}
   enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, ...}

   static Collection<Suit> Arrays.asList(Suit.values());
   static Collection<Rank> Arrays.asList(Rank.values());

   List<Card> deck = new ArrayList<>();
   for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
      for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
         deck.add(new Card(i.next(), j.next()));
```

위 예시는 반복자를 사용한 일반 for 문의 잘못된 예시입니다.

`next()`는 숫자 하나당 한 번씩 호출되어야 하는데 카드 하나 당 한 번씩 불리고 있습니다. 

그래서 숫자가 바닥나면 `NoSuchElementException` 예외가 발생합니다.

조금 더 보완을 하면 아래와 같이 작성할 수 있지만 아직도 깔끔해보이진 않습니다.

```java
   for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
      Suit suit = i.next();
      for (Iterator<Rank> j = ranks.itertor(); j.hasNext(); )
         deck.add(new Card(suit, j.next()));
   }
```

그럼 해당 코드를 for-each 문을 통해 깔끔하게 변경해보겠습니다.

```java
   for (Suit suit : suits)
      for (Rank rank : ranks)
         deck.add(new Card(suit, rank));
```

단지 일반 for 문을 for-each 문으로 변경했을 뿐인데 눈에 띄게 간결해진 것을 볼 수 있습니다.

## 3. for-each 문을 사용할 수 없는 상황

🍀 **파괴적인 필터링**

   * 컬렉션을 순회하면서 동시에 원소를 제거하는 경우

   * Collection의 removeIf 메서드로 명시적 순회 회피 가능

🍀 **변형**

   * 컬렉션 또는 배열을 순회하면서 원소의 값 일부 혹은 전체를 교체하는 경우

🍀 **병렬 반복**

   * 여러 컬렉션을 병렬로 순회하는 경우

## 4. for-each 문 주의사항

for-each 문은 Iterable 인터페이스를 구현한 객체만 사용할 수 있습니다.

따라서 원소들의 묶음을 표현하는 타입을 작성하는 경우, iterable을 구현하는 쪽으로 고민해봅시다.

## 5. 정리

이번 포스트는 일반 for 문의 복잡성을 해결할 수 있는 for-each 문에 대해 알아보았습니다.

for-each 문은 명료하고 유연하며 버그를 사전에 예방해주는 장점이 있습니다.

그러므로 가능한 일반 for 문 대신 for-each 문을 사용합시다.