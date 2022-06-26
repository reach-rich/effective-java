## 1. 들어가기

상속은 객체 지향 프로그래밍의 3대 요소(캡슐화, 상속, 다형성)로 불릴 만큼 매우 중요하고 유용합니다.

많은 교육 과정에서 코드를 재사용하기 위해 상속을 권장하곤 합니다.

하지만 매번 상속을 하는 것이 최선이 아닌 경우가 있습니다.

어떤 경우에 상속하면 안될까요?

## 2. 클래스를 상속하면 안되는 경우

상위 클래스 설계자가 확장을 고려하지 않았거나, 문서화를 제대로 해두지 않은 경우

상위 클래스의 릴리스마다 내부 구현이 달라질 수 있고, 그 여파로 하위 클래스에서 오동작 할 수 있습니다.

하나의 예시를 들어보겠습니다.

```java
/* 상속의 잘못된 예시 */
  public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;       // 추가한 원소의 수

    public InstrumentedHashSet() {}
    
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

    public int getAddCount() { return addCount; }
  }
```

해당 예시의 클래스는 HashSet이 처음 생성된 이후 원소가 몇 개 더해졌는지 계측할 수 있는 클래스입니다.

언뜻 보기에는 잘 구현된 것처럼 보이네요!

아래의 클라이언트 코드를 추가하면 어떤 결과가 나올까요?

```java
/* Client */
  InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
  s.addAll(List.of("상훈", "진흥", "성재", "나현"));

  System.out.println(s.addCount);
```

대부분 4를 반환하리라 기대할 것입니다.

하지만 실제 결과는 8을 반환합니다.

왜 그럴까요?

HashSet 클래스가 상속받는 AbstractCollection 클래스의 addAll 메서드를 살펴보겠습니다.

```java
/* AbstractCollection 클래스의 addAll 메서드 */
  public boolean addAll(Collection<? extends E> c) {
    boolean modified = false;
    for (E e : c)
      if (add(e))   // add 메서드 사용
        modified = true;
    return modified;
  }
```

보다시피 addAll 내부에서 add 메서드를 호출하는 것을 볼 수 있습니다.

그렇기 때문에 addCount에 값이 한 요소당 2씩 더해져 최종값이 8로 반환된 것입니다.

이처럼 상위 클래스의 내부를 잘 알지 못한다면 잘못된 결과를 초래할 수 있습니다.

그럼 이를 어떻게 해결할 수 있을까요?

## 3. 해결 방법 추적

차근차근 쉬운 방법부터 생각해봅시다.

* 자신이 다른 부분을 사용하는 방법

   가장 쉬운 방법은 addAll 메서드를 재정의하지 않는 방법입니다.

   하지만 이는 addAll이 add를 이용해 구현했음을 가정한 해법이라는 한계가 있습니다.

* 다른 방법으로 재정의하는 방법

   addAll 메서드를 다른 식으로 재정의할 수도 있습니다.

   상위 클래스의 addAll 메서드를 호출해서 add 메서드를 사용하는 방법 대신
   
   addAll 메서드 내부에서 원소 하나 당 add 메서드 한 번을 호출하는 것입니다.

   ```java
   /* 새로 재정의한 addAll 메서드 */
     @Override
     public boolean addAll(Collection<? extends E> c) {
       boolean modified = false;
       for (E e : c)
         if (add(e))   // add 메서드 사용
           modified = true;
       return modified;
     }
   ```

   InstrumentedHashSet의 addAll 내부 구현은 AbstractCollection 클래스의 addAll과 동일하지만

   결과는 4를 반환하는 것을 볼 수 있습니다.

   addAll이 add를 사용하는지와 상관없이 옳은 결과를 반환한다는 점이 나은 해법이기는 하나

   이 방법은 시간이 더 들고 자칫 오류를 내거나 성능을 더 떨어뜨릴 수 있는 단점이 있습니다.

* 새로운 메서드를 추가하는 방법

   새로운 메서드를 추가하면 확실히 안전할 수 있습니다.

   하지만 만약, 다음 릴리스에서 상위 클래스에 새로운 메서드가 추가 됐는데,
   
   운 없게도 하위 클래스에 추가한 메서드의 시그니처가 같다면 컴파일조차 되지 않습니다.

> 그럼 방법이 없는걸까?

## 4. 컴포지션

다행히도 이상의 문제를 피해 가는 방법이 있습니다.

바로, 클래스를 컴포지션으로 만드는 방법인데요.

컴포지션은 기존 클래스가 새로운 클래스의 구성요소로 만들어 기존 클래스를 확장시키는 방법입니다.

상속이 IS-A 관계라면, 컴포지션은 HAS-A 관계라고 볼 수 있습니다.

그리고 새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드 호출을 통해 결과를 반환합니다.

이 방식은 전달(forwarding) 이라고 하며, 이러한 메서드들을 전달 메서드라 부릅니다.

## 5. 컴포지션을 활용한 해결 방법

컴포지션과 전달 방식을 통해 다시 예시를 작성해보겠습니다.

```java
/* ForwardingSet 클래스 - 전달 클래스 */
  public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s)                    { this.s = s; }

    public boolean add(E e)                           { return s.add(e); }
    public boolean addAll(Collection<? extends E> c)  { return s.addAll(c); }
    
    ...
  }
```

```java
/* InstrumentedSet 클래스 - 집합 클래스 자신 */
  public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s)        { super(s); }

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

    public int getAddCount() { return addCount; }

    ...
  }
```

집합 클래스를 위와 같이 작성한 후에 앞서 사용했던 클라이언트 코드로 테스트 해보겠습니다.

```java
/* Client */
  InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
  s.addAll(List.of("상훈", "진흥", "성재", "나현"));

  System.out.println(s.getAddCount());
```

예상 결과와 동일하게 4가 나오는 것을 볼 수 있습니다.

추가로 InstrumentedSet은 Set 인터페이스를 상속 받기 때문에 

어떠한 Set 인스턴스도 생성자의 매개변수로 받아 인스턴스 생성이 가능하기에 유연성이 증가했습니다.

이와 같이 다른 Set 인스턴스를 감싸고 있다는 뜻에서 InstrumentedSet 클래스를 래퍼 클래스라고 하고,

다른 Set에 계측 기능을 덧씌운다는 뜻에서 데코레이터 패턴이라고 합니다.

## 6. 정리

상속은 반드시 상위 클래스와 하위 클래스가 IS-A 관계인 경우에만 사용해야 합니다.

Stack나 Properties 라이브러리처럼 자바 플랫폼 라이브러리 중에도 잘못된 예시가 있습니다.

컴포지션을 써야 할 상황에서 상속을 사용한다면 내부 구현을 불필요하게 노출해

API가 내부 구현에 묶이고 클래스의 성능도 영원히 제한됩니다.

컴포지션 대신 상속을 사용하기로 결정하기 전, 

* 확장하려는 클래스의 API에 아무런 결함이 없는가?

* 결함이 있다면, 이 결함이 클래스의 API까지 전파돼도 괜찮은가?

라는 자문을 통해 다시 한번 생각해보도록 합시다.
