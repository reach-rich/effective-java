# 한정적 와일카드를 사용해 API 유연성을 높이라

### 1. 불공변 방식보다 유연한 무언가가 필요해!
- List\<String>, List\<Object> 와 같은 매개변수화 타입은 불공변이다
- List\<String>이 List\<Object>가 하는 일을 제대로 수행할 수 없으니까 하위타입이 될 수 업는 것
- 이런 불공변 방식보다 유연한 무언가가 필요하다

#
### 2. 

```java
  public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
  }
```

- 이런 스택 API에 메서드를 추가한다고 생각해보자

```java
  public void pushAl(Iterable<E> src){
    for(E e : src){
      push(e);
    }
  }
```

- 이 메서드는 컴파일 되지만 완벽하진 않음
- Itetable src 의 원소 타입이 스택과 일치하지 않는다면 작동하지 않음
- Number 스택일 때 하위인 Integer을 넣어도 안됨
- 매개변수화 타입이 불공변이기 때문!

<br>

- 해결방법 : 한정적 와일드카드 타입 사용하기

```java
  public void pushAll(Interable<? extends E> src) {
    for (E e : src) {
      push(e);
    }
  }
```

- 이 수정으로 클라이언트 코드까지 깔끔히 컴파일 된다
- pushAll과 짝이 될 popAll을 작성해보자

```java
  public void popAll(Collection<E> dst){
    while (!isEmpty()){
      dst.add(pop());
    }
```
