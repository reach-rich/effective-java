# 이왕이면 제네릭 타입으로 만들라

- JDK가 제공하는 제네릭 타입과 메서드를 사용하는 일은 일반적으로 쉬운 편, 제네릭 타입을 새로 만드는 일은 좀 어려움
- 하지만 배워두면 그 값어치는 할 것임!

### 1. 제네릭이 필요한 예시

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0
            throw new EmptyStackException();
        Object result = elemtns[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }
    
    public boolean isEmpty() {
        return size == 0;
    }
    
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

- 이 클래스는 원래 제네릭 타입이어야 마땅
- 제네릭으로 바꾼다고 해서 현재 버전을 사용하는데 아무런 문제 없음
- 오히려 이 그대로 쓰면 스택에서 객체 꺼내면서 형변환 안해서 런타임 오류 날 수 있음

#
### 2. 일반클래스 -> 제네릭 클래스
- 클래스 선언 타입에 매개변수 추가하기

```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elemtns[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }
}
 ```
- 여기서 오류발생!
- E 같은 실체화 불가 타입으로는 배열 만들기 불가능~

<br>

- __첫번째 해결 방법__) Object 배열을 생성한 다음 제네릭 배열로 형변환하기

```java
@SuppressWarnings("unchecked") //우리가 증명해서 붙여줌
elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
```
- 이 비검사 형변환은 안전하다 
  - elements는 private 필드에 저장
  - 클라이언트로 반환되거나 다른 메서드에 전달되는 일이 없음
  - 원소의 타입은 항상 E

<br>

- __두번째 해결 방법__) elements의 필드 타입을 Object[]로 바꾸기

 ```java
  E result = elements[--size];
 ```
- 배열이 반환한 원소를 E 로 형변환하면 오류 대신 경고가 뜸
- E는 실체화 불가 타입이므로 컴파일러는 모름
- 우리가 증명해야해

<br>
 
```java
public E pop() {
    if (size == 0)
        throw new EmptyStackException();
    
    @SuppressWarnings("unchecked")
    E result = (E) elements[--size];
    
    elements[size] = null; // 다 쓴 참조 해제
    return result;
}
```
- push 에서 E 타입만 허용하니까 이 형변환은 안전하다 해줌


#
### 3.제네릭 배열 생성을 제거하는 두 방법 비교
- 첫번째 방법이 가독성은 좋고 코드도 짧다
- 첫번째 방법은 형변환을 배열 생성 시 단 한번만 해주면 됨
- 두번째 방식은 형변환을 원소 읽을 때마다 해줌
- 현업에서는 첫번째 방식을 더 선호

- 하지만 힙 오염을 일으키기도 해서 (컴파일시와 런타임시에 배열의 타입이 달라짐)맘에 걸리면 두번째거 씀


#
### 4. 마무리
- 좀 더 성장해서 다시 보자






