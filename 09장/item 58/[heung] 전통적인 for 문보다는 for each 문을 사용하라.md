Effective Java의  58번째 아이템 "전통적인 for 문보다는 for-each 문을 사용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. for 문과 for-each 문

다음은 전통적인 for 문으로 컬렉션을 순회하는 코드다.

```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
  Element e = i.next();
  ... // e
}
```

```java
for (int i=0; i<a.lentgh; i++) {
  ... // a[i]
}
```

이 관용구들은 while 문보다는 낫지만 가장 좋은 방법은 아니다.

* 반복자와 인덱스 변수는 모두 코드를 지저분하게 할 뿐 우리에게 진짜 필요한 건 원소들뿐이다. 
* 쓰이는 요소 종류가 늘어나면 오류가 생길 가능성이 높아진다.
* 컬렉션이냐 배열이냐에 따라 코드 형태가 상당히 달라지므로 주의해야 한다. 

<br>

이상의 문제는 for-each 문을 사용하면 모두 해결된다. 

```java
for (Element e : elements) {
  ... // e로 무언가를 한다.
}
```

* 반복자와 인덱스 변수를 사용하지 않으니 코드가 깔끔해지고 오류가 날 일도 없다.
* 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있어서 어떤 컨테이너를 다루는지는 신경 쓰지 않아도 된다.
* for-each 문이 만들어내는 코드는 사람이 손으로 최적화한 것과 같다.

<br>

## 2. for-each 문의 장점 - 중첩된 컬렉션 순회

컬렉션을 중첩해 순회해야 한다면 for-each 문의 이점이 더욱 커진다. 다음 코드에서 버그를 찾아보자.

```java
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING }
...
static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayLis(t<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
  for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); ) {
    deck.add(new Card(i.next(), j.next()));
  }
}
```

문제는 바깥 컬렉션(suits)의 반복자에서 next 메서드가 너무 많이 불린다는 것이다. 숫자가 바닥나면 반복문에서 NoSuchElementException을 던진다. 

심지어 바깥 컬렉션의 크기가 안쪽 컬렉션의 크기의 배수라면 이 반복문은 잘못된 동작을 수행한 후 예외를 던지지 않고 종료한다. 아래 코드를 보자.

```java
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }
...
Collection<Face> faces = EnumSet.allOf(face.class);

for (Iterator<Face> i = faces.iterator(); i.hasNext(); ) 
  for (Iterator<Face> j = faces.iterator(); j.hasNext(); ) 
    System.out.println(i.next() + " " + j.next());
```

이 프로그램은 예외를 던지지 않지만, 가능한 조합을 단 여섯 쌍만 출력하고 끝나버린다. 이 문제를 해결하려면 바깥 반복문에 바깥 원소를 저장하는 변수를 하나 추가해야 한다. 

for-each 문을 중첩하는 것으로 이 문제는 간단하게 해결된다. 심지어 코드도 간결해진다.

```java
for (Suit suit : suits)
  for (Rank rank : ranks)
    deck.add(new Card(suit, rank));
```

<br>

## 3. for-each를 사용할 수 없는 상황

안타깝게도 for-each 문을 사용할 수 없는 상황이 세 가지 존재한다. 

* 파괴적인 필터링 : 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 한다.
* 변형 : 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.
* 병렬 반복 : 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다. 

세 가지 상황 중 하나에 속할 때는 일반적인 for 문을 사용하되 이번 아이템에서 언급한 문제들을 경계해야 한다.

<br>

## 4. 핵심 정리

* 전통적인 for 문과 비교했을 때 for-each 문은 명료하고, 유연하고, 버그를 예방해준다. 성능 저하도 없다.
* 가능한 모든 곳에서 for 문이 아닌 for-each 문을 사용하자.
