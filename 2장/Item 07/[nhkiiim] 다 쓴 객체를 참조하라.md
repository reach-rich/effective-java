# 다 쓴 객체를 참조하라

### 1. 자바의 가비지 콜랙터
- C에서 넘어오면 너무 좋은 기술
- 하지만 메모리 관리를 안해도 된다 생각했다면 경기도 오산

- 메모리 누수 위치는 어디일까?

```java
public class Stack{
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
  public Stack(){
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }
  
  public void push(Object e){
    ensureCapacity();
    elements[size++] = e;
  }
  
  public Object pop(){
    if(size==0)
      throw new EmptyStackException();
     return elements[--size];
  }
  
  private void ensureCapacity(){
    if(elements.length == size)
      elements = Arrays.copyOf(elements, 2 * size + 1)
  }
}
```

- 문제는 메모리 사용량 증가와 가비지 컬랙션 활동으로 인한 성능 저하!
- OutOfMemortError이 일어나 프로그램이 종료되기도 함

- 그래서 메모리 누수는 어디?? `스택이 커졌다 줄어들 때 스택에서 빼온 객체들!`
- 그 객체들은 가비지 컬랙터가 회수하지 않음 (쓰지않아도 노 회수)
- obsolete reference를 가지기 때문
  - https://stackoverflow.com/questions/6843102/what-is-meant-by-obsolete-reference-in-java
  - 값이 메서드에서 튀어나온 경우에도 해당 값이 인덱스에 의해 배열 내부에서 계속 가리키고 있다는 것
  - 따라서 프로그램이 실제로 전달된 참조 복사본으로 완료되더라도 스택에서 사용되는 요소 배열 때문에 여전히 힙에 있음

#

### 2. 참조를 다 썼으면 null 처리 해주기
- 가비지 컬렉션 언어에서 메모리 누수 찾기는 어렵다

- 객체 참조 하나를 살려두면 가비지 컬랙터는 참조하는 모든 객체를 회수하지 못한다! -> 성능 악영향으로 갈 수 있음
- 객체 참조를 다 사용했을 경우 null(참조해제) 해주면 메모리 누수 막을 수 있음

```java
public Object pop(){
  if(size==0)
    throw new EmptyStackException();
    
   Object result = elements[--size];
   elements[size] = null;
   return result;
}
```
- null 처리의 이점 많음!
- 실수로 Null 접근시 예외를 던지며 종료 가능 (조기에 오류 발견 귣)
 
- 하지만 `아무데나 null 처리 남발하는 것도 안좋음!!`
- 객체 참조를 null 처리하는 일은 예외적인 경우여야함
- 참조를 해재하는 가장 좋은 방법은 그 참조를 담은 변수를 유효범위 밖으로 밀어내기!
- 변수의 범위를 최소가 되게 정의했다면 아주 좋지


#
### 3. 무조건적인 null 처리는 No! 그렇다면 언제??
- 스택은 자기 메모리를 직접 관리 -> 배열 활용
- 배열은 활성영역과 비활성 영역이 있는데 -> 가비지 컬렉터 : " 내가 어케 알어유 ^ㅠ^ "

- 프로그래머만 비활성 영역임을 안다 -> 그래서 null 처리 해줘야한다
 ` 자기 메모리를 직접 관리하는 클래스라면 메모리 누수에 주의하세요`
 
#
### 4. 캐시도 메모리 누수의 주범!
- 객체 참조를 캐시에 넣고 나서 까먹으먄? 한참을 그냥 놔두게 됨
- WeakHashMap : 캐시 외부에서 키를 참조하는 동안만 엔트리가 살아있는 캐시가 필요한 상황에 사용
  - https://blog.breakingthat.com/2018/08/26/java-collection-map-weakhashmap/
  
  - 키가 참조하지 않는 객체는 제거하는 HashMap 
   
```java
public class WeakHashMapTest {

    public static void main(String[] args) {
        WeakHashMap<Integer, String> map = new WeakHashMap<>();

        Integer key1 = 1000;
        Integer key2 = 2000;

        map.put(key1, "test a");
        map.put(key2, "test b");

        key1 = null;

        System.gc();  //강제 Garbage Collection

        map.entrySet().stream().forEach(el -> System.out.println(el));

    }
}
```

- 위의 상황이 아니라면 엔트리 청소 가끔씩 해줘야 한다!
  - Scheduled TreadPoolExecutor 같은 백그라운드 스레드 활용
  -  https://codechacha.com/ko/java-scheduled-thread-pool-executor/ `실습 해보기`

- LinkedHashMap은 removeEldestEntry 메서드를 써서 부수 작업을 수행

- java.lang.ref 패키지 직접 활용해서 만들면 더 복잡한 캐시 만들기 가능



#
### 5. 메모리 누수의 또 다른 범인! -> 리스너 / 콜백
- 클라이언트가 콜백만하고 해지하지 않으면 콜백이 계속 쌓임

- 콜백함수 
  1. 다른 함수의 인자로써 이용되는 함수.
  2. 어떤 이벤트에 의해 호출되어지는 함수.

- 콜백함수는 약한 참조를 해두자! -> WeakHashMap처럼~!!

