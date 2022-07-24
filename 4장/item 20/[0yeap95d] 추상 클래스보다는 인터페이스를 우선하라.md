### 🔎 다중 구현 메커니즘

**: 다중 구현 메커니즘은 '인터페이스'와 '추상클래스' 두 가지가 있다**

<br>

- 자바 8부터는 인터페이스도 디폴트 메서드 제공 가능해 두 메커니즘 모두 인스턴스 메서드를 구현 형태로 제공할 수 있다

- 추상클래스가 정의한 타입을 구현하는 클래스는 반드시 추상클래스의 하위 클래스가 되어야 한다는 점에서 차이가 있다

- 자바는 단일 상속만 지원하므로 추상클래스 방식은 새로운 타입을 정의하는 데 커다란 제약이 존재

- 인터페이스가 선언한 메서드를 모두 정의하고 그 일반 규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속했든 같은 타입으로 취급한다

<br>

---

<br>

### ⚙ 인터페이스의 장점

**기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다**

<br>

- 인터페이스가 요구하는 메서드를 추가하고, 클래스 선언에 `implements` 구문을 추가

- 기존 클래스 위에 새로운 추상 클래스를 끼워넣기는 어렵다

  > 두 클래스가 같은 추상클래스를 확장하길 원한다면, 그 추상 클래스는 계층구조상 두 클래스의 공통 조상이어야 한다

- 인터페이스는 믹스인(mixin) 정의에 안성맞춤이다

  > 믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다
  >
  > `Comparable`은 자신을 구현한 클래스의 인스턴스들끼리의 순서를 정할 수 있다고 선언하는 믹스 인터페이스이다

- 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다

  **✏ #01 예제소스 | 계층을 엄격히 구분하기 어려운 개념**

  ```java
  public interface Singer {
      AudioClip sing(Song s);
  }
  
  public interface Songwriter {
      Song compose(int chartPosition);
  }
  ```

  ```java
  public interface SingerSongwriter extends Singer, Songwriter {
      AudioClip strum();
      void actSensitive();
  }
  ```

  > `Singer`와 `Songwriter` 모두를 구현해도 전혀 문제되지 않는다
  >
  > 두 개 모두를 확장하고 새로운 메서드까지 추가한 제 3의 인터페이스를 정의할 수 있다

- 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다

<br>

---

<br>

### 📗 템플릿 메서드 패턴

**: 인터페이스와 추상 골격 구현 클래스의 장점을 모두 취하는 방법**

- 인터페이스로 타입을 정의하고, 필요하면 디폴트 메서드 몇 개도 함께 제공한다
- 관례상 인터페이스 이름이 `Interface`라면 골격 구현 클래스는 `AbstractInterface`로 짓는다

<br>

**✏ #02 예제소스 | 골격 구현을 사용해 완성한 구체 클래스**

```java
static List<Integer> intArrayAsList(int[] a) {
    Objects.requireNonNull(a);
    
    return new AbstractList<>() {
        @Override public Integer get(int i) {
            return a[i];
        }
        
        @Override public Integer set(int i, Integer val) {
            int oldVal = a[i];
            a[i] = val;
            return oldVal;
        }
        
        @Override public int size() {
            return a.length;
        }
    };
}
```

> `int` 배열을 받아 `Integer`인스턴스의 리스트 형태로 보여주는 어댑터
>
> `int`값과 `Integer` 인스턴스 사이의 변환 때문에 성능은 그리 좋지 않다
>
> 익명 클래스 형태를 사용했음에 주목

<br>

- 골격 구현 클래스는 추상클래스처럼 구현을 도와주는 동시에 추상클래스로 타입을 정의할 때 따라오는 심각한 제약에서도 자유롭다

- 구조상 골격 구현을 확장하지 못하는 처지라면 인터페이스를 직접 구현해야 한다

  이런 경우라도 인터페이스가 직접 제공하는 디폴트 메서드의 이점을 여전히 누릴 수 있다

- 골격 구현 클래스를 우회적으로 사용할 수도 있다

  > 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 `private` 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달하는 것 

<br>

---

<br>

### 📝 골격 구현 작성

- 인터페이스를 잘 살펴 다른 메서드들의 구현에 사용되는 기반 메서드들을 선정

  *이 기반 메서드들은 골격 구현에서는 추상 메서드가 될 것이다*

- 기반 메서드들을 사용해 직접 구현할 수 있는 메서드를 모두 디폴트 메서드로 제공한다

  단, `equals`와 `hashCode` 같은 `Object`의 메서드는 디폴트 메서드로 제공하면 안된다

- 인터페이스의 메서드 모두가 기반메서드와 디폴트메서드가 된다면 골격 구현 클래스를 별도로 만들 이유는 없다

- 기반 메서드나 디폴트 메서드로 만들지 못한 메서드가 남아 있다면, 이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어 남은 메서드들을 작성해 넣는다

- 골격 구현 클래스에는 필요하면 `public`이 아닌 필드와 메서드를 추가해도 된다

<br>

**✏ #03 예제소스 | 골격 구현 클래스**

```java
public abstract class AbstractMapEntry<K, V> implements Map.Entry<K, V> {
    @Override public V setValue(V value) {
        throw new UnsupportedOperationException();
    }
    
    @Override public boolean equals(Object o) {
		if (o == this) return true;
        if (!(o instanceof Map.Entry)) return false;
        Map.Entry<?,?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(), getKey()) && Objects.equals(e.getValue(), getValue());
    }
    
    @Override public int hashCode() {
		return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }
    
    @Override public String toString() {
		return getKey() + "=" + getValue();
    }
}
```

>`Map.Entry` 인터페이스나 그 하위 인터페이스로는 이 골격 구현을 제공할 수 없다
>
>디폴트메서드는 `eqauls`, `hashCode`, `toString` 같은 `Object`메서드를 재정의할 수 없기 때문이다

<br>

---

<br>

### 📘 단순 구현(simple implementation)

- 단순 구현은 골격 구현의 작은 변종으로, `Abstarct Map.SimpleEntry`가 좋은 예시이다 

- 단순 구현도 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만 추상클래스가 아니다
- 단순 구현은 그대로 써도 되고 필요에 맞게 확장해도 된다

<br>

---

<br>

### 📌 핵심정리

**일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다**

**복잡한 인터페이스라면 구현하는 수고를 덜어주는 골격 구현을 함께 제공하는 방법을 고려해라**

**골격 구현은 '가능한 한' 인터페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다**

**(인터페이스에 걸려 있는 구현상의 제약 떄문에 골격 구현을 추상 클래스로 제공하는 경우가 더 흔하다)**

<br>
