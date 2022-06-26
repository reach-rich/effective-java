

***❗ 주의사항***

*아래의 내용에서 상속이란 클래스가 다른 **클래스를 확장하는 구현상속**으로 클래스가 인터페이스를 또는 인터페이스가 다른 **인터페이스를 상속하는 경우는 제외**한다*



### 🔒 상속의 문제점

**: 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다**

**✏ #01 예제소스 | 상속을 잘못 사용**

```java
public class InstrumentedHashSet<E> extends HashSet<E>{
    private int addCount = 0;
    
    public InstrumentedHashSet() {
    }
    
    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }
    
    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    
    public int getAddCount() {
		return addCount;
    }
}
```

>  **메서드 재정의 시 문제**
>
> - 

>**메서드 추가 시 문제**
>
>- 

<br>

---

<br>

### 🗝 컴포지션

**: 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조**

**✏ #02 예제소스 | 상속 대신 컴포지션 사용**

```java
public class InstrumentedHashSet<E> extends HashSet<E>{
    private int addCount = 0;
    
    public InstrumentedHashSet(Set<E> s) {
        super(s);
    }
    
    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    
    public int getAddCount() {
		return addCount;
    }
}
```



**✏ #03 예제소스 | 재사용할 수 있는 전달 클래스**

```java
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

```

