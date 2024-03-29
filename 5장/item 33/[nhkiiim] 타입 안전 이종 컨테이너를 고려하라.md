# 타입 안전 이종 컨테이너를 고려하라

### 1. 타입 안전 이종 컨테이너
- 제네릭은 `Set<E\>, Map<K,V\>` 와 같은 컬렉션이나 `TreadLocal<T>, AutomicReference<T>`등의 단일 컨테이너에도 흔히 쓰임
- 이러 쓰임에서 매개변수화 되는 대상은 원소가 아닌 컨테이너 자신! -> Set의 타입!! Map의 키, 값의 타입 처럼 타입을 뜻하는 애들만 필요함

<br>

- 이럴 때 말고 더 유연한 수단이 필요할 때도 있음
- ex) 데이터베이스 한 행을 읽을 때 다양한 열의 타입을 안전하게 이용할 수 있다면 멋질 것 
- 이를 위해 컨테이너 대신 키를 매개변수화 한 다음 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공할 수 있음

- 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장할 것
- 이러한 설계방식을 타입 안전 이종 컨테이너 패턴이라고 함

#
### 2. Favorites 클래스 예시
__1) 타입별로 즐겨찾는 인스턴스를 저장하고 검색할 수 있는 Favorite 클래스__

- 각 타입의 Class 객체를 매개변수화한 키 역할로 사용할거임
- class 리터럴(String.class 같은 거)의 타입은 Class 가 아닌 Class<T\> (Class<String\>)
- 컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴을 타입 토큰(type token)이라고 함

```java
public class Favorites {
  public <T> void putFavorite(Class<T> type, T instance);
  public <T> T getFavorite(Class<T> type);
}
```

```java
public static void main(String[] args) {
  Favorites f = new Favorites();
  
  f.putFavorite(String.class, "Java");
  f.putFavorite(Integer.class, 1234);
  f.putFavorite(Class.class, Favorites.class);

  String favoriteString = f.getFavorite(String.class);
  int favoriteInteger = f.getFavorite(Integer.class);
  Class<?> favoriteClass = f.getFavorite(Class.class);

  System.out.printf("%s %d %s%n", favoriteString, favoriteInteger, favoriteClass.getName());
}
```
- Favorites 인스턴스는 타입 안전하며 일반적인 맵과 달리 여러 가지 타입의 원소를 담을 수 있음!
- 타입 안전 이종 컨테이너의 예시가 됨


<br>

__2) 타입 안전 이종 컨테이너 - 구현__

```java
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

- Map<Class<?\>, Object\> 은 맵이 아니라 키가 비한정적인 와일드카드 타입
- 모든 키가 서로 다른 매개변수화 타입일 수 있게 됨!
- 맵의 값 타입은 단순한 Object 이기 때문에 키로 명시한 타입인지 보증하진 않음

  - (어떤게 들어올지 모르는 상황이니 ? 로 받아만 주는 것 : Map<?,?> 로 선언할 순 없겠지만 키가 ?라 OK)
  - 그렇다면 T 이렇게 안쓰고 ?쓰는 이유는? 까먹었으니 스터디 시간에 물어보기

<br>

- putFavorite 구현은 값이 키 타입이라는 타입 링크는 끊어지지만 getFavorite에서 다시 살아나서 상관 없음

<br>

- getFavorite는 맵에서 꺼낸 Object 객체를 T 타입으로 바꿔야함
- Class의 cast 메서드를 사용해 이 객체 참조를 class 객체가 가리키는 타입으로 동적 변환

<br>

- cast 메서드는 형변환 연산자의 동적 버전
- 주어진 인수가 Class 객체가 알려주는 타입의 인스턴스인지 검사한 다음, 맞다면 그대로 반환하고 아니면 ClassCastException을 던짐
- 맵 안의 값이 해당 키의 타입과 항상 일치함을 보장

#
### 3. Favorites 클래스의 제약
__1) 악의적인 클라이언트가 Class객체를 로타입으로 넘길 때 인스턴스의 타입 안전성이 쉽게 깨짐__
- 이러한 코드는 컴파일 시 비검사 경고가 뜰 것
- 동적 형변환을 쓰면 타입 불변식을 어기는 일이 없도록 보장 가능

```java
public <T> void putFavorite(Class<T> type, T instance) {
  favorites.put(Objects.requireNonNull(type), type.cast(instance));
}
```
- java.util.Collections 에 checkedSet(), checkedList(), checkedMap() 메서드들도 해당 방식을 적용한 컬렉션 래퍼

__2) 실체화 불가 타입에는 사용 불가__
- String, String[] 는 저장할 수 있지만 List<String\>은 불가 (String.class 가 아니라 List.class임)
- 이 제약은 우회할 완벽한 방법은 없음
- 슈퍼타입 토큰으로 해결하려는 시도가 있으나 주의해서 사용해야함

#
### 4. 한정적 토큰과 사용
- Favorites가 사용하는 타입 토큰은 비한정적 -> 어떤 Class 객체도 OK
- 허용하는 타입을 제한하고 싶다면, 한정적 타입 토큰 활용 가능

```java
public <T extends Annotation> T getAnnotation(Class<T> annotationType);
```  

- 런타임에 어노테이션 읽어오는 기능
- 명시한 타입의 어노테이션이 대상 요소에 달려 있다면 그 애너테이션을 반환, 없다면 Null을 반환 (타입 안전 이종 컨테이너)

<br>

__* Class<?\> 타입의 객체를 한정적 타입 토큰을 받는 메서드에 넘기는 방법__

- Class<? extends Annotation\> 으로 형변환 할 수 있지만 비검사 경고가 뜰 것
- asSubClass 메서드를 통해 안전하고 동적으로 형변환 가능

```java
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
    Class<?> annotationType = null;

    try {
        annotationType = Class.forName(annotationTypeName);
        //Class.forName()은 컴파일 타임에 직접적인 참조 없이 런타임에 동적으로 클래스를 로드
    } catch (Exception ex) {
        throw new IllegalArgumentException(ex);
    }

    return element.getAnnotation(annotationType.asSubclass(Annotation.class));
}
```
- asSubClass() : 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환
