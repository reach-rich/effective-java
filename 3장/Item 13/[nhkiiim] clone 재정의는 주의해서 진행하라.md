# clone 재정의는 주의해서 진행하라

### 0. java의 Clone
https://staticclass.tistory.com/77

https://logical-code.tistory.com/188

#
### 1. Coloneable은 메서드가 없는 인터페이스
- Coloneable : 원본 객체의 필드 값과 동일한 값을 가지는 새로운 객체를 생성해줌
- clone 이라는 메서드를 사용 BUT 메서드가 Coloneable은 인터페이스가 아닌 오브젝트에 선언되어 있음 
- 의도한 목적을 제대로 이루지 못함!!

- 이외에도 여러가지 문제점이 있지만, 널리 쓰이고 있기 때문에 알아두자~!!

#
### 2. Object의 protected 메서드 clone
- Coloneable은 Object의 protected 메서드인 clone의 동작 방식을 결정

- 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환
- 그렇지 않으면 CloneNotSupportedException! -> 그래서 예외처리는 필수~!!

- 안터페이스는 원래 정의한 기능을 사용할거야~ 하고 선언하는 기능을 하지만, Colneable은 상위 클래스의 메서드의 동작만 변경함
- 실무에서는 Cloneable을 구현한 클래스는 clone메서드를 public으로 제공하며 사용자는 복제가 될거라고 믿음

<br>

```java
@Override
public PhoneNumber clone() {
   try {
      return (PhoneNumber) super.clone();
   } catch (CloneNotSupportedException e) {
      throw new AssertionError(); // 일어날 수 없는 일이다.
   }
}
```

- 아무튼 그래서 clone을 사용하려면 Override 해줘야한다고
- 확실히 통상적인 인터페이스 사용 방법은 아냐~
- 그리고 이게 끝이 아님!!

#
### 3. clone 메서드의 허술한 일반 규약

    이 객체의 복사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다. 
    일반적인 의도는 다음과 같다. 어떤 객체 x에 대해 다음 식은 참이다.

    x.clone() != x
    
    또한 다음 식도 참이다.

    x.clone().getClass() == x.getClass()

    하지만 이상의 요구를 반드시 만족해야 하는 것은 아니다.
    한편 다음 식도 일반적으로 참이지만, 역시 필수는 아니다.

    x.clone().equals(x)

    관례상, 이 메서드가 반환하는 객체는 super.clone을 호출해서 얻어야 한다. 이 클래스와 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.

    x.clone().getClass() == x.getClass()
    관례상, 반환된 객체와 원본 객체는 독립적이야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다. 


#
### 4. 문제가 굉장히 많아
- 불변 객체만 참조한다면 문제 없지만 가변 객체를 참조하게 되면 여기서 또 문제가 생김

- 가변 객체를 참조하면 원본과 복제본 모두 동일한 가변 객체를 참조하게 됨 -> 이후 문제 발생 가능

```java
public class Stack {
   private Object[] elements;
   private int size = 0;
   private static final int DEFAULT_INITIAL_CAPACITY = 16;
   //...
}
```
- 이 클래스를 복제하면 복제된 인스턴스가 생성자를 통해 생성되는 것이 아니기 때문에 element 필드는 원본 인스턴스와 동일한 배열 참조

- 한쪽에서 elements의  값을 변경시키면? -> 버그 발생 가능 ㅠ0ㅜ


### 해결방법 1

- elements의 배열의 재귀적으로 호출

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

- 하지만 이 방법은 필드가 final일 때는 통하지 않음 -> 새로운 값 할당 불가~!!


### 해결방법 2

- 재귀적 호출로만은 충분하지 않다 -> 해시테이블용 clone을 보라~
- 해시 테이블의 내부는 버킷들의 배열, 각 버킷은 값-쌍을 담는 연결 리스트의 첫번째 엔트리를 참조

- 그리고 성능을 위해 직접 구현한 경량 연결리스트를 써보자!

```java
public class HashTable implements Cloneable {
    private Entry[] buckets;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }

    @Override
    protected HashTable clone() {

        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = buckets.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```
- 복제된 인스턴스는 자신만의 버킷 배열을 가지지만 동일한 연결 리스트를 참조

- 각 버킷을 구성하는 연결리스트를 복사해야함

```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
        
        Entry deepCopy() {
            return new Entry(key, value, next == null ? null : next.deepCopy());
        }
    }

    @Override
    protected HashTable clone() {

        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            for (int i = 0; i < buckets.length; i++) {
                if (buckets[i] != null)
                    result.buckets[i] = buckets[i].deepCopy();
            }
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```
- 깊은 복사를 하도록 했음
- 이 방법은 간단하며 잘 작동한다 BUT 연결리스트를 복제하는 방법으로는 좋지 않음,, (어쩌라고~)
- 스택오버플로우를 일으킬 위험이 있음


### 반복자를 사용하는 방법으로 순회

```java
Entry deepCopy() {
  Entry result = new Entry(key, value, next);
  for (Entry p = result; p.next != null; p = p.next)
    p.next = new Entry(p.next.key, p.next.value, p.next.next);
  return result;
}
```


#
### 5. 그래서 결론..? -> 그냥 복사 생성자 / 복사 팩터리를 사용해라

- Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone 메서드 역시 적절히 동기화해줘야함

- Object의 clone() 메서드는 동기화를 신경쓰지 않았으므로 super.clone() 호출 외에 다른 할 일이 없더라도 clone()을 재정의하고 동기화해줘야함!!
- Cloneable을 이미 구현한 클래스를 확장한다면 어쩔 수 없이 clone()을 잘 작동하도록 구현해야 하지만 그게 아니라면 그냥 복사 생성자 / 복사 팩터리를 쓰자


### 복사 생성자

```java
public Yum(Yum yum) { ... };
```

### 복사 팩터리

```java
public static Yun newInstance(Yum yum) { ... };
```

<br>

- 복사 생성자와 복사 팩토리는 Cloneable 방식보다 낫자!
- 생성자를 쓰지 않는 방식의 객체 생성 메커니즘을 사용하지 않는다!
- 엉성하게 문서화된 규약에 기대지 않아 정상적인 final 필드 용법과도 충돌하지 않는다!
- 불필요한 검사 예외를 던지지 않고, 형변환도 필요하지 않다!

- 해당 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있다!

