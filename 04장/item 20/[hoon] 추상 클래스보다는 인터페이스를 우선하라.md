## 1. 들어가기

이번 item은 '추상 클래스보다는 인터페이스를 우선하라' 입니다.

들어가기 앞서 추상 클래스와 인터페이스는 무엇이고 차이점은 무엇일까요?

이런 질문은 기술 면접 질문으로도 종종 나오는데요.

그럼 이 둘의 특징과 차이점을 한번 알아보도록 하겠습니다.

## 2. 추상 클래스와 인터페이스

먼저, 추상 클래스와 인터페이스는 무엇일까요?

* 추상 클래스

  추상 클래스는 말 그대로 클래스입니다. 

  그래서 일반 클래스처럼 필드 또는 메서드를 가질 수 있습니다. 

  하지만, 일반적인 클래스와의 차이점은 추상 메서드를 선언해서 

  상속을 통해 하위 클래스에서 완성하도록 유도하는 클래스이므로
  
  일반적인 클래스와는 다르게 객체를 생성할 수 없습니다.

* 인터페이스

  인터페이스는 추상 클래스처럼 다른 클래스를 작성하는데 도움을 주는 목적을 가지고 있습니다.

  비록, 클래스는 아니지만 인터페이스 또한 필드와 메서드를 가질 수 있고, 객체를 생성할 수 없습니다.

추상 클래스와 인터페이스는 매우 비슷한 것으로 보입니다.

그럼 이런 질문을 할 수 있습니다.

> 굳이 인터페이스를 사용하는 이유가 무엇인가요?

이번에는 추상 클래스와 인터페이스의 차이점을 알아보도록 하겠습니다.

* 다중 상속 여부

  추상 클래스는 단일 상속만 가능하고, 인터페이스는 다중 상속이 가능합니다.

* 사용 의도

  주로 추상 클래스는 "IS-A 관계" 에서 사용하고, 인터페이스는 "HAS-A 관계" 에서 사용합니다.

  예를 들어 클래스의 구분은 추상 클래스, 그 클래스의 특징은 인터페이스로 구현합니다.

  <div class="div-post-img">
    <img src="https://ss-hoon.github.io/assets/img/effective_java/item20/interface_abstract.png" width="80%" height="40%" /><br>
    추상 클래스와 인터페이스<br>
    [ 출처 : https://myjamong.tistory.com/150 ]
  </div>

## 3. 인터페이스의 장점

본론으로 넘어와서 왜 이번 item에서는 추상 클래스보다는 인터페이스를 우선하라는 걸까요?

* 인터페이스는 믹스인 정의에 안성맞춤이다.

  믹스인이란, 클래스가 구현할 수 있는 타입으로 
  
  클래스의 주된 타입 외에 특정 선택적 행위를 제공한다고 선언하는 효과를 줍니다.

  이렇게만 봐서는 정확히 어떤 의미인지 알기 어렵네요.

  그래서 한 가지 예시로 Comparable 인터페이스를 들 수 있습니다.
  
  Comparable을 구현한 클래스의 인스턴스들끼리는 순서를 정할 수 있다고 선언하는 것이기에

  Comparable 인터페이스를 믹스인 인터페이스라고 할 수 있습니다.

  클래스는 단일 상속만 가능하고, 클래스의 계층구조에서 믹스인을 삽입하기 위한 위치가 없기 때문에

  추상 클래스로는 믹스인을 정의할 수 없습니다.

* 계층구조가 없는 타입 프레임워크를 만들 수 있다.

  예를 들어 가수 인터페이스와 작곡가 인터페이스가 있습니다.

  ```java
    public interface Singer {}
  ```

  ```java
    public interface Songwriter {}
  ```

  하지만 대표적인 싱어송라이터인 아이유는 가수이지만 작곡도 합니다.

  ```java
    public interface SingerSongwriter extends Singer, Songwriter {}
  ```
  추상 클래스는 다중 상속이 되지 않아 둘을 확장하는데 제약이 있는 반면, 

  인터페이스는 다중 상속이 허용되기 때문에 마음껏 확장할 수 있는 장점이 있습니다.

* 디폴트 메서드가 추가되었다.

  자바 8 이상부터는 인터페이스에서도 구현 메서드를 작성할 수 있게 되었습니다.

  이를 디폴트 메서드라고 하는데 인터페이스의 메서드 중 구현 방법이 명백한 것이 있다면

  그 구현을 디폴트 메서드로 제공해 프로그래머의 일감을 덜어줄 수 있습니다.

## 4. 골격 구현 클래스

한편, 인터페이스와 추상 클래스의 장점을 모두 취하는 방법도 있습니다.

인터페이스와 함께 추상 골격 구현 클래스를 함께 제공하는 방법인데요.

인터페이스로는 타입과 디폴트 메서드를 정의하고, 

골격 구현 클래스에서 나머지 메서드까지 구현하는 방법입니다.

이렇게 해두면 단순히 골격 구현의 확장으로 인터페이스를 구현하는 데 필요한 일이 대부분 완료됩니다.

그리고 이러한 패턴을 디자인 패턴에서는 템플릿 메서드 패턴이라고 합니다.

관례상 인터페이스의 이름이 Interface라면, 골격 구현 클래스의 이름은 AbstractInterface로 짓습니다.

하나의 예시로 Collection 인터페이스의 골격 구현 클래스 이름은 AbstractCollection입니다.

## 5. 골격 구현 클래스 작성 방법

골격 구현 클래스는 어떻게 작성할 수 있을까요?

1. 인터페이스를 잘 살펴 다른 메서드들의 구현에 사용되는 기반 메서드를 선정한다.

   이 기반 메서드들은 골격 구현에서 추상 메서드가 될 것입니다.

2. 기반 메서드들을 사용해 직접 구현할 수 있는 메서드를 디폴트 메서드로 제공한다.

   만약, 모든 메서드가 기반 메서드와 디폴트 메서드가 된다면 골격 구현 클래스를 만들 이유가 없습니다.

   추가로 equals나 hashCode 같은 Object 메서드들은 디폴트 메서드로 제공하면 안됩니다.

## 6. 골격 구현 클래스 예시

골격 구현 클래스 작성 방법을 토대로 만든 예시로 Map.Entry 인터페이스를 살펴보겠습니다.

```java
/* Map.Entry 인터페이스 */
  interface Entry<K, V> {
    // 기반 메서드
    K getKey();
    V getValue();
    V setValue(V value);

    // Object 메서드
    /**
      * Compares the specified object with this entry for equality.
      * Returns {@code true} if the given object is also a map entry and
      * the two entries represent the same mapping.  More formally, two
      * entries {@code e1} and {@code e2} represent the same mapping
      * if<pre>
      *     (e1.getKey()==null ?
      *      e2.getKey()==null : e1.getKey().equals(e2.getKey()))  &amp;&amp;
      *     (e1.getValue()==null ?
      *      e2.getValue()==null : e1.getValue().equals(e2.getValue()))
      * </pre>
      * This ensures that the {@code equals} method works properly across
      * different implementations of the {@code Map.Entry} interface.
      *
      * @param o object to be compared for equality with this map entry
      * @return {@code true} if the specified object is equal to this map
      *         entry
      */
    boolean equals(Object o);

    /**
      * Returns the hash code value for this map entry.  The hash code
      * of a map entry {@code e} is defined to be: <pre>
      *     (e.getKey()==null   ? 0 : e.getKey().hashCode()) ^
      *     (e.getValue()==null ? 0 : e.getValue().hashCode())
      * </pre>
      * This ensures that {@code e1.equals(e2)} implies that
      * {@code e1.hashCode()==e2.hashCode()} for any two Entries
      * {@code e1} and {@code e2}, as required by the general
      * contract of {@code Object.hashCode}.
      *
      * @return the hash code value for this map entry
      * @see Object#hashCode()
      * @see Object#equals(Object)
      * @see #equals(Object)
      */
    int hashCode();
  }
```

getKey와 getValue는 확실히 기반 메서드이고, 선택적으로 setValue도 포함할 수 있습니다.

이 인터페이스는 equals와 hashCode의 동작 방식도 정의해놨습니다.

Object 메서드는 디폴트 메서드로 제공하지 말아야 하므로 해당 메서드는 골격 구현 클래스에서 구현합니다.

```java
/* Map.Entry 인터페이스의 골격 구현 클래스 */
  public abstract class AbstractMapEntry<K, V> implements Map.Entry<K, V> {
    @Override
    public V setValue(V value) {
      throw new UnsupportedOperationException();
    }

    // Map.Entry.equals 일반 규약 정의
    @Override public boolean equals(Object o) {
        if (o == this)
            return true;

        if (!(o instanceof Map.Entry))
            return false;

        Map.Entry<?,?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(), getKey()) && Objects.equals(e.getValue(), getValue());
    }

    // Map.Entry.hashCode 일반 규약 정의
    @Override public int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }

    @Override public String toString() {
        return getKey() + "=" + getValue();
    }
  }
```

## 7. 단순 구현

단순 구현은 골격 구현의 작은 변종으로 AbstractMap.SimpleEntry가 좋은 예입니다.

```java
  public static class SimpleEntry<K,V> implements Entry<K,V>, java.io.Serializable {
    private final K key;
    private V value;

    public SimpleEntry(K key, V value) {
      this.key   = key;
      this.value = value;
    }

    public K getKey() {
      return key;
    }

    public V getValue() {
      return value;
    }

    public V setValue(V value) {
        V oldValue = this.value;
        this.value = value;
        return oldValue;
    }

    public boolean equals(Object o) {
      if (!(o instanceof Map.Entry))
          return false;
      Map.Entry<?,?> e = (Map.Entry<?,?>)o;
      return eq(key, e.getKey()) && eq(value, e.getValue());
    }

    public int hashCode() {
      return (key   == null ? 0 :   key.hashCode()) ^
              (value == null ? 0 : value.hashCode());
    }
  }
```

단순 구현도 추상 클래스가 아니라는 점이 골격 구현 클래스와 다릅니다.

이러한 단순 구현은 그대로 써도 되고 필요에 맞게 확장해도 됩니다.

## 8. 정리

이번 포스트에서는 추상 클래스와 인터페이스, 골격 구현 클래스에 대해 알아보았습니다.

일반적으로 다중 구현용 타입으로는 인터페이스를 사용하는 것이 가장 적합합니다.

복잡한 인터페이스라면 골격 구현과 함께 제공해봅시다.

## 9. 참조

* [https://myjamong.tistory.com/150](https://myjamong.tistory.com/150)

* [https://yhmane.tistory.com/182](https://yhmane.tistory.com/182)