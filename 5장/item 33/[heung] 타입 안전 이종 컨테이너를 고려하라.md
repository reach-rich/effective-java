Effective Java의 서른세 번째 아이템 "타입 안전 이종 컨테이너를 고려하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 타입 안전 이종 컨테이너 패턴

제네릭은 매개변수화되는 대상이 원소가 아닌 컨테이너 자신이다. 따라서 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한된다. 예를 들어 set에는 원소의 타입을 뜻하는 단 하나의 타입 매개변수만 있으면 되며, Map에는 키와 값의 타입을 뜻하는 2개만 필요한 식이다.

하지만 더 유연한 수단이 필요할 때도 종종 있다. 예를 들어 데이터베이스의 행(row)은 임의 개수의 열(column)을 가질 수 있는데, 모두 열을 타입 안전하게 이용할 수 있다면 멋질 것이다. 다행히 쉬운 해법이 있다. **컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공**하면 된다. 이렇게 하면 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장해줄 것이다. 이러한 설계 방식을 **타입 안전 이종 컨테이너 패턴(type safe heterogeneous container pattern)**이라 한다.

<br>

## 2. 예제 - favorites 클래스

간단한 예로 타입별로 즐겨 찾는 인스턴스를 저장하고 검색할 수 있는 Favorites 클래스를 생각해보자. 각 타입의 Class 객체를 매개변수화한 키 역할로 사용하면 되는데, 이 방식이 동작하는 이유는 class의 클래스가 제네릭이기 때문이다. 이 프로그램은 즐겨 찾는 String, Integer, Class 인스턴스를 저장, 검색, 출력한다.

```java
// 타입 안전 이종 컨테이너 패턴 - API
public class Favorites {
  public <T> void putFavorite(Class<T> type, T instance);
  public <T> T getFavorite(Class<T> type);
}
```

```java
// 타입 안전 이종 컨테이너 패턴 - 클라이언트
public static void main(String[] args) {
  Favorites f = new Favorites();
  
  f.putFavorite(String.class, "Java");
  f.putFavorite(Integer.class, 0xcatebabe);
  f.putFavorite(Class.class, Favorites.class);
  
  String favoriteString = f.getFavorite(String.class);
  int favoriteInteger = f.getFavorite(Integer.class);
  Class<?> favoriteClass = f.getFavorite(Class.class);
  
  System.out.printf("%s %x %s%n", favoriteString,
                   favoriteInteger, favoriteClass.getName());
}
```

Favorites 인스턴스는 타입 안전하다. String을 요청했는데 Integer를 반환하는 일은 절대 없다. 또한 모든 키의 타입이 제각각이라, 일반적인 맵과 달리 여러 가지 타입의 원소를 담을 수 있다. 따라서 Favorites는 타입 안전 이종 컨테이너라 할 만하다.

>class의 리터럴 타입은 Class가 아닌 Class\<T>다.
>
>컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴을 **타입 토큰(type token)**이라 한다. 

<br>

favorites의 구현은 간단하다.

```java
// 타입 안전 이종 컨테이너 패턴 - 구현
public class Favorites {
  private Map<Class<?>, Object> favorites = new HashMap<>();
  
  public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), instance);
  }
  
  public <T> T getFavorite(Class<T> type) {
    return type.cast(favorites.get(type));
  }
}
```

### Point 1. favorites 필드

**1) Favorites가 사용하는 private 맵 변수인 favorites의 타입은 Map\<Class\<?>, Object>이다.** 

비한정적 와일드카드 타입이라 이 맵 안에 아무것도 넣을 수 없다고 생각할 수 있지만, 사실은 그 반대다. 와일드카드 타입이 중첩되었다는 점을 깨달아야 한다. 맵이 아니라 키가 와일드카드 타입인 것이다. 이는 모든 키가 서로 다른 매개변수화 타입일 수 있다는 뜻이며, 다양한 타입을 지원하는 힘이 여기서 나온다.

**2) favorites 맵의 값 타입은 단순히 Object이다.**

이 맵은 키와 값 사이의 타입 관계를 보증하지 않는다는 말이다. 즉, 모든 값이 키로 명시한 타입임을 보증하지 않는다. 사실 자바의 타입 시스템에서는 이 관계를 명시할 방법이 없다. 하지만 우리는 이 관계가 성립함을 알고 있고, 즐겨찾기를 검색할 때 그 이점을 누리게 된다.

<br>

### Point 2. putFavorite 메서드

주어진 Class 객체와 즐겨찾기 인스턴스를 favorites에 추가해 관계를 짓는다. 여기서 키와 값 사이의 '타입 링크(type linkage)' 정보는 버려진다. 즉, 그 값이 그 키 타입의 인스턴스라는 정보가 사라진다. 

<br>

### Point 3. getFavorite 메서드

주어진 Class 객체에 해당하는 값을 favorites 맵에서 꺼낸다. 이 객체가 바로 반환해야 할 객체가 맞지만, 잘못된 컴파일타임 타입을 가지고 있다. 이 객체의 타입은 Object이나, 우리는 이를 T로 바꿔 반환해야 한다. 따라서 Class의 cast 메서드를 사용해 이 객체 참조를 Class 객체가 가리키는 타입으로 동적 형변환한다.

<br>

### 제약 1. 악의적인 클라이언트가 Class 객체를 로 타입으로 넘기면 Favorites 인스턴스의 타입 안정성이 쉽게 깨진다.

```java
f.putFavorite((Class)Integer.class, "Integer의 인스턴스가 아닙니다.");
int favoriteInteger = f.getFavorite(Integer.class);
```

putFavorite을 호출할 때는 아무 불평이 없다가 getFavorite을 호출할 때 ClassCastException을 던진다. 하지만 아래와 같이 인수로 주어진 instance의 타입이 type으로 명시한 타입과 같은지 확인하면 해결할 수 있다. 

```java
public <T> void putFavorite(Class<T> type, T instance) {
  favorites.put(Objects.requireNonNull(type), type.cast(instance));
}
```

<br>

### 제약 2. 실체화 불가 타입에는 사용할 수 없다.

다시 말해, 즐겨 찾는 String이나 String[]은 저장할 수 있어도 즐겨 찾는 List\<String>은 저장할 수 없다. List\<String>을 저장하려는 코드는 컴파일되지 않을 것이다. List\<String>용 Class 객체를 얻을 수 없기 때문이다. List\<String>과 List\<Integer>는 List.class라는 같은 Class 객체를 공유한다.

두 번째 제약을 슈퍼 타입 토큰으로 해결하려는 시도도 있다. 실제로 아주 유용하여 스프링 프레임워크에서는 아예 PrameterizedTypeReference라는 클래스로 미리 구현해놓았다. 

본문 예제의 Favorites에 슈퍼 타입 토큰을 적용하면 다음 코드처럼 제네릭 타입도 문제없이 저장할 수 있다.

```java
Favorites f = new Favorites();
List<String> pets = Arrays.asList("개", "고양이", "앵무");

f.putFavorite(new TypeRef<List<String>>(){}, pets);
List<String> listOfStrings = f.getFavorite(new TypeRef<List<String>>(){});
```

단, 슈퍼 타입 토큰도 완벽하지는 않으니 주의해서 사용해야 한다. 이 방식의 한계는 를 참고하자.

<br>

## 3. 예제 - 어노테이션 API

한정적 타입 토큰이란 단순히 한정적 타입 매개변수나 한정적 와일드카드를 사용하여 표현 가능한 타입을 제한하는 타입 토큰이다.

어노테이션 API는 한정적 타입 토큰을 적극적으로 사용한다. 예를 들어 다음은 AnnotatedElement 인터페이스에 선언된 메서드로, 대상 요소에 달려 있는 어노테이션을 런타임에 읽어 오는 기능을 한다. 이 메서드는 리플렉션의 대상이 되는 타입들, 즉 클래스(java.lang.Class\<T>), 메서드(java.lang.reflect.Method), 필드(java.lang.reflect.Field) 같이 프로그램 요소를 표현하는 타입들에서 구현한다.

```java
public <T extends Annotation> T getAnnotation(Class<T> annotationType);
```

여기서 annotationType 인수는 어노테이션 타입을 뜻하는 한정적 타입 토큰이다. 이 메서드는 토큰으로 명시한 타입의 어노테이션이 대상 요소에 달려 있다면 그 어노테이션을 반환하고, 없다면 null을 반환한다. 즉, 어노테이션된 요소는 그 키가 어노테이션 타입인, 타입 안전 이종 컨테이너인 것이다.

Class\<?> 타입의 객체가 있고, 이를 한정적 타입 토큰을 받는 메서드에 넘기려면 어떻게 해야 할까? Class 클래스가 이런 형변환을 안전하게 그리고 동적으로 수행해주는 인스턴스 메서드를 제공한다. 바로 asSubclass 메서드로, 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환한다.

다음은 컴파일 시점에는 타입을 알 수 없는 어노테이션을 asSubclass 메서드를 사용해 런타임에 읽어내는 예다.

```java
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
  Class<?> annotationType = null;
  try {
    annotationType = Class.forName(annotationTypeName);
  } catch (Exception ex) {
    throw new IllegalArgumentException(ex);
  }
  return element.getAnnotation(annotationType.asSubclass(Annotation.class));
}
```

<br>

## 4. 핵심 정리

- 컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있다.
- 하지만 컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨테이너를 만들 수 있다.
- 타입 안전 이종 컨테이너는 Class를 키로 쓰며, 이런 식으로 쓰이는 Class 객체를 타입 토큰이라 한다. 또한, 직접 구현한 키 타입도 쓸 수 있다.

<br>

## 5. Related Posts

* 로 타입 (Item 26)
* 실체화 불가 타입 (Item 28)
* 한정적 타입 매개변수 (Item 29)
* 한정적 와일드카드 (Item 31)
* 어노테이션 API (Item 39)

