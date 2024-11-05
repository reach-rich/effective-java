

### ⚙ ```equals```를 재정의하지 않아야 하는 경우

- 각 인스턴스가 본질적으로 고유하다
- 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다
- 상위 클래스에서 정의한 ```equals```가 하위 클래스에도 딱 들어맞는다
- 클래스가 ```private```이거나 ```package-private```이고 ```equals```를 호출할 일이 없다



---



### 🛠 ```equals```를 재정의해야하는 경우

- 논리적 동치성을 확인해야하는데, 상위 클래스의 ```equals```가 논리적 동치성을 비교하도록 재정의되지 않은 경우

- 값 클래스라해도, 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면 ```equals```를 재정의 하지않아도 된다

  ex) ```Enum``` 클래스



---



### ⚖ equals 재정의 규약

equals 메서드는 동치관계(equivalence relation)를 구현하며, 다음을 만족한다.

- 반사성(reflexivity)
- 대칭성(symmetry)
- 추이성(transitivity)
- 일관성(consistency)
- null-아님

이 규약을 어기면 프로그램이 이상작동하거나 종료되어 원인 찾기 힘듬

~~여기까진 규약을 어기면 큰일난다는 것을 알려주는 빌드업 단계~~



**Object명세에서 말하는 동치관계란?**

집합을 서로 같은 원소들로 이뤄진 부분집합으로 나누는 연산

equals메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속하는 어떤 원소와도 교환할 수 있어야한다



#### 1. 반사성

​	**: 객체는 자기 자신과 같아야 한다 ** 

~~앵간해선 어기기 힘듦~~



#### 2. 대칭성

​	**: 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다**

**✏ #01 예제소스 | 대칭성 위배**

```java
public final class CaseInsensitiveString {
    private final String s;
    
    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }
    
    // 대칭성 위배
    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase((CaseInsensitiveString) o).s);
        if (o instanceof String)
            return s.equalsIgnoreCase((String) o).s);
        return false;
    }
}
```

> ```java
> CaseInsensitiveString cis = new CaseInsensitiveString(Polish);
> String s = "polish";
> 
> cis.equals(s) : true;
> s.equals(cis) : false;
> ```
>
> ```CaseInsensitiveString```의 ```equals```는 순진하게 일반 문자열과 비교를 시도한다
>
> ```CaseInsensitiveString```는 ```String``` 을 알고 있지만 ```String```은 ```CaseInsensitiveString```를 알지 못해서 발생하는 것

> ```java
> List<CaseInsensitiveString> list = new ArrayList<>();
> list.add(cis);
> list.continas(s); // false
> ```
>
> 위의 경우 JDK버전에 따라 true를 반환하거나 런타임 에러가 발생할 수 있음

**✏ #02 예제소스 | 대칭성 위배 수정**

```java
// 대칭성 위배
@Override
public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString &&
        ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

**equalse 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다.**



#### 3. 추이성

​	**: A객체와 B객체가 같고, B객체와 C객체가 같으면, A객체와 C객체도 같아야한다**

**✏ #03 예제소스 | 상위 클래스에 없는 새로운 필드를 하위 클래스에 추가하는 경우**

```java
// 상위 클래스
public class Point {
    private final int x;
    private final int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == && p.y == y;
    }
}

// 하위 클래스 (색상 추가)
public class ColorPoint extends Point {
    private final Color color;
    
    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
}
```

**✏ #04 예제소스 | 추이성 위배** 

```java
@Override public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;
    
    // o가 일반 Point면 색상을 무시하고 비교한다.
    if (!(o instanceof ColorPoint))
        return o.equals(this);
    
    // o가 ColorPoint면 색상까지 비교한다.
    return super.equals(o) && ((ColorPoint)) o).color == color;
}
```

> 대칭성은 지켜주지만, 추이성이 깨진 경우이다
>
> ```java
> ColorPoint p1 = new Color Point(1, 2, Color.RED);
> Point p2 = new Point(1, 2);
> ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
> ```
>
> ```p1.equals(p2)```와 ```p2.equals(p3)```는 ```true```를 반환하는데, ```p1.equals(p3)```는 ```false```를 반환한다.
>
> 또 다른 하위 클래스 ```SmellPoint``` 를 만들고 ```myColorPoint.equals(mySmellPoint)```를 호출하면 StackOverflowError 발생
>
> **구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재 X**

> ```java
> @Override public boolean equals(Object o) {
>     if (o == null || o.getClass() != getClass())
>         return false;
>     Point p = (Point) o;
>     return p.x == x && p.y == y;
> }
> ```
>
> 위와 같이 ```equals```안의 ```instanceof``` 검사를 ```getClass```로 검사로 바꾸더라도 **리스코프 치환 원칙**을 위배한다
>
> 
>
> **🔎 리스코프 치환 원칙**
> 		: 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야한다

~~이론과 예제가 제대로 이해되지 않아서 보강해야함~~



#### 4. 일관성

​	**: 두 객체가 같다면 앞으로도 영원히 같아야 한다**

클래스가 불변이든 가변이든 ```equals```의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다

이 제약을 어기면 일관성 조건을 만족시키기가 매우 어렵다

> **예시** 
>
> ```java.net.URL```의 ```equals```는 주어진 URL과 매핑된 호스트 IP 주소를 이용해 비교
>
> 호스트 이름을 IP 주소로 바꾸려면 네트워크를 통해야하는데, 그 결과가 항상 같다고 보장 X



#### 5. null-아님

​	**: 모든 객체가 null과 같지 않아야 한다**

1. ```==```연산자를 사용해 입력이 자기 자신의 참조인지 확인한다

   > 자기 자신이면 ```true```를 반환한다
   >
   > 단순 성능 최적화용으로, 비교 작업이 복잡한 상황일 때 유용

2. ```instanceof```연산자로 입력이 올바른 타입인지 확인한다

   > 올바른 타입은 ```equals```가 정의된 클래스이지만, 그 클래스가 구현한 특정 인터페이스가 될 수 도 있다
   >
   > 어떤 인터페이스는 자신을 구현한 서로 다른 클래스끼리 비교할 수 있도록 ```equals``` 규약을 수정하기도 한다
   >
   > 이런 인터페이스를 구현한 클래스라면 ```equals```에서 해당 인터페이스를 사용해야 한다
   >
   > ```Set, List, Map, Map.Entry``` 등의 컬렉션 인터페이스들이 여기 해당한다.

3. 입력을 올바른 타입으로 형변환한다

4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다

   > 모든 필드가 일치하면 ```true```를, 하나라도 다르면 ```false```를 반환한다



**✏ #05 예제소스 | 규약에 따라 작성한 소스** 

```java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;
    
    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix = rangeCheck(prefix, 999, "프리픽스");
        this.lineNum = rangeCheck(lineNum, 9999, "가입자 번호");
    }
    
    private static short rangeChekc(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }
    
    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix
            	&& pn.areaCode == areaCode
    }
}
```



**대칭적인가? 추이성이 있는가? 일관적인가?**



---



### ✔ 주의사항

- ```equals```를 재정의할 땐 ```hashCode```도 반드시 재정의하자

- 너무 복잡하게 해결하려 들지마라

  > 필드들의 동치성만 검사해도 ```equals``` 규약을 어렵지 않게 지킬 수 있음
  >
  > 별칭(alias)는 비교하지 않는게 좋다

- ```Object``` 외의 타입을 매개변수로 받는 ```equals``` 메서드는 선언하지 말자

  > ```java
  > public boolean equals(MyClass o) {
  > 	...
  > }
  > ```
  >
  > 이 메서드는 ```Object.equals```를 재정의한 게 아니다
  >
  > 입력 타입이 ```Object```가 아니므로 재정의가 아니라 다중정의한 것
  >
  > ```@Override```어노테이션을 일관되게 사용하면 이러한 실수를 예방할 수 있다



---



### 📌 핵심 정리

**꼭 필요한 경우가 아니면 ```equals```를 재정의하지 말자!!
많은 경우에 Object의 equals가 원하는 비교를 정확히 수행해준다
재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야 한다**
