## 1. 들어가기

우리가 자주 사용하는 컨테이너에서는 제네릭이 흔히 쓰입니다.

하지만, 하나의 컨테이너는 보통 사용할 수 있는 타입의 수가 제한됩니다.

예를 들면 Set은 한 개의 타입, Map은 두 개의 타입을 사용할 수 있습니다.

그 이상은 사용할 수 없는 것이죠.

그럼 여러 타입을 사용하는 방법은 없을까요?

## 2. 타입 안전 이종 컨테이너 패턴

제네릭을 여러 타입으로 사용하는 방법은 **타입 안전 이종 컨테이너 패턴**으로 해결할 수 있습니다.

타입 안전 이종 컨테이너 패턴이란, 컨테이너 자체에 매개변수화를 하는 것이 아닌

키를 매개변수화 해서 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하는 패턴입니다.

```java
  public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();  // 키를 매개변수화

    public <T> void putFavorite(Class<T> type, T instance) {
      favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
      return type.cast(favorites.get(type));
    }
  }

  public static void main(String[] args) {
    Favorites f = new Favorites();

    f.putFavorite(String.class, "Java");
    f.putFavorite(Integer.class, 1234);
    f.putFavorite(Class.class, Favorites.class);

    String favoriteString = f.getFavorite(String.class);
    int favoriteInteger = f.getFavorite(Integer.class);
    Class<?> favoriteClass = f.getFavorite(Class.class);

    System.out.println(favoriteString);
    System.out.println(favoriteInteger);
    System.out.println(favoriteClass);
  }
```

```
  Java
  1234
  class Main$Favorites
```

## 3. 의문점

앞의 예시는 두 가지의 의문점이 있습니다.

1. 일반적인 경우 와일드 카드 타입에는 null 외에 다른 값을 넣을 수 없는데 어떻게 값을 넣을 수 있을까요?

   Favorites 클래스의 Map 컬렉션을 자세히 보면 와일드카드 타입이 중첩된 것을 볼 수 있습니다.

   즉, Map 자체가 와일드카드가 아닌, 키 값이 와일드카드이기 때문에 가능합니다.

2. Map의 값이 Object 타입이면 키와 값 사이 타입 관계를 보증하지 못하는 것이 아닐까요?

   먼저, putFavorite 메서드를 살펴보겠습니다.

   ```java
    public <T> void putFavorite(Class<T> type, T instance) {
      favorites.put(Objects.requireNonNull(type), instance);
    }
   ```

   putFavorite 메서드는 Map 컬렉션에 키와 값을 넣는 메서드입니다.
   
   매개변수 부분을 보면 Class 매개변수화 타입과 인스턴스의 타입이 동일합니다.

   그렇기에 putFavorite 메서드 사용 시 키로 지정한 타입과 동일하지 않으면 컴파일 오류가 발생합니다.

   getFavorite 메서드도 함께 살펴보겠습니다.

   ```java
    public <T> T getFavorite(Class<T> type) {
      return type.cast(favorites.get(type));
    }
   ```

   현재 Map 컬렉션의 값에 해당하는 부분은 Object 타입입니다. 

   그래서 사용 시에 Object를 키에 해당하는 값으로 바꿔 반환해야 합니다.

   Class의 cast 메서드는 Class 객체가 가리키는 타입으로 동적 형변환합니다.

   우리는 putFavorite 메서드에서 무조건 키와 동일한 타입이 값으로 들어간다는 것을 알고 있습니다.

   그러므로 Class의 cast 메서드를 통해 안전하게 형변환해 사용할 수 있습니다.

## 4. 제약

Favorites 클래스에는 제약이 두 가지 있습니다.

1. Class 객체를 raw 타입으로 넘기면 타입 안전성이 쉽게 깨진다.

   ```java
    f.putFavorite((Class)Integer.class, "악의적인 코드");
    int favoriteInteger = f.getFavorite(Integer.class);
   ```

   악의적인 클라이언트가 다음과 같이 작성한다면
   
   컴파일 오류는 발생하지 않지만 ClassCastException이 발생합니다.

   이 정도의 문제를 감수하겠다면 Class의 cast 메서드로 런타임 시 타입 안전성을 확보할 수 있습니다.

   ```java
    public <T> void putFavorite(Class<T> type, T instance) {
      favorites.put(Objects.requireNonNull(type), type.cast(instance));
    }
   ```

2. 실체화 불가 타입에는 사용할 수 없다.

   String이나 Integer와 같이 실체화할 수 있는 타입은 String.class, Integer.class로 저장할 수 있지만

   List\<String>이나 List\<Integer>와 같이 실체화할 수 없는 타입은 사용할 수 없습니다.

   List\<String>과 List\<Integer>는 같은 List.class를 공유하므로

   이를 사용한다면 Favorites 객체의 내부는 아수라장이 될 것입니다.

   이 제약에 대한 우회로로 슈퍼 타입 토큰을 제시했으나 이 또한 만족스러운 우회로는 아닙니다.

## 5. 한정적 타입 토큰

Favorites 클래스가 사용하는 타입 토큰은 비한정적입니다.

즉, getFavorite 메서드와 putFavorite 메서드는 어떤 Class 객체든 받아들입니다.

때론, 이 메서드들이 허용하는 타입을 제한하고 싶을 수 있는데 한정적 타입 토큰을 활용하면 가능합니다.

다음은 AnnotatedElement 인터페이스에서 사용한 한정적 타입 토큰 예시입니다.

```java
  <T extends Annotation> T getAnnotation(Class<T> annotationClass);
```

이 메서드는 토큰으로 명시한 타입의 어노테이션이 대상 요소에 달려있으면

그 어노테이션을 반환하고 아니면 null을 반환합니다.

즉, 어노테이션된 요소는 그 키가 어노테이션 타입인 타입 안전 이종 컨테이너입니다.

이 메서드에 매개변수를 넘기기 위해 객체를 Class<? extends Annotation>으로 형변환할 수도 있지만,

이것은 비검사이므로 컴파일 시 경고가 뜰 것입니다.

다행히도 Class 클래스는 안전하게 형변환해주는 메서드를 제공합니다.

바로, asSubclass 메서드로 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환해줍니다.

형변환에 성공하면 해당 클래스 객체를 반환하고, 실패하면 ClassCastException이 발생합니다.

```java
  static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
    Class<?> annotationType = null;

    try {
      annotationType = Class.forName(annotationTypeName);
    } catch(Exception e) {
      throw new IllegalArgumentException(e);
    }

    return element.getAnnotation(annotationType.asSubClass(Annotation.class));
  }
```

## 6. 정리

이번 포스트는 타입 안전 이종 컨테이너 패턴에 대해 알아보았습니다.

일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입이 한정되어 있지만,

타입 안전 이종 컨테이너 패턴을 사용하면 여러 타입을 가진 컨테이너를 만들 수 있습니다.