# 상속보다는 컴포지션을 사용하라

### 1. 상속이 문제가 되는 경우
- 상속은 코드를 재사용하는 강력한 수단이지만, 항상 최선은 아님!
- 잘못 사용하면 오류를 내기 쉬운 소프트웨어를 만든다
- 상위 클래스와 하위 클래스를 모두 같은 프로그래머가 통제하는 패키지 안이라면 상속은 안전한 방법이다
- 확장할 목적으로 설계되고 문서화도 잘 된 클래스도 안전하다
- 하지만 일반적인 구체 클래스 패키지 경계를 넘어~ 다른 패키지의 구체 클래스를 상속하는 일은 위험하다!
- 이번 파트에서는 클래스가 다른 클래스를 확장하는 구현 상속에 대한 이야기임을 명심하고 들어가보자!

#
### 2. 상속은 캡슐화를 깨뜨려요
- 메서드 호출과 다르게 상속은 캡슐화를 깨뜨린다
- 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다!
- 상위 클래스는 릴리스마다 구현이 달라질 수 있어서 갑자기 가마니 있던 하위 클래스에 문제가 생기는 것!
- 이런 이유로 상위 클래스 설계자가 확장을 충분히 고려하고 문서화도 제대로 하지 않으먄 하위는 맨날 상위 따라 수정 해야함

__1) HashSet을 사용하는 프로그램으로 알아보자__
```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    public InstrumentedHashSet(){
    }

    public InstrumentedHashSet(int initCap, float loadFactor){
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
- 성능을 높이기 위해 HashSet은 처음 생성된 루 원소가 몇개 더해졌는지 알아야함
- 그런 메서드를 구현한 예시이다
- 이 클래스는 제대로 작동하지 않는다! -> HashSet의 addAll 메서드가 add 메서드를 사용해 만들어져 중복으로 더해짐!

- 하위 클래스에서 addAll 메서드를 재정의 하지 않으면 문제는 고쳐진다!
- 하지만 당장에는 제대로 동작할지라도 어케될지 머른다!
- 암튼 문제다 문제!

__2) 다음 릴리즈에서 상위 클래스에  새로운 메서드를 추가하는 예시로 알아보자__
- 보안 때문에 컬렉션에 추가된 모든 원소가 특정 조건을 만족해야하는 경우
- 메서드 재정의로 인한 문제 발생 가능

#
### 3. 이런 문제를 해결하는 방법 ! 컴포지션 구성
- 기존 클래스를 확장하는 대신 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하자
- 기존 클래스가 새로운 클래스의 구성요소로 쓰이는 컴포지션 구성
- 새클래스의 메서드들을 전달 메서드라고 하며 기존 클래스에 대응하는 메서드를 호출해 그 결과를 반환한다
- 기존 클래스에 새로운메서드가 추가돼도 전혀 영향을 받지 않는다

__ex) 재사용 할 수 있는 전달 클래스__
```java
public class ForwardingSet<E> implements Set<E> {
  private final Set<E> s;

  public ForwardingSet(Set<E> s) {
    this.s = s;
  }

  public void clear() {
    s.clear();
  }

  public boolean contains(Object o) {
    return s.contains(o);
  }

  public boolean isEmpty() {
    return s.isEmpty();
  }

  public int size() {
    return s.size();
  }

  public Iterator<E> iterator() {
    return s.iterator();
  }

  public boolean add(E e) {
    return s.add(e);
  }

  public boolean remove(Object o) {
    return s.remove(o);
  }

  public boolean containsAll(Collection<?> c) {
    return s.containsAll(c);
  }

  public boolean addAll(Collection<? extends E> c) {
    return s.addAll(c);
  }

  public boolean removeAll(Collection<?> c) {
    return s.removeAll(c);
  }

  public boolean retainAll(Collection<?> c) {
    return s.retainAll(c);
  }

  public Object[] toArray() {
    return s.toArray();
  }

  public <T> T[] toArray(T[] a) {
    return s.toArray(a);
  }

  @Override
  public boolean equals(Object o) {
    return s.equals(o);
  }

  @Override
  public int hashCode() {
    return s.hashCode();
  }

  @Override
  public String toString() {
    return s.toString();
  }
}
```
- HashSet의 모든 기능을 정의하고 있는 Set 인터페이스를 활용해 설계
- 임의의 Set에 계측 기능을 덧씌워 새로운 Set으로 만드는 것이 이 클래스의 핵심!
- 다른 Set 인스턴스를 감싸고 있다는 뜻에서 래퍼 클래스, 데코레이터 패턴 이라고도 한다
- 컴포지션과 전달의 조합은 넓은 의미로 위임이라고 하는데 엄밀히 따지면 내부 객체에 자기 자신의 참조를 넘기는 경우에만 위임이라고 함!

#
### 4. 래퍼클래스의 단점
- 거의 없지만 딱하나 있지
- 래퍼 클래스는 콜백 프레임워크와 어울리지 않음


