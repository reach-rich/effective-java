### 🔍 hashCode 재정의

**:```equals```를 재정의한 클래스 모두에서 ```hashCode```도 재정의해야한다.**

그렇지 않으면 HashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.

<br>

> ```hashCode``` 일반 규약?
>
> - equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 ```hashCode``` 메서드는 일관되게 항상 같은 값을 반환해야 한다.
> - ```equals(Object)```가 두 객체를 같다고 판단했다면, 두 객체의 ```hashCode```는 똑같은 값을 반환해야 한다.
> - ```equals(Object)```가 두 객체를 다르다고 판단했더라도, 두 객체의 ```hashCode```가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

<br>

---

<br>

### ⚖두 객체가 같은 경우

**: 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다**

<br>

**✏ #01 예제소스 | hashCode 재정의 x**

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(111, 234, 987), "제니");

m.get(new PhoneNumber(111, 234, 987));	// null을 반환
```

> ```HashMap```에 넣을 때와 꺼낼 때 총 2개의 ```PhoneNumber```가 사용 되었음
>
> ```PhoneNumber``` 클래스는 ```hashCode```를 재정의 하지 않아서 서로 다른 해시코드 반환

<br>

**✏ #02 예제소스 | 최악의 hashCode 구현**

```java
@Override public int hashCode() { return 42; }
```

>동치인 모든 객체가 똑같은 해시코드를 반환하니 틀린건 아니지만 사용하지 말 것
>
>모든 객체가 해시테이블의 버킷 하나에 담겨, 평균 수행 시간이 O(1)에서 O(n)으로 느려짐

<br>

---

<br>

### 🎯 좋은 해시 함수란?

**: 서로 다른 인스턴스에 다른 해시코드를 반환한다**

<br>

**📝좋은 ```hashCode```를 작성하는 간단한 요령**

1. ```int``` 변수인 ```result```를 선언한 후 값을 ```c```로 초기화한다.
   이 때, ```c```는 해당 객체의 첫번째 핵심 필드를 단계 2.a 방식으로 계산한 해시코드이다. 

   (여기서 핵심 필드는 ```equals``` 비교에 사용되는 필드를 말한다.)

2. 해당 객체의 나머지 핵심 필드인 ```f``` 각각에 대해 다음 작업을 수행한다.

   - a. 해당 필드의 해시코드 ```c``` 를 계산한다.
     - 기본 타입 필드라면, ```Type.hashCode(f)```를 수행한다. 여기서 ```Type```은 해당 기본 타입의 박싱 클래스다.
     - 참조 타입 필드면서, 이 클래스의 ```equals``` 메소드가 이 필드의 ```equals```를 재귀적으로 호출하여 비교한다면, 이 필드의 ```hashCode```를 재귀적으로 호출한다.
     - 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다.
       모든 원소가 핵심 원소라면 ```Arrays.hashCode```를 사용한다.

   - b. 단계 2.a에서 계산한 해시코드 ```c```로 ```result```를 갱신한다.
     ```result = 31 * result + c;```

3. ```result```를 반환한다.

> 다른 필드로부터 계산해낼 수 있는 필드는 모두 무시해도 된다.
>
> ```equals```비교에 사용되지 않는 필드는 반드시 제외해야 한다.

<br>

**✏ #03 예제소스 | 전형적인 hashCode 메서드**

```java
@Override public int hashCode() {
	int result = Short.hashCode(areaCode);
	result = 31 * result + Short.hashCode(prefix);
	result = 31 * result + Short.hashCode(lineNum);
	return result;
}
```

> 비결정적 요소는 전혀 없으므로 동치인 PhoneNumber 인스턴스들은 같은 해시코드를 가진다

*해시 충돌이 더욱 적은 방법은 구아바의 com.google.common.hash.Hashing 를 참고*

<br>

**✏ #04 예제소스 | 한 줄짜리 hashCode 메서드**

```java
@Override public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```

> 코드는 간결하지만 속도가 더 느려 성능이 살짝 아쉽다
>
> 민감하지 않은 상황에서만 사용하자

<br>

**✏ #05 예제소스 | 해시코드를 지연 초기화하는 hashCode 메서드**

```java
@Override public int hashCode() {
	int result = hashCode;
    if (result == 0) {
		result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
		result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
	return result;
}
```

> 필드를 지연 초기화하려면 그 클래스를 스레드 안전을 고려해야 한다

<br>

---

<br>

### 💡 주의사항

**성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다**

> 속도는 빨라지겠지만, 해시 품질이 나빠져 해시테이블 성능이 심각하게 떨어짐
>
> 어떤 필드는 특정 영역에 몰린 인스턴스를 퍼뜨려줄 수도 있다

<br>

**hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자**

> 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀수도 있다
>
> 결함을 발견하거나 더 나은 해시 방식을 알아낸 경우 다음 릴리즈에서 수정

<br>

---

<br>

### 📌 핵심정리

**```equals```를 재정의할 때는 ```hashCode```도 반드시 재정의해야 한다!!**

<br>
