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
### 2. 예시
- 타입별로 즐겨찾는 인스턴스를 저장하고 검색할 수 있는 Favorite 클래스를 생각해보자
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

