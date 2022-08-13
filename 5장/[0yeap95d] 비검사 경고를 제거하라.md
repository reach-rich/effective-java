

### ❗ 제너릭 사용으로 발생하는 수많은 컴파일러 경고

- 비검사 형변환 경고

- 비검사 메서드 호출 경고

- 비검사 매개변수화 가변인수 타입 경고

- 비검사 변환 경고

>`Set<Lark> exaltation = new HashSet();`
>
>`Set<Lark> exaltation = new HashSet<>();` - 다이아몬드 연산자로 해결

<br>

---

<br>

### ✔ 할 수 있는 한 모든 비검사 경고를 제거하라

- 타입 안전성 보장
- 런타임에 `ClassCastException` 발생할 일이 없음

<br>

---

<br>

### 🔎 @SuppressWarnings("unchecked")

**경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 `@SuppressWarnings("unchecked")` 애너테이션을 달아 경고를 숨겨라**

- 타입 안전함을 검증해야한다

- 변수 선언, 아주 짧은 메서드 혹은 생성자 등 가능한 한 좁은 범위에 적용해라

<br>

**✏ #01 예제소스**

```java
public <T> T[] toArray(T[] a) {
	if (a.length < size)
        return (T[]) Arrays.copyOf(elements, size, a.getClass());
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}

// 컴파일 경고 발생!
```

```tex
ArrayList.java:305: warning [unchecked] unchecked cast
		return (T[]) Arrays.copyOf(elements, size, a.getClass());
								   ^
	required:	T[]
	found:		Object[]
```

>애너테이션은 선언에만 달 수 있기 때문에 `return`문에는 `@SuppressWarning`를 다는 게 불가능
>
>전체에 달면 범위가 필요 이상으로 넓어지니 자제

<br>

**✏ #02 예제소스 | 지역변수를 추가해 @SuppressWarning 범위를 좁힌다**

```java
public <T> T[] toArray(T[] a) {
	if (a.length < size) {
        @SuppressWarning("unchecked") 
        T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
        return result;
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

>반환 값을 담을 지역변수를 선언하고 그 변수에 애너테이션 추가

<br>

---

<br>

### 📌 핵심정리

**비검사 경고는 중요하니 무시하지말자**

**모든 비검사 경고는 런타임에 `ClassCastException`을 일으킬 수 있는 잠재적 가능성을 뜻하니 최선을 다해 제거하라**

**경고를 없앨 방법을 차지 못하겠다면, 그 코드가 타입 안전함을 증명하고 가능한 한 범위를 좁혀 `@SuppressWarnings("unchecked")` 애너테이션으로 경고를 숨겨라**

**그런 다음 경고를 숨기기로 한 근거를 주석으로 남겨라**

<br>
