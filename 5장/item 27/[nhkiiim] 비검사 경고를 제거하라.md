# 비검사 경고를 제거하라 

### 1. 비검사 경고 
- 제네릭을 사용하기 시작하면 많은 컴파일러 경고들을 보게 됨
- 비검사 형변환 경고, 비검사 메서드 호출 경고, 비검사 매개변수화 가변인수 타입 경고, 비검사 변환 경고
- 제네릭을 쓰면 코드를 한번에 깔끔하게 쓰지 쉽지 않다

```java
  Set<Lark> exaltation = new HashSet(); //컴파일 경고
  Set<Lark> exaltation = new HashSet<>(); //올바른 방법
```

#
### 2. 비검사 경고를 제거하자
- 할 수 있는 모든 비검사 경고를 제거하자!
- 모두 제거하면 안정성이 보장됨
- 런타임에 ClassCastException이 발생할 일이 없고 의도한대로 작동하게 됨

<br>

- 경고를 제거할 순 없지만 안전하다고 확신할 수 있다면 __@SuppressWarnings("unchecked")__ 어노테이션을 달아 경고를 숨겨라
- 안전하다 검증하지 않은 채 제거하면 망한다

#
### 3. @SuppressWarnings
- 개별 지역변수 선언부터 클래스 선언까지 어떤 선언에도 달 수 있다
- 하지만 항상 좁은 법위에 적용하자! 그래야 심각한 경고를 놓치지 않을 수 있다
- 한줄이 넘는 메서드나 생성자라면 지역변수로 옮기자!

```java
public <T> T[] toArray(T[] a){
  if(a.length < size){
    @SuppressWarnings("unchecked") T[] result = (T[] Arrays.copyOf(elements, size, a.getClass());
    return result;
  }
  System.arraycopy(elements, 0, a, 0, size);
  if(a.length > size) a[size] = null;
  return a;
}
```

- 이 코드는 깔끔하게 컴파일되고 비검사 경고를 숨기는 범위도 최소로 해준다!
- @SuppressWarnings을 생략하려면 주석으로 생략해도 되는 이유를 명시해주자
