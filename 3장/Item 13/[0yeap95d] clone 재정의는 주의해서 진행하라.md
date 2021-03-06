### π clone μ¬μ μ

**: `clone`λ©μλκ° μ μΈλ κ³³μ΄ `Cloneable`μ΄ μλ `Object`μ΄κ³ , κ·Έλ§μ λ `protected`μ΄λ―λ‘, `Cloneable`μ κ΅¬ννλ κ²λ§μΌλ‘λ `clone` λ©μλλ₯Ό νΈμΆν  μ μλ€.**

<br>

μ€λ¬΄μμ `Cloneable`μ κ΅¬νν ν΄λμ€λ `clone` λ©μλλ₯Ό `pulblic` μΌλ‘ μ κ³΅νλ©°, μ¬μ©μλ λΉμ°ν λ³΅μ κ° μ λλ‘ μ΄λ£¨μ΄μ§λ¦¬λΌ κΈ°λνλ€

μ΄λ₯Ό μν΄μ ν΄λΉ ν΄λμ€μ λͺ¨λ  μμ ν΄λμ€λ λ³΅μ‘νκ³ , κ°μ ν  μ μκ³ , νμ νκ² κΈ°μ λ νλ‘ν μ½μ μ§μΌμΌνλλ° κ·Έ κ²°κ³Ό λͺ¨μμ μΈ λ©μ»€λμ¦μ΄ νμνλ€

<br>

> **π `clone` λ©μλ νμ ν(?) μΌλ° κ·μ½**
>
> - μ΄ κ°μ²΄μ λ³΅μ¬λ³Έμ μμ±ν΄ λ°ννλ€. 'λ³΅μ¬'μ μ νν λ»μ κ·Έ κ°μ²΄λ₯Ό κ΅¬νν ν΄λμ€μ λ°λΌ λ€λ₯Ό μ μλ€. μΌλ°μ μΈ μλλ λ€μκ³Ό κ°λ€.
> - `x.clone() != x`  μ΄λ€ κ°μ²΄ xμ λν΄ λ€μ μμ μ°Έμ΄λ€
> - `x.clone().getClass() == x.Class()` | μ΄λ€ κ°μ²΄ xμ λν΄ λ€μ μμ μ°Έμ΄λ€
> - `x.clone().equals(x)` | μΌλ°μ μΌλ‘ μ°Έμ΄μ§λ§, νμλ μλλ€
> - `x.clone().getClass() == x.getClass()`
>   - κ΄λ‘μ, μ΄ λ©μλκ° λ°ννλ κ°μ²΄λ `super.clone`μ νΈμΆν΄ μ»μ΄μΌνλ€.
>   - μ΄ ν΄λμ€μ (Objectλ₯Ό μ μΈν) λͺ¨λ  μμ ν΄λμ€κ° μ΄ κ΄λ‘λ₯Ό λ°λ₯Έλ€λ©΄ μ°Έμ΄λ€.
> - κ΄λ‘μ, λ°νλ κ°μ²΄μ μλ³Έ κ°μ²΄λ λλ¦½μ μ΄μ΄μΌνλ€. μ΄λ₯Ό λ§μ‘±νλ €λ©΄ `super.clone`μΌλ‘ μ»μ κ°μ²΄μ νλ μ€ νλ μ΄μμ λ°ν μ μ μμ ν΄μΌ ν  μλ μλ€.

<br>

---

<br>

### π Stack ν΄λμ€μ clone μ¬μ μ

**β #01 μμ μμ€ | κ°λ³ μνλ₯Ό μ°Έμ‘°νμ§ μμ ν΄λμ€μ© clone λ©μλ**

```java
@Override public PhoneNumber clone() {
	try {
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError(); // μΌμ΄λ  μ μλ μΌμ΄λ€
    }
}
```

>`super.clone`μ νΈμΆν¨μΌλ‘μ¨ λ³΅μ λ³Έ μ»μ μ μμ

<br>

**β #02 μμ μμ€ | Stack(κ°λ³κ°μ²΄)**

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

>`super.clone`μ κ²°κ³Όλ₯Ό κ·Έλλ‘ λ°ννλ€λ©΄ `Stack` μΈμ€ν΄μ€μ `size` νλλ μ¬λ°λ₯Έ κ°μ κ°μ§λ§ `elements` νλλ μλ³Έ `Stack`μΈμ€ν΄μ€μ λκ°μ λ°°μ΄μ μ°Έμ‘°
>
>λ μ€ νλλ§ μμ νμ¬λ `NullPointerException` λ¬Έμ  λ°μ

<br>

**β #03 μμ μμ€ | κ°λ³ μνλ₯Ό μ°Έμ‘°νλ ν΄λμ€μ© clone λ©μλ**

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

>`clone`λ©μλλ μ¬μ€μ μμ±μμ κ°μ ν¨κ³Ό
>
>`clone`μ μλ³Έ κ°μ²΄μ μλ¬΄λ° ν΄λ₯Ό λΌμΉμ§ μλ λμμ λ³΅μ λ κ°μ²΄μ λΆλ³μμ λ³΄μ₯ν΄μΌ ν¨
>
>`elements` νλκ° `final`μ΄ μλ€λ©΄ μ μ μλ X
>
>Cloneable μν€νμ²λ 'κ°λ³ κ°μ²΄λ₯Ό μ°Έμ‘°νλ νλλ `final`λ‘ μ μΈνλΌ'λ μΌλ° μ©λ²κ³Ό μΆ©λ

<br>

---

<br>

### π HashTable ν΄λμ€μ clone μ¬μ μ

**β #04 μμ μμ€ | HashTable**

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

**β #05 μμ μμ€ | μλͺ»λ cloneλ©μλ - κ°λ³μνλ₯Ό κ³΅μ νλ€!**

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

> λ³΅μ λ³Έμ μμ λ§μ λ²ν· λ°°μ΄μ κ°μ§λ§, μλ³Έκ³Ό κ°μ μ°κ²°λ¦¬μ€νΈλ₯Ό μ°Έμ‘°
>
> μ΄λ₯Ό ν΄κ²°νλ €λ©΄ κ° λ²ν·μ κ΅¬μ±νλ μ°κ²° λ¦¬μ€νΈλ₯Ό λ³΅μ¬ν΄μΌ νλ€

<br>

**β #06 μμ μμ€ | λ³΅μ‘ν κ°λ³ μνλ₯Ό κ°λ ν΄λμ€μ© μ¬κ·μ  clone λ©μλ**

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

> `private` ν΄λμ€μΈ `HashTable.Entry`λ κΉμλ³΅μ¬λ₯Ό μ§μνλλ‘ λ³΄κ°
>
> μ μ ν ν¬κΈ°μ μλ‘μ΄ λ²ν· λ°°μ΄μ ν λΉν λ€ `deepCopy`λ©μλ μ¬κ·μ μΌλ‘ νΈμΆ
>
> λ¦¬μ€νΈκ° κΈΈλ©΄ μ€ν μ€λ²νλ‘λ₯Ό μΌμΌν¬ μνμ΄ λ°μ
>
> λ°λΌμ `deepCopy`λ₯Ό μ¬κ· νΈμΆ λμ  λ°λ³΅μλ₯Ό μ¨μ μννλ λ°©λ²μ΄ νμ

<br>

**β #07 μμ μμ€ | μνΈλ¦¬ μμ μ΄ κ°λ¦¬ν€λ μ°κ²° λ¦¬μ€νΈλ₯Ό λ°λ³΅μ μΌλ‘ λ³΅μ¬νλ€.**

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

### π λ³΅μ‘ν κ°λ³ κ°μ²΄λ₯Ό λ³΅μ νλ λ§μ§λ§ λ°©λ²

1. `super.clone`μ νΈμΆνμ¬ μ»μ κ°μ²΄μ λͺ¨λ  νλλ₯Ό μ΄κΈ° μνλ‘ μ€μ 

2. μλ³Έ κ°μ²΄μ μνλ₯Ό λ€μ μμ±ν΄μ£Όκ³  κ³ μμ€ λ©μλλ€μ νΈμΆ

   **HashTableμ κ²½μ°**

   > `buckets`νλλ₯Ό μλ‘μ΄ λ²ν· λ°°μ΄λ‘ μ΄κΈ°ν
   >
   > μλ³Έ νμ΄λΈμ λ΄κΈ΄ λͺ¨λ  ν€-κ° μ κ°κ°μ λν΄ λ³΅μ λ³Έ νμ΄λΈμ `put(key, value)` λ©μλλ₯Ό νΈμΆν΄ λμ λ΄μ©μ λκ°ν΄μ€λ€

μ΄μ²λΌ κ³ μμ€ APIλ₯Ό νμ©νλ©΄ κ°λ¨νκ³  μ°μν μ½λ κ°λ₯νμ§λ§ μ μμ€μμ λ°λ‘ μ²λ¦¬ν  λλ³΄λ€ λλ¦Ό

`Cloneable`μν€νμ² κΈ°μ΄κ° λλ νλ λ¨μ κ°μ²΄ λ³΅μ¬λ₯Ό μ°ννκΈ° λλ¬Έμ μ΄μΈλ¦¬μ§ μλ λ°©μ

<br>

---

<br>

### π‘ λ³΅μ¬ μμ±μ & λ³΅μ¬ ν©ν°λ¦¬

**: λ³΅μ¬ μμ±μμ λ³΅μ¬ ν©ν°λ¦¬λΌλ λ λμ κ°μ²΄ λ³΅μ¬ λ°©μμ μ κ³΅ν  μ μλ€**

`Cloneable`μ κ΅¬ννλ λͺ¨λ  ν΄λμ€λ `clone`μ μ¬μ μν΄μΌ νμ§λ§ κ·Έλ μ§ μμ κ²½μ° λ³΅μ¬ μμ±μμ λ³΅μ¬ ν©ν°λ¦¬ λ°©μ μ κ³΅

> - μΈμ΄ λͺ¨μμ μ΄κ³  μνμ²λ§ν κ°μ²΄ μμ± λ©μ»€λμ¦μ μ¬μ©νμ§ μμ
>
> - μμ±νκ² λ¬Έμνλ κ·μ½μ κΈ°λμ§ μμ
>
> - μ μμ μΈ `final`νλ μ©λ²κ³Ό μΆ©λνμ§ μμ
>
> - λΆνμν κ²μ¬ μμΈλ₯Ό λλμ§ μκ³ , νλ³νλ νμμΉ μμ
> - ν΄λΉ ν΄λμ€κ° κ΅¬νν 'μΈν°νμ΄μ€' νμμ μΈμ€ν΄μ€λ₯Ό μΈμλ‘ λ°μ μ μμ

<br>

---

<br>

### π ν΅μ¬μ λ¦¬

**μλ‘μ΄ μΈν°νμ΄μ€λ₯Ό λ§λ€ λλ μ λ `Cloneable`μ νμ₯ν΄μλ μ λλ©°, μλ‘μ΄ ν΄λμ€λ μ΄λ₯Ό κ΅¬νν΄μλ μλλ€.**
**`final`ν΄λμ€λΌλ©΄ `Cloneable`μ κ΅¬νν΄λ μνμ΄ ν¬μ§ μμ§λ§ λ³ λ€λ₯Έ λ¬Έμ  μμ λλ§ λλ¬Όκ² νμ©ν΄μΌνλ€.**
**κΈ°λ³Έ μμΉμ 'λ³΅μ  κΈ°λ₯μ μμ±μμ ν©ν°λ¦¬λ₯Ό μ΄μ©νλκ² μ΅κ³ 'λΌλ κ²!!**
