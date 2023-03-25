### 🛡 방어적 복사본

- 자바로 작성한 클래스는 불변식이 지켜짐
- 자바라 해도 다른 클래스로부터의 침범까지는 막지 못함
- 클라이언트가 불변식을 깨뜨릴 수 있기에 방어적으로 프로그래밍을 해야 함

<br>

**✏ #01 예제소스 | 기간을 표현하는 클래스 - 불변식을 지키지 못했다**

```java
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param  start 시작 시각
     * @param  end 종료 시각. 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발생한다.
     */
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(start + " after " + end);
        this.start = start;
        this.end   = end;
    }

    public Date start() {
        return start;
    }
            
    public Date end() {
        return end;
    }  
}
```

>시작 시각이 종료 시각보다 늦을 수 없다는 불변식이 잘 지켜질 것 같지만, `Date`가 가변이라는 점을 이용해 깨뜨릴 수 있음

<br>

---

<br>

### ⚔ 인스턴스 공격 01

**: 객체 외부에서 내부를 수정하도록 허락하는 경우**

<br>

**✏ #02 예제소스 | Period 인스턴스의 내부를 공격**

```java
Date start	= new Date();
Date end	= new Date();
Period p	= new Period(start, end);
end.setYear(78);	// p의 내부를 수정
```

>`Date` 대신 불변인 `Instant`를 사용하여 해결 가능
>
>`Date`는 낡은 API이므로 새로운 코드를 작성할 때는 더 이상 사용하면 안됨
>
>그러나 API와 내부구현에 낡은 코드의 잔재가 남아있음

<br>

**❓ API와 내부구현에 남아있는 낡은 코드를 어떻게 대처할 것인가**

- 외부 공격으로부터 `Period` 인스턴스의 내부를 보호하려면 생성자에게 받은 가변 매개변수 각각을 방어적으로 복사(defensive copy)해야 함
- 이후 `Period` 인스턴스의 원본이 아닌 복사본을 사용

<br>

**✏ #03 예제소스 | 수정한 생성자 - 매개변수의 방어적 복사본 생성**

```java
public Period(Date start, Date end) {
    this.start = new Date(start.getTiem());
    this.end = new Date(end.getTime());
    
    if (this.start.compareTo(this.end) > 0)
        throw new IllegalArgumentException(this.start + " after " + this.end);
}
```

>매개변수의 유효성을 검사하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사
>
>멀티스레딩 환경에서는 원본 객체의 유효성을 검사한 후 복사본을 만드는 순간 다른 스레드가 원본 객체를 수정할 위험이 존재
>
>방어적 복사를 매개변수 유효성 검사 전에 수행하면 이러한 위험을 없앨 수 있음

***검사시점/사용시점(time-of-check/time-of-use)**

<br>

**💡 방어적 복사에 Date 의 clone 메서드를 사용하지 않는 이유**

- `Date`는 `final`이 아니므로 `clone`의 `Date`가 정의한 것이 아닐 수 있음
- `clone`이 악의를 가진 하위 클래스의 인스턴스를 반환할 수 있음
- 하위 클래스는 `start`와 `end`필드의 참조를 `private` 정적 리스트에 담아뒀다가 공격자가 리스트에 접근 할 수도 있음
- 매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 `clone`을 사용해서는 안 됨

<br>

---

<br>

### ⚔ 인스턴스 공격 02

**: 내부의 가변 정보를 직접 드러내기 때문에 `Period` 인스턴스는 아직 변경 가능**

<br>

**✏ #04 예제코드 | Peroid 인스턴스를 향한 두 번째 공격**

```java
Date start	= new Date();
Date end	= new Date();
Period p	= new Period(start, end);
p.end().setYear(78);	// p의 내부를 수정
```

<br>

**✏ #05 예제코드 | 수정한 접근자 - 필드의 방어적 복사본을 반환**

```java
public Date start() {
	return new Date(start.getTime());
}
            
public Date end() {
	return new Date(end.getTime());
}  
```

>가변 필드의 방어적 복사본을 반환하여 해결

<br>

**💡 생성자와 달리 접근자 메서드에서는 방어적 복사에 clone 사용 가능**

- `Period`가 가지고 있는 `Date` 객체는 `java.util.Date`가 확실하기 때문
- 그럼에도 인스턴스를 복사할 때에는 일반적으로 생성자나 정적 팩터리를 사용하는 것이 좋음

<br>

---

<br>

### ❗ 주의사항

**메서드든 생성자든 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관해야 할 때면 항시 그 객체가 잠재적으로 변경될 수 있는지 생각해야 함**

- 변경될 수 있는 객체라면 변경되어도 문제없이 작동하는지 확인
- 예를 들어 클라이언트가 건네준 객체를 내부의 `Set` 인스턴스나 `Map` 인스턴스의 키로 사용한다면, 변경될 경우 불변식이 깨질 수 있음

<br>

**내부 객체를 클라이언트에 건네주기 전에 방어적 복사본을 만드는 이유도 같음**

- 클래스가 불변이든 가변이든, 가변인 내부 객체를 클라이언트에게 반환할 때 반드시 심사숙고 필요
- 안심할 수 없다면 방어적 복사본을 반환
- 길이가 1이상인 배열은 무조건 가변임을 명심
- 따라서 내부에서 사용하는 배열을 클라이언트에 반환할 때 항상 방어적 복사를 수행해야 하거나 혹은 배열의 불변 뷰를 반환해야 함

<br>

**가능한 불변 객체들을 조합해 객체를 구성해야 방어적 복사를 할 일이 줄어듦**

- Period 예제의 경우, 자바8 이상이라면 `Instant`를 사용
- 이전 버전의 경우 `Date.getTime()`을 반환하는 `long` 정수를 사용하는 방법도 있음

<br>

**방어적 복사는 성능 저하가 따르고, 항상 쓸 수 있는 것도 아님**

- 호출자가 컴포넌트 내부를 수정하지 않으리라 확신하면 방어적 복사를 생략할 수 있음
- 이러한 상황이라도 호출자에서 해당 매개변수나 반환값을 수정하지 말아야 함을 명확히 문서화 필요

<br>

**다른 패키지에서 사용한다고 해서 넘겨받은 가변 매개변수를 항상 방어적으로 복사해 저장해야 할 필요는 없음**

- 방어적 복사를 생략해도 되는 상황은 해당 클래스와 그 클라이언트가 상호 신뢰할 수 있을 때, 혹은 불변식이 깨져도 영향이 오직 호출한 클라이언트로 국한될 때로 한정해야 함
- 래퍼 클래스의 경우 클라이언트는 래퍼에 넘긴 객체에 여전히 직접 접근할 수 있지만 그 영향을 오직 클라이언트 자신만 받게 됨

<br>

---

<br>

### 📌 핵심정리

**클래스가 클라이언트로부터 받는 혹은 클라이언트로 반환하는 구성요소가 가변이라면 그 요소는 반드시 방어적으로 복사해야 한다**

**복사 비용이 너무 크거나 클라이언트가 그 요소를 잘못 수정할 일이 없음을 신뢰한다면 방어적 복사를 수행하는 대신 해당 구성요소를 수정했을 때의 책임이 클라이언트에 있음을 문서에 명시하도록 하자**
