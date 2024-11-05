Effective Java의 여섯 번째 아이템 "불필요한 객체 생성을 피하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 0. 들어가며

똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을 때가 많습니다. 본문에서는 불필요한 객체를 생성하는 예를 알아보고, 이를 개선해 봅니다.

<br>

## 1. 불필요한 객체 생성 예

### 1.1. String

```java
String s = new String("heung");
```

이 코드는 실행될 때마다 String 인스턴스를 새로 만듭니다. 반복문이나 빈번히 호출되는 메서드 안에 있다면, 쓸데없는 String 인스턴스가 수백만 개 만들어질 수도 있습니다. 

```java
String admin = new String("heung");
String user = new String("heung");

System.out.println(admin == user); // false
```

우리는 생성자에 넘겨진 `"heung"` 자체가 이 생성자로 만들어내려는 String과 기능적으로 완전히 동일하다는 것을 알고 있습니다. 따라서 다음과 같이 개선할 수 있습니다.	

```java
String s = "heung";
```

이 코드는 매번 새로운 인스턴스를 만드는 대신 하나의 String 인스턴스를 사용합니다. 나아가 이 방식을 사용하면 같은 가상 머신 안에서 이와 똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함이 보장됩니다. 

```java
String admin = "heung";
String user = "heung";

System.out.println(admin == user); // true
```

<br>

### 1.2. 불변 클래스

다음은 불변 클래스 Boolean 타입의 객체를 생성하는 예제입니다.

```java
Boolean b1 = new Boolean(true);
Boolean b2 = new Boolean(true);

System.out.println(b1 == b2); // false
```

보이는 것과 같이 생성자는 호출할 때마다 새로운 객체를 만듭니다. 쓸모없는 인스턴스가 여러 개 만들어질 수 있는 것이죠. 따라서 Boolean은 정적 팩터리 메서드를 제공합니다(실제로 Boolean의 생성자는 Java 9에서 deprecated API로 지정되었습니다).

```java
public final class Boolean {

    public static final Boolean TRUE = new Boolean(true);

    public static final Boolean FALSE = new Boolean(false);
  
    ...
  
    public static Boolean valueOf(String s) {
      return parseBoolean(s) ? TRUE : FALSE;
    }
  
    ...
}
```

```java
Boolean b1 = Boolean.valueOf(true);
Boolean b2 = Boolean.valueOf(true);

System.out.println(b1 == b2); // true
```

**생성자 대신 정적 팩터리 메서드를 제공하는 불변 클래스에서는 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있습니다.** 또한 불변 객체뿐만 아니라 가변 객체라 해도 사용 중에 변경되지 않을 것임을 안다면 재사용할 수 있습니다.

<br>

### 1.3. 비싼 객체

생성 비용이 아주 큰 객체도 더러 있습니다. 이런 **'비싼 객체'가 반복해서 필요하다면 캐싱하여 재사용하길 권합니다.** 다음은 주어진 문자열이 유효한 로마 숫자인지를 확인하는 메서드입니다.

```java
public class RomanNumerals {
  
    static boolean isRomanNumeral(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }
}
```

이 방식은 문제는 String.matches 메서드를 사용한다는 것에 있습니다. String.matches는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복해 사용하기엔 적합하지 않습니다. 이 메서드가 내부에서 만드는 정규표현식용 Pattern 인스턴스는, 한 번 쓰고 버려져서 곧바로 GC의 대상이 됩니다. Pattern은 입력받은 정규표현식에 해당하는 유한 상태 머신을 만들기 때문에 인스턴스 생성 비용이 높습니다. 

다음은 캐싱을 사용해 개선한 코드입니다.

```java
public class RomanNumerals {
  
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

정규표현식을 포현하는 (불변인) Pattern 인스턴스를 클래스 초기화(정적 초기화) 과정에서 직접 생성해 캐싱해두고, 나중에 isRomanNumeral 메서드가 호출될 때마다 이 인스턴스를 재사용합니다. 이렇게 하면 isRomanNumeral가 빈번히 호출되는 상황에서 성능을 상당히 끌어올릴 수 있습니다. 

다음은 성능 비교를 위해 작성한 테스트 코드입니다.

```java
long start = System.currentTimeMillis();
for (int j = 0; j < 1000000; j++) {
  isRomanNumeral("MCMLXXVI");
}
long end = System.currentTimeMillis();

System.out.println(end - start);
```

isRomanNumeral 메서드를 백만 번 호출하여 런타임 시간을 측정합니다. 참고로 `MCMLXXVI`는 로마 숫자이며 1976를 의미합니다.

```java
캐싱을 사용하지 않은 코드 : 647 ms
캐싱을 사용한 코드 : 139 ms
```

코드 개선(캐싱)을 통해 약 4.5배 빨라진 것을 확인할 수 있었습니다. 

성능과는 별개로 한 가지 더 이점이 있습니다. 코드가 더 명확해졌다는 것입니다. 개선 전에는 존재조차 몰랐던 Pattern 인스턴스를 static final 필드로 끄집어내고 이름을 지어주어 코드의 의미가 훨씬 잘 드러납니다. 

<br>

### 1.4. 오토박싱

오토박싱(auto boxing)은 프로그래머가 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해 주는 기술입니다. 그런데 오토박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아닙니다. 의미상으로는 별다를 것 없지만 성능에서는 그렇지 않습니다.

다음은  `Integer.MAX_VALUE`까지의 총합을 구하는 예제입니다.

```java
private static long getSum() {
  Long sum = 0L;
  for (long i = 0; i <= Integer.MAX_VALUE; i++)
    sum += i;
  
  return sum;
}
```

long 타입의 i가 Long 타입으로 오토박싱이 일어납니다. 변수 sum을 long이 아닌 Long으로 선언해서 불필요한 Long 인스턴스가 약 2^31개나 만들어진 것입니다. 다음은 성능 비교를 위한 테스트 코드입니다.

```java
long start = System.currentTimeMillis();
sum();
long end = System.currentTimeMillis();

System.out.println(end - start);
```

위의 getSum 메서드에서 변수 sum의 타입이 Long 일 때와 long 일 때의 런타임 시간을 비교해보겠습니다.

```java
Long sum 일 때 : 2963 ms
long sum 일 때 : 692 ms
```

약 4.2배 빨라졌네요. 따라서, 우리는 **박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 유의해야 합니다.**

<br>

## 2. 주의할 점

이번 아이템을 "객체 생성은 비싸니 피해야 한다"로 오해하면 안 됩니다. 특히나 요즘의 JVM에서는 별다른 일을 하지 않는 작은 객체를 생성하고 회수하는 일이 크게 부담되지 않습니다. 프로그램의 명확성, 간결성, 기능을 위해서 객체를 추가로 생성하는 것이라면 일반적으로 좋은 일입니다.

거꾸로, 아주 무거운 객체가 아닌 다음에야 단순히 객체 생성을 피하고자 자신만의 객체 풀(pool)을 만드는 것은 좋지 않습니다. 물론 객체 풀을 만드는 것이 나을 때도 있습니다. 하지만 일반적으로는 자체 객체 풀은 코드를 헷갈리게 만들고 메모리 사용량을 늘리고 성능을 떨어뜨립니다. 요즘 JVM의 GC는 상당히 잘 최적화되어서 가벼운 객체용을 다룰 때는 직접 만든 객체 풀보다 훨씬 빠릅니다.

<br>

## 3. Related Posts

- 불변 (Item 17)
- [정적 팩터리 메서드 (Item 1)](https://heung27.github.io/posts/effective-java-item-1-%EC%83%9D%EC%84%B1%EC%9E%90-%EB%8C%80%EC%8B%A0-%EC%A0%95%EC%A0%81-%ED%8C%A9%ED%84%B0%EB%A6%AC-%EB%A9%94%EC%84%9C%EB%93%9C%EB%A5%BC-%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC/)
- 기본 타입과 박싱된 기본 타입 (Item 61)
