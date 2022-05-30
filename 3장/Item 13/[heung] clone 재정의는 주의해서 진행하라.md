Effective Java의 열세 번째 아이템 "clone 재정의는 주의해서 진행하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 0. 들어가며

Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스(mixin interface)지만, 아쉽게도 의도한 목적을 제대로 이루지 못했습니다. 가장 큰 문제는 clone 메서드가 선언된 곳이 Cloneable이 아닌 Object이고, 그마저도 protected라는 데 있습니다. 그래서 Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메서드를 호출할 수 없습니다.

하지만 이를 포함한 여러 문제점에도 불구하고 Cloneable 방식은 널리 쓰이고 있어서 잘 알아두는 것이 좋습니다. 이번 포스팅에서는 clone 메서드를 잘 동작하게끔 해주는 구현 방법과 언제 그렇게 해야 하는지, 그리고 가능한 다른 선택지에 대해 알아보겠습니다.

<br>

## 1. Cloneable 인터페이스

위에서 언급했던것 처럼 Cloneable은 복제해도 되는 클래스임을 명시하는 용도로 사용되는 인터페이스입니다. 다음은 java.lang 패키지에 구현된 실제 코드입니다.

```java
package java.lang;

public interface Cloneable {
}
```

이상한 점이 있습니다. 일반적으로 인터페이스를 구현한다는 것은 해당 클래스가 그 인터페이스에서 정의한 기능을 제공한다고 선언하는 행위입니다. 그런데 Cloneable은 어떠한 메서드도 정의하고 있지 않습니다. 그렇다면 대체 무슨 일을 하는 걸까요?

**Cloneable은 Object의 protected 메서드인 clone의 동작 방식을 결정합니다.** Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던집니다. 참고로 이는 이례적으로 사용한 예이니 따라하지 않는 것이 좋습니다.

실무에서 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이루어지라 기대합니다. 이 기대를 만족시키려면 그 클래스와 모든 상위 클래스는 허술하게 기술된 프로토콜을 지켜야만 하는데, 그 결과로 깨지기 쉽고, 위험하고, 모순적인 메커니즘이 탄생합니다. 생성자를 호출하지 않고도 객체를 생성할 수 있게 되는 것입니다.

<br>

## 2. 일반 규약

다음은 Object 명세서에서 가져온 clone 메서드의 일반 규약입니다.

>이 객체의 복사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다. 일반적인 의도는 다음과 같다. 어떤 객체 x에 대해 다음 식은 참이다.
>
>`x.clone() != x`
>
>`x.clone().getClass() == x.getClass()`
>
>`x.clone().equals(x)`
>
>하지만 위의 요구를 반드시 만족해야 하는 것은 아니다.
>
>
>
>관례상, 이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야 한다. 이 클래스와 (Object를 제외한) 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.
>
>`x.clone().getClass() == x.getClass()`
>
>관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

위 규약은 허술합니다. clone 메서드가 super.clone이 아닌, 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일러는 불평하지 않을 것입니다. 하지만 이 클래스의 하위 클래스에서 super.class를 호출한다면 잘못된 클래스의 객체가 만들어져, 결국 하위 클래스의 clone 메서드가 제대로 동작하지 않게 됩니다. 

<br>

## 3. 작성 요령

상위 클래스가 제대로 동작하는 clone 메서드를 가지고 있다고 생각하고, 이를 상속해 Cloneable을 구현하는 방법을 소개합니다. 구현 방법은 해당 클래스가 어떤 필드를 가지고 있는지에 따라 세 가지로 나뉘는데요. 하나하나 예제를 통해 알아보겠습니다.

### 3.1. 가변 상태를 참조하지 않는 클래스

클래스의 모든 필드가 기본 타입이거나 불변 객체를 참조할 경우입니다.

```java
public class PhoneNumber implements Cloneable {
  
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = (short) areaCode;
        this.prefix   = (short) prefix;
        this.lineNum  = (short) lineNum;
    }

    @Override
    public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

Object의 clone 메서드는 Object를 반환하지만 PhoneNumber의 clone 메서드는 PhoneNumber을 반환합니다. 자바가 공변 반환 타이핑(convariant return typing)을 지원하니 이렇게 하는 것이 가능하고 권장하는 방식입니다. 다시 말해, 재정의한 메서드의 반환 타입은 상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있습니다. 

<br>

### 3.2. 가변 상태를 참조하는 클래스

간단했던 앞서의 구현이 클래스가 가변 객체를 참조하는 순간 재앙으로 돌변합니다.

```java
public class Stack implements Cloneable {

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

    // 원소를 위한 공간을 적어도 하나 이상 확보한다.
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

이 클래스의 clone 메서드가 단순히 super.clone의 결과를 그대로 반환한다면 어떻게 될까요? 반환된 Stack 인스턴스의 size 필드는 올바른 값을 갖겠지만, elements 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조할 것입니다. 따라서 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변식을 해칠 것입니다.

Stack 클래스의 하나뿐인 생성자를 호출하면 이러한 상황은 절대 일어나지 않습니다. clone 메서드는 사실상 생성자와 같은 효과를 냅니다. 즉, **clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 합니다.**

Stack의 clone 메서드는 제대로 동작하려면 스택 내부 정보를 복사해야 합니다. **가장 쉬운 방법은 elements 배열의 clone을 재귀적으로 호출하는 것입니다.**

```java
@Override
public Stack clone() {
  try {
    Stack result = (Stack) super.clone();
    result.elements = elements.clone();
    return result;
  } catch (CloneNotSupportedException e) {
    throw new AssertionError();
  }
}
```

배열의 clone은 런타임 타입과 컴파일 타입 모두가 원본 배열과 똑같은 배열을 반환합니다. 따라서 배열을 복제할 때는 배열의 clone 메서드를 사용하라고 권장합니다. **사실, 배열은 clone 기능을 제대로 사용하는 유일한 예라고 할 수 있습니다.**

<br>

### 3.3. 복잡한 가변 상태를 참조하는 클래스

clone을 재귀적으로 호출하는 것만으로는 충분하지 않을 때도 있습니다. 해시테이블용 clone 메서드를 생각해 보겠습니다. 해시테이블 내부는 버킷들의 배열이고, 각 버킷은 키-값 쌍을 담는 연결 리스트의 첫 번째 엔트리를 참조합니다.

```java
public class HashTable implements Cloneable {

    private Entry[] buckets = new Entry[10];

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public void add(Object key, Object value) {
            this.next = new Entry(key, value, null);
        }
    }
  
    ...
}
```

Stack에서처럼 단순히 버킷 배열의 clone을 재귀적으로 호출하면 어떻게 될까요? 

```java
@Override
public HashTable clone() {
    HashTable result = null;
    try {
        result = (HashTable)super.clone();
        result.buckets = this.buckets.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw  new AssertionError();
    }
}
```

복제본은 자신만의 버킷 배열을 갖지만, 이 배열은 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 가능성이 생깁니다. 이를 해결하려면 **각 버킷을 구성하는 연결 리스트를 복사해야 합니다.**

```java
@Override
public HashTable clone() {
    HashTable result = null;
    try {
        result = (HashTable) super.clone();
        result.buckets = new Entry[this.buckets.length];

        for (int i = 0; i < this.buckets.length; i++) {
            if (buckets[i] != null) {
                result.buckets[i] = this.buckets[i].deepCopy(); // 연결 리스트 복사
            }
        }
        return result;
    } catch (CloneNotSupportedException e) {
        throw  new AssertionError();
    }
}
```

책에서는 deepCopy를 구현하는 세 가지 방법을 소개합니다.

#### 3.3.1 재귀

```java
public Entry deepCopy() {
    return new Entry(key, value, next == null ? null : next.deepCopy());
}
```

private 클래스인 HashTable.Entry는 깊은 복사를 지원하도록 보강되었습니다. HashTable의 clone 메서드는 먼저 적절한 크기의 새로운 버킷 배열을 할당한 다음, 원래의 버킷 배열을 순회하며 비어있지 않은 각 버킷에 대해 깊은 복사를 수행합니다. 이때 Entry의 deepCopy 메서드는 자신이 가리키는 연결 리스트 전체를 복사하기 위해 자신을 재귀적으로 호출합니다.

이 기법은 간단하며, 버킷이 너무 길지 않다면 잘 작동합니다. 하지만 재귀 호출 때문에 리스트의 원소 수만큼 스택 프레임을 소비하여, **리스트가 길면 스택 오버플로를 일으킬 위험이 있기 때문에 연결 리스트를 복제하는 방법으로는 그다지 좋지 않습니다.** 

#### 3.3.2 반복

```java
public Entry deepCopy() {
    Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p = p.next) {
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    }
    return result;
}
```

deepCopy가 재귀 호출 대신 반복자를 사용해 순회하면 스택 오버플로 문제를 해결할 수 있습니다. 

#### 3.3.3 고수준 메서드

super.clone을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정한 다음, 원본 객체의 상태를 다시 생성하는 고수준 메서드들을 호출합니다. 즉, buckets 필드를 새로운 버킷 배열로 초기화한 다음 원본 테이블에 담긴 모든 키-값 쌍 각각에 대해 복제본 테이블의 HashTable.put(key, value) 메서드를 호출해 둘의 내용이 같게 해주는 것입니다.

**고수준 API를 활용해 복제하면 간단하게 작성할 수 있지만, 아무대로 저수준에서 바로 처리할 때보다는 느립니다.** 또한 Cloneable 아키텍처의 기초가 되는 필드 단위 복사를 우회하기 때문에 전체 Cloneable 아키텍처와는 어울리지 않는 방식이기도 합니다.

<br>

## 4. 주의할 점

**1) clone에서는 재정의될 수 있는 메서드를 호출하지 않아야 한다.**

만약 clone이 하위 클래스에서 재정의한 메서드를 호출하면, 하위 클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃게 되어 원본과 복제본의 상태가 달라질 가능성이 큽니다. 

**2) public인 clone 메서드에서는 throws 절을 없애야 한다.**

Object의 clone 메서드는 CloneNotSupportedException을 던진다고 선언했지만 재정의한 메서드는 그렇지 않습니다. 검사 예외를 던지지 않아야 그 메서드를 사용하기 편하기 때문입니다.

**3) 상속용 클래스는 Cloneable을 구현해서는 안 된다.**

Object의 방식을 모방하여, 제대로 작동하는 clone 메서드를 구현해 protected로 두고 CloneNotSupportedException도 던질 수 있다고 선언하는 것입니다. 다른 방법으로는 clone을 동작하지 않게 구현해놓고 하위 클래스에서 재정의하지 못하게 할 수도 있습니다. 

**4) Cloneable을 구현한 thread-safe한 클래스를 작성할 때는 clone 메서드 역시 적절히 동기화해줘야 한다.**

Object의 clone 메서드는 동기화를 고려하지 않았습니다. 그러니 super.clone 호출 외에 다른 할 일이 없더라도 clone을 재정의하고 동기화해줘야 합니다. 

<br>

## 5. 더 나은 객체 복사 방식

Cloneable을 이미 구현한 클래스를 확장한다면 어쩔 수 없이 clone을 잘 작동하도록 구현해야 합니다. 그렇지 않은 상황에서는 **복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공할 수 있습니다.** 여기서 복사 생성자와 복사 팩터리란 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 것을 말합니다.

```java
// 복사 생성자
public Yum(Yum yum) { ... };

// 복사 팩터리
public static Yum newInstance(Yum yum) { ... };
```

모순적이고 위험한 객체 생성 메커니즘(생성자를 쓰지 않는 방식)을 사용하지 않으며, 엉성하게 문서화된 규약에 기대지 않고, 정상적인 final 필드 용법과도 충돌하지 않으며, 불필요한 검사 예외를 던지지도 않고, 형변환도 필요하지 않습니다. 

추가로 **해당 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있습니다.** 인터페이스 기반 복사 생성자와 복사 팩터리의 더 정확한 이름은 '변환 생성자(conversion constructor)'와 '변환 팩터리(conversion factory)'입니다. **이들을 이용하면 클라이언트는 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있습니다.**

다음은 HashSet 객체를 TreeSet 타입으로 복제한 클라이언트 코드입니다.

```java
Set<String> hashSet = new HashSet<>();
hashSet.add("heung");
hashSet.add("bae");
hashSet.add("hoon");
hashSet.add("na");
System.out.println("HashSet: " + hashSet);

Set<String> treeSet = new TreeSet<>(hashSet); // 복제(변환)
System.out.println("TreeSet: " + treeSet);

/* 실행 결과
HashSet: [na, hoon, bae, heung]
TreeSet: [bae, heung, hoon, na]
*/
```

<br>

## 6. 핵심 정리

- 새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서는 안 된다.
- 새로운 클래스도 Cloneable을 구현해서는 안 된다.
- final 클래스라면 Cloneable을 구현해도 위험이 크지 않지만, 성능 최적화 관점에서 검토한 후 별다른 문제가 없을 때만 그물게 허용해야 한다.
- 기본 원칙은 '복제 기능은 생성자와 팩터리를 이용하는 것이 최고'라는 것이다.
- 예외적으로 배열은 clone 메서드 방식이 가장 깔끔한 복제 방법이다.

<br>

## 7. Related Posts

- 믹스인 인터페이스 (Item 20)
- 리플렉션 (Item 65)
- Checked Exception (Item 71)
- 상속을 고려한 설계 (Item 19)
- 동기화 (Item 78)
