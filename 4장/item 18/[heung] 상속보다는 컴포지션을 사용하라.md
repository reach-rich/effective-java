Effective Java의 열여덟 번째 아이템 "상속보다는 컴포지션을 사용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 상속의 문제점

상속은 코드를 재사용하는 강력한 수단이지만, 항상 최선은 아니다. 잘못 사용하면 오류를 내기 쉬운 소프트웨어를 만들게 된다.

상위 클래스와 하위 클래스를 모두 같은 프로그래머가 통제하는 패키지 안에서라면 상속도 안전한 방법이다. 확장할 목적으로 설계되었고 문서화도 잘 된 클래스도 마찬가지로 안전하다. 하지만 일반적인 구체 클래스를 패키지 경계를 넘어, 즉 다른 패키지의 구체 클래스를 상속하는 일은 위험하다. 

**본문에서 말하는 '상속'은 클래스가 다른 클래스를 확장하는 구현 상속만을 말한다.**

<br>

### 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.

다르게 말하면, **상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.** 이러한 이유로 상위 클래스 설계자가 확장을 충분히 고려하고 문서화도 제대로 해두지 않으면 하위 클래스는 상위 클래스의 변화에 발맞춰 수정돼야만 한다.

다음은 상속을 잘못 사용한 예이다. 성능을 높이기 위해 HashSet이 처음 생성된 이후 원소가 몇 개 더해졌는지 알 수 있도록 한다. 

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
  
  // 추가된 원소의 수
  private int addCount = 0;
  
  public InstrumentedHashSet() {}
  
  public InstrumentedHashSet(int initCap, float, loadFactor) {
    super(initCap, loadFactor);
  }
  
  @Override
  public boolean add(E e) {
    addCount++;
    return super.add(e);
  }
  
  @Override
  public boolean addAll(Collection<? extends E> c) {
    addCount += c.size();
    return super.addAll(c);
  }
  
  public int getAddCount() {
    return addCount;
  }
}
```

이 클래스는 잘 구현된 것처럼 보이지만 제대로 작동하지 않는다. 이 클래스의 인스턴스에 addAll 메서드로 원소 3개를 더했다고 해보자. 이제 getAddCount 메서드를 호출하면 3을 반환하리라 기대하겠지만, 실제로는 6을 반환한다.

그 원인은 HashSet의 addAll 메서드가 add 메서드를 사용해 구현된 데 있다. addAll 메서드에서 3을 더하고 다시 add 메서드를 세번 호출하며 3을 더해 6이 된 것이다.

<br>

이 문제를 고치는 방법에는 세 가지가 있다. 하지만 또 다른 문제를 동반한다.

- **addAll 메서드르 재정의하지 않는다.**
  - 하지만 HashSet의 addAll이 add 메서드를 이용해 구현했음을 가정한 해법이라는 한계를 지닌다. 따라서 가정에 기댄 InstrumentedHashSet도 깨지기 쉽다.
- **주어진 컬렉션을 순회하며 원소 하나당 add 메서드를 한 번만 호출한다.**
  - 하지만 상위 클래스의 메서드 동작을 다시 구현하는 이 방시은 어렵고, 시간도 더 들고, 자칫 오류를 내거나 성능을 떨어뜨릴 수도 있다.
  - 하위 클래스에서는 접근할 수 없는 private 필드를 써야 하는 상황이라면 이 방식으로는 구현 자체가 불가능하다.
- **메서드를 재정의하는 대신 새로운 메서드로 추가한다.**
  - 다음 릴리스에서 상위 클래스에 새 메서드가 추가됐는데, 운 없게도 하필 하위 클래스에 추가한 메서드와 시그니처가 같고 반환 타입은 다르다면 컴파일조차 되지 않는다. 
  - 만약 반환 타입이 같다면 세 메서드를 재정의한 꼴이 되며, 상위 클래스의 메서드가 요구하는 규약을 만족하지 못할 가능성이 크다.

<br>

## 2. 컴포지션

다행히 위에서 제시한 문제를 모두 피해가는 묘안이 있다. **기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하자.** 기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 이러한 설계를 **컴포지션(composition)**이라 한다.

새 클래스의 인스턴스 메서드들은 기존 클래스에 대응하는 메서드를 호출해 그 결과를 반환한다. 이 방식을 **전달(forwarding)**이라 하며, 새 클래스의 메서드들을 **전달 메서드(forwarding method)**라 부른다. 그 결과 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 심지어 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향받지 않는다.

다음은 이전에 봤던 InstrumentedHashSet을 컴포지션과 전달 방식으로 다시 구현한 코드이다.

```java
// 래퍼 클래스
public class InstrumentedSet<E> extends ForwardingSet<E> {
  
  private int addCount = 0;
  
  public InstrumentedSet(Set<E> s) {
    super(s);
  }
  
  @Override
  public boolean add(E e) {
    addCount++;
    return super.add(e);
  }
  
  @Override
  public boolean addAll(Collection<? extends E> c) {
    addCount += c.size();
    return super.addAll(c);
  }
  
  public int getAddCount() {
    return addCount;
  }
}
```

```java
// 재사용할 수 있는 전달 클래스
public class ForwardingSet<E> implements Set<E> {
  
  private final Set<E> s;
  public ForwardingSet(Set<E> s) { this.s = s; }
  
  public void clear() { s.clear(); }
  public boolean contains(Object o) { return s.contains(o); }
  public boolean isEmpty() { return s.isEmpty(); }
  public int size() { return s.size(); }
  public Iterator<E> iterator() { return s.iterator(); }
  public boolean add(E e) { return s.add(e); }
  public boolean remove(Object o) { return s.remove(o); }
  public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
  public boolean addAll(Collection<? extends E> c) { return s.addAll(c); }
  public boolean removeAll(Collextion<?> c) { return s.removeAll(c); }
  public boolean retainAll(Collection<?> c) { return s.retainAll(c); }
  public Object[] toArray() { return s.toArray(); }
  public <T> T[] toArray(T[] a) { return s.toArray(a); }
  @Override public boolean equals(Object o) { return s.equals(o); }
  @Override public int hashCode() { return s.hashCode(); }
  @Override public String toString() { return s.toString(); }
}
```

InstrumentedSet은 HashSet의 모든 기능을 정의한 Set 인터페이스를 활용해 설계되어 견고하고 아주 유연하다. 임의의 Set에 계측 기능을 덧씌워 새로운 Set으로 만드는 것이 이 클래스의 핵심이다.

상속 방식은 구체 클래스 각가을 따로 확장해야 하며, 지원하고 싶은 상위 클래스의 생성자 각각에 대응하는 생성자를 별도로 정의해줘야 한다. 하지만 지금 선보인 컴포지션 방식은 **한 번만 구현해두면 어떠한 Set 구현체라도 계측할 수 있으며, 기존 생성자들과도 함께 사용할 수 있다.**

컴포지션과 전달의 조합은 넓은 의미로 **위임(delegation)**이라고 부른다. 단, 엄밀히 따지면 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우만 위임에 해당한다.

<br>

## 3. 래퍼 클래스

다른 Set 인스턴스를 감싸고(wrap) 있다는 뜻에서 InstrumentedSet 같은 클래스를 **래퍼 클래스**라 하며, 다른 Set에 계측 기능을 덧씌운다는 뜻에서 **데코레이터 패턴**이라고 한다.

래퍼 클래스는 담점이 거의 없다. 한가지, 래퍼 클래스가 콜백 프레임워크와는 어울리지 않는다는 점만 주의하면 된다. 콜백 프레임워크에서는 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출(콜백)때 사용하도록 한다. 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 대신 자신(this)의 참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체를 호출하게 된다. 이를 **SELF 문제**라고 한다.

전달 메서드들을 작성하는게 지루하겠지만, 재사용할 수 있는 전달 클래스를 인터페이스당 하나씩만 만들어두면 원하는 기능을 덧씌우는 래퍼 클래스들을 아주 손쉽게 구현할 수 있다.

<br>

## 4. 핵심 정리

- 상속은 강력하지만 캡슐화를 해친다는 문제가 있다.
- 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 써야한다.
- 그런데 is-a 관계일 때도 하위 클래스의 패키지가 상위 클래스와 다르고, 상위 클래스가 확장을 고려해 설계되지 않았다면 여전히 문제가 될 수 있다.
- 상속의 취약점을 피하려면 상속 대신 컴포지션과 전달을 사용하자.
- 특히 래퍼 클래스로 구현할 적당한 인터페이스가 있다면 컴포지션을 사용하자.

<br>

## 5. Related Posts

- 문서화 (Item 19)
