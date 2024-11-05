### 🔍 clone 재정의

**: `clone`메서드가 선언된 곳이 `Cloneable`이 아닌 `Object`이고, 그마저도 `protected`이므로, `Cloneable`을 구현하는 것만으로는 `clone` 메서드를 호출할 수 없다.**

<br>

실무에서 `Cloneable`을 구현한 클래스는 `clone` 메서드를 `pulblic` 으로 제공하며, 사용자는 당연히 복제가 제대로 이루어지리라 기대한다

이를 위해서 해당 클래스와 모든 상위 클래스는 복잡하고, 강제할 수 없고, 허술하게 기술된 프로토콜을 지켜야하는데 그 결과 모순적인 메커니즘이 탄생한다

<br>

> **📝 `clone` 메서드 허술한(?) 일반 규약**
>
> - 이 객체의 복사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다. 일반적인 의도는 다음과 같다.
> - `x.clone() != x`  어떤 객체 x에 대해 다음 식은 참이다
> - `x.clone().getClass() == x.Class()` | 어떤 객체 x에 대해 다음 식은 참이다
> - `x.clone().equals(x)` | 일반적으로 참이지만, 필수는 아니다
> - `x.clone().getClass() == x.getClass()`
>   - 관례상, 이 메서드가 반환하는 객체는 `super.clone`을 호출해 얻어야한다.
>   - 이 클래스와 (Object를 제외한) 모든 상위 클래스가 이 관례를 따른다면 참이다.
> - 관례상, 반환된 객체와 원본 객체는 독립적이어야한다. 이를 만족하려면 `super.clone`으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

<br>

---

<br>

### 📗 Stack 클래스의 clone 재정의

**✏ #01 예제소스 | 가변 상태를 참조하지 않은 클래스용 clone 메서드**

```java
@Override public PhoneNumber clone() {
	try {
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError(); // 일어날 수 없는 일이다
    }
}
```

>`super.clone`을 호출함으로써 복제본 얻을 수 있음

<br>

**✏ #02 예제소스 | Stack(가변객체)**

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
		this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e) {
		ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
    
    private void ensureCapacity() {
		if (elements.length == size)
        elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

>`super.clone`의 결과를 그대로 반환한다면 `Stack` 인스턴스의 `size` 필드는 올바른 값을 갖지만 `elements` 필드는 원본 `Stack`인스턴스와 똑같은 배열을 참조
>
>둘 중 하나만 수정하여도 `NullPointerException` 문제 발생

<br>

**✏ #03 예제소스 | 가변 상태를 참조하는 클래스용 clone 메서드**

```java
@Override public Stack clone() {
	try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

>`clone`메서드는 사실상 생성자와 같은 효과
>
>`clone`은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 함
>
>`elements` 필드가 `final`이 었다면 정상 작동 X
>
>Cloneable 아키텍처는 '가변 객체를 참조하는 필드는 `final`로 선언하라'는 일반 용법과 충돌

<br>

---

<br>

### 📘 HashTable 클래스의 clone 재정의

**✏ #04 예제소스 | HashTable**

```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;
    
    private static class Entry {
        final Object key;
        Object value;
        Entry next;
        
        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }
   	...
}
```

<br>

**✏ #05 예제소스 | 잘못된 clone메서드 - 가변상태를 공유한다!**

```java
@Override public HashTable clone() {
    try {
        HashTable result = (HashTable) super.clone();
        result.buckets = buckets.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

> 복제본은 자신만의 버킷 배열을 갖지만, 원본과 같은 연결리스트를 참조
>
> 이를 해결하려면 각 버킷을 구성하는 연결 리스트를 복사해야 한다

<br>

**✏ #06 예제소스 | 복잡한 가변 상태를 갖는 클래스용 재귀적 clone 메서드**

```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;
    
    private static class Entry {
        final Object key;
        Object value;
        Entry next;
        
        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
        
        Entry deepCopy() {
            return new Entry (key, value, 
            	next == null ? null : next.deepCopy());
        }
    }
   	
    @Override public HashTable clone() {
    try {
        HashTable result = (HashTable) super.clone();
        result.buckets = new Entry[buckets.length];
        for (int i = 0; i < buckets.length; i++)
            if (buckets[i] != null)
                result.buckets[i] = buckets[i].deepCopy();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
    ...
}
```

> `private` 클래스인 `HashTable.Entry`는 깊은복사를 지원하도록 보강
>
> 적절한 크기의 새로운 버킷 배열을 할당한 뒤 `deepCopy`메서드 재귀적으로 호출
>
> 리스트가 길면 스택 오버플로를 일으킬 위험이 발생
>
> 따라서 `deepCopy`를 재귀 호출 대신 반복자를 써서 순회하는 방법이 필요

<br>

**✏ #07 예제소스 | 엔트리 자신이 가리키는 연결 리스트를 반복적으로 복사한다.**

```java
Entry deepCopy() {
    Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p = p.next)
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    return result;
}
```

<br>

---

<br>

### 📙 복잡한 가변 객체를 복제하는 마지막 방법

1. `super.clone`을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정

2. 원본 객체의 상태를 다시 생성해주고 고수준 메서드들을 호출

   **HashTable의 경우**

   > `buckets`필드를 새로운 버킷 배열로 초기화
   >
   > 원본 테이블에 담긴 모든 키-값 쌍 각각에 대해 복제본 테이블의 `put(key, value)` 메서드를 호출해 둘의 내용을 똑같해준다

이처럼 고수준 API를 활용하면 간단하고 우아한 코드 가능하지만 저수준에서 바로 처리할 때보다 느림

`Cloneable`아키텍처 기초가 되는 필드 단위 객체 복사를 우회하기 때문에 어울리지 않는 방식

<br>

---

<br>

### 💡 복사 생성자 & 복사 팩터리

**: 복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공할 수 있다**

`Cloneable`을 구현하는 모든 클래스는 `clone`을 재정의해야 하지만 그렇지 않은 경우 복사 생성자와 복사 팩터리 방식 제공

> - 언어 모순적이고 위험천만한 객체 생성 메커니즘을 사용하지 않음
>
> - 엉성하게 문서화된 규약에 기대지 않음
>
> - 정상적인 `final`필드 용법과 충돌하지 않음
>
> - 불필요한 검사 예외를 더닞지 않고, 형변환도 필요치 않음
> - 해당 클래스가 구현한 '인터페이스' 타입의 인스턴스를 인수로 받을 수 있음

<br>

---

<br>

### 📌 핵심정리

**새로운 인터페이스를 만들 때는 절대 `Cloneable`을 확장해서는 안 되며, 새로운 클래스도 이를 구현해서는 안된다.**
**`final`클래스라면 `Cloneable`을 구현해도 위험이 크지 않지만 별 다른 문제 없을 때만 드물게 허용해야한다.**
**기본 원칙은 '복제 기능은 생성자와 팩터리를 이용하는게 최고'라는 것!!**
