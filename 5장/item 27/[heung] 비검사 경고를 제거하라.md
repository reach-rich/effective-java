Effective Java의 스물일곱 번째 아이템 "비검사 경고를 제거하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 비검사 경고

제네릭을 사용하기 시작하면 수많은 컴파일 경고를 보게 된다. 

- 비검사 형변환 경고
- 비검사 메서드 호출 경고
- 비검사 매개변수화 가변인수 타입 경고
- 비검사 변환 경고

<br>

다음은 잘못 작성한 코드다.

```java
Set<Lark> exaltation = new HashSet();
```

대부분 비검사 경고는 컴파일러가 무엇이 잘못됐는지 친절히 설명해주기 때문에 쉽게 제거할 수 있다. 

```java
Set<Lark> exaltation = new HashSet<Lark>();
```

사실 컴파일러가 알려준 타입 매개변수를 명시하지 않고, Java 7부터 지원하는 다이아몬드 연산자(<>)만으로 해결할 수 있다. 그러면 컴파일러가 올바른 실제 타입 매개변수를 추론해준다.

```java
Set<Lark> exaltation = new HashSet<>();
```

<br>

제거하기 훨씬 어려운 경고도 있다. 곧바로 해결되지 않아도 포기하지 말자. 할 수 있는 한 모든 비검사 경고를 제거하라. 모두 제거한다면 그 코드는 타입 안전성이 보장된다. 즉, 런타임에 ClassCastException이 발생할 일이 없고, 의도한 대로 잘 동작하리라 확신할 수 있다. 

<br>

## 2. @SuppressWarnings

**경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 @SuppressWarnings("unchecked") 어노테이션을 달아 경고를 숨기자.** 단, 타입 안전함을 검증하지 않은 채 경고를 숨기면 스스로에게 잘못된 보안 인식을 심어주는 꼴이다.

@SuppressWarnings 어노테이션은 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 달 수 있다. 하지만 **@SuppressWarnings 어노테이션은 항상 가능한 한 좁은 범위에 적용하자.** 보통은 변수 선언, 아주 짧은 메서드, 혹은 생성자가 될 것이다. 자칫 심각한 경고를 놓칠 수 있으니 절대로 클래스 전체에 적용해서는 안 된다.

<br>

한 줄이 넘는 메서드나 생성자에 달린 @SuppressWarnings 어노테이션을 발견하면 지역변수 선언 쪽으로 옮기자. ArrayList에서 가져온 다음의 toArray 메서드를 예로 생각해보자.

```java
public <T> T[] toArray(T[] a) {
  if (a.length < size) 
    return (T[]) Arrays.copyOf(elements, size, a.getClass()); // unchecked cast
  System.arraycopy(elements, 0, a, 0, size);
  if (a.length > size)
    a[size] = null;
  return a;
}
```

ArrayList를 컴파일하면 이 메서드에서 unchecked cast 경고가 발생한다. 어노테이션은 선언에만 달 수 있기 때문에 return 문에는 @SuppressWarnings를 다는 게 불가능하다. 그렇다고 메서드 전체에 달기에는 필요 이상으로 범위가 넓어진다. 그 대신 반환값을 담을 지역변수를 하나 선언하고 그 변수에 어노테이션을 달아주자.

```java
public <T> T[] toArray(T[] a) {
  if (a.length < size) {
    // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로 올바른 형변환이다.
    @SuppressWarnings("unchecked") T[] result =
      (T[]) Arrays.copyOf(elements, size, a.getClass());
    return result;
  }
  System.arraycopy(elements, 0, a, 0, size);
  if (a.length > size)
    a[size] = null;
  return a;
}
```

<br>

**@SuppressWarnings("unchecked") 어노테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.** 다른 사람이 그 코드를 이해하는 데 도움이 되며, 더 중요하게는 다른 사람이 그 코드를 잘못 수정하여 타입 안전성을 잃는 상황을 줄여준다.

<br>

## 3. 핵심 정리

- 모든 비검사 경고는 런타임에 ClassCastException을 일으킬 수 있는 잠재적 가능성을 뜻하니 최선을 다해 제거하라.
- 경고를 없앨 방법을 찾지 못했다면, 그 코드가 타입 안전함을 증명하고 가능한 한 범위를 좁혀 @SuppressWarnings("unchecked") 어노테이션으로 경고를 숨겨라. 그런 다음 경고를 숨기기로 한 근거를 주석으로 남겨라.
