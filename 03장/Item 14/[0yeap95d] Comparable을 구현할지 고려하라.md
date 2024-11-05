### 🔍Comparable 구현



- ```compareTo```는 ```Object```의 메서드가 아니다

- 성격은 두 가지만 빼면 ```Object```의 ```equals```와 같다

- ```compareTo```는 단순 동치성 비교에 더해 순서까지 비교할 수 있으며, 제너릭하다

<br>

> **📝 compareTo 메서드의 일반 규약**
>
> - `Compareable`을 구현한 클래스는 모든 ```x```, ```y```에 대해 ```sgn(x.compareTo(y)) == -sgn(y.compareTo(x))```여야 한다(따라서 ```x.compareTo(y)```는 ```y.compareTo(x)```가 예외를 던질때에 한해 예외를 던져야 한다).
>
> - ```Comparable```을 구현한 클래스는 추이성을 보장해야 한다. 즉, ```(x.compareTo(y) > 0 && y.compareTo(z) > 0)```이면 ```x.compareTo(z) > 0``` 이다.
>
> - ```Comparable```을 구현한 클래스는 모든 ```z```에 대해 ```x.compareTo(y) == 0```이면 ```sgn(x.compareTo(z)) == sgn(y.compareTo(z))```다.
>
> - 이번 권고가 필수는 아니지만 꼭 지키는 게 좋다. ```(x.compareTo(y) == 0) == (x.equals(y))```여야 한다. ```Comparable```을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다. 다음과 같이 명시하면 적당할 것이다.
>
>   "주의: 이 클래스의 순서는 ```equals``` 메서드와 일관되지 않다."

<br>

정렬된 컬렉션인 ```TreeSet```과 ```TreeMap```

검색과 정렬 알고리즘을 활요하는 유틸리티 클래스 ```Collections```와 ```Arrays```

<br>

---

<br>

**✔ equals 규약과 똑같이 반사성, 대칭성, 추이성 충족**

>**[첫번째 규약]** 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다
>
>**[두번째 규약]** a가 b보다 크고 b가 c보다 크면 a는 c보다 커야한다
>
>**[마지막 규약]** 같은 객체끼리는 어떤 객체와 비교하더라도 항상 같아야한다
>
>- ```compareTo```의 마지막 규약은 필수는 아니지만 꼭 지키길 권한다
>- ```compareTo```와 ```equals``` 결과가 같아야 한다 
>- 두개 의 결과가 일치하지 않아도 작동을 할테지만 정렬된 컬렉션들은 동치성을 비교할때 ```equals``` 대신 ```compareTo```를 사용하기 때문이다

<br>

**✔ equals 주의사항과 우회법도 똑같다**

> **[주의사항]**
>
> 기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가했다면 ```compareTo``` 규약을 지킬 방법이 없다

>**[우회법]**
>
>```Comparable```을 구현한 클래스를 확장해 값 컴포넌트를 추가하고 싶다면, 확장하는 대신 독립된 클래스를 만들고, 이 클래스에 원래 클래스의 인스턴스를 가리키는 필드를 둔다
>그런 다음 내부 인스턴스를 반환하는 '뷰' 메서드를 제공하면 된다

<br>

---

<br>

### 💡 compareTo 메서드 작성요령

```equals```와 거의 비슷하지만 몇 가지 차이점을 주의

- ```Comparable```은 타입을 인수로 받는 제너릭 인터페이스이므로 ```compareTo```메서드의 인수타입은 컴파일타임에 정해진다
- 입력 인수의 타입을 확인하거나 형변환 필요 x
- 인수 타입 오류시 컴파일 자체가 x
- ```null```을 인수를 넣어 호출하면 ```NullPointerException``` 발생해야함

<br>

**✏ #01 예제소스 | 객체 참조 필드가 하나뿐인 비교자**

```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
    ...
}
```

>```CaseInsensitiveString```의 참조는 ```CaseInsensitiveString``` 참조와만 비교할 수 있다는 뜻
>
>```Comparable```을 구현할 때 일반적으로 따르는 패턴

<br>

**✏ #02 예제소스 | 기본 타입 필드가 여럿일 때의 비교자**

```java
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if (result == 0) {
        result = Short.compare(prefix, pn.prefix);
        if (result == 0)
            result = Short.compare(lineNum, pn.lineNum);
    }
    return result;
}
```

>```compareTo``` 메서드에서 관계 연산자 ```<```와 ```>```를 사용하는 이전 방식은 추천하지 않는다

<br>

**✏ #03 예제소스 | 비교자 생성 메서드를 활용한 비교자**

```java
private static final Comparator<PhoneNumber> COMPARATOR =
    comparingInt((PhoneNumber pn) -> pn.areaCode)
    	.thenComparingInt(pn -> pn.prefix)
    	.thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```

>자바 8에서는 ```Comparator``` 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자 생성 가능
>
>간결한 방식이지만, 약간의 성능 저하가 발생
>
>```comparingInt```는 객체 참조를 ```int``` 타입 키에 매핑하는 키 추출 함수를 인수로 받아, 그 키를 기준으로 순서를 정하는 비교자를 반환하는 정적 메서드

<br>

**✏ #04 예제소스 | 해시코드 값의 차를 기준으로 하는 비교자 - 추이성을 위배한다!**

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
	public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
}
```

>이 방식은 정수 오버플로를 일으키거다 부동소수점 계산 방식에 따라 오류가 발생한다
>
>월등히 빠르지도 않기에 아래의 두 방식 중 하나를 사용해라

<br>

**✏ #05 예제소스 | 정적 compare 메서드를 활용한 비교자**

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
}; 
```

**✏ #06 예제소스 | 비교자 생성 메서드를 활요한 비교자**

```java
static Comparator<Object> hashCodeOrder = 
    Comparator.comparingInt(o -> o.hashCode());
```

<br>

---

<br>

### 📌 핵심정리

**순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여, 그 인터페이스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야 한다.**

**compareTo 메서드에서 필드의 값을 비교할 때 <와 > 연산자는 쓰지말아야한다.**

**그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.**
