# 한정적 와일카드를 사용해 API 유연성을 높이라

### 1. 불공변 방식보다 유연한 무언가가 필요해!
- List\<String>, List\<Object> 와 같은 매개변수화 타입은 불공변이다
- List\<String>이 List\<Object>가 하는 일을 제대로 수행할 수 없으니까 하위타입이 될 수 업는 것
- 이런 불공변 방식보다 유연한 무언가가 필요하다

#
### 2. 불공변 방식보다 유연한 무언가

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

- 이것도 잘 돌아가고 컴파일도 잘 되지만 완벽하진 않다
- Stack\<Number>의 원소를 Object용 컬렉션으로 옮기면? 오류 발생한다
- 이번에도 와일드타입으로 해결 가능

```java
  public void popAll(Collection<? super E> dst {
    while(!isEmpty()){
      dst.add(pop());
    }
  }
```
- 이제 스택과 클라이언트 코드 모두 깔끔하게 컴파일~

<br>

- __결론__ : 유연성을 극대화하려면 원소의 생산자나 소비자영 입력 매개변수에 와일드카드 타입을 사용하라!
- 하지만 생산자와 소비자 역할을 동시에 한다면 와일드카트 타입을 쓸 필요가 없다 (쓰지마라)

#
### 3. PECS (producer-extends, consumer-super)
- 매개변수화 타입 T가 생산자라면 <? extends T>
- 매개변수화 타입 T가 소비자라면 <? extends T>
- 와일드카드 타입을 사용하는 것이 기본 원칙

<img width="527" alt="image" src="https://user-images.githubusercontent.com/59560592/188314963-f3b2d5ca-d66b-456d-a73a-99dc74be4df2.png">

- 생산자 매개변수에 와일드카드 타입 적용

```java
  public Chooser(Collection<? extends T> choices)
```

```java
  public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
```
- 반환타입에는 한정적 와일드카드를 쓰면 안된다!!!!!
- 클래스 사용자는 와일드 카드가 사용됐는지도 모르게 잘 작동한다
- 클래스 사용자가 와일드카드 타입을 신경써야한다면 그건 잘못된 API

- 이 코드는 자바 8부터 사용 가능하다


<br>

- 자바7에서는 명시적 타입 인수를 추가해주면 가능
 
```java
  Set<Number> numbers = union(integers, doubles);
  Set<Number> numbers = Union.<Number>union(integers, doubles);
```

#
### 4. Comparable\<? super E>

```java
  publuc static <E extends Comparable<? super E>> E max(List<? extends E> list)
```
- PECS 공식을 두번 적용힘
- Comparable\<E>는 E 인스턴스를 소비해 Comparable\<? super E> 사용

#
### 5. 메서드를 정의할 때는 타입 매개변수나 와일드카드 둘 다 써도 된다

```java
  public static <E> void swap<List<E> list, int i, int j);
  public static void swap<List<?> list, int i, int j);
```
- public API라면 두번쨰다 낫다
- 어떤 리스트든 넘기면 명시한 인덱스의 원소들로 교환해준다

- __메서드 선언에 타입 매개변수가 한번만 나오면 와일드 카드로 대체하라__

<br>

- List<?>에는 null 말고는 들어갈수가 없어서 private 도우미 메서드를 써줘야할 때도 있다

```java
  private static <E> void swapHelper(List<E> list, int i, int j){
    list.set(i, list.set(j, list.get(i)));
  }
```
