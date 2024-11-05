## 1. 들어가기

Garbage Collection(GC)의 위력을 아시나요?

> * Garbage Collection
>
> 시스템에서 더 이상 사용하지 않는 동적 할당된 메모리를 찾아 자동으로 회수하는 메모리 관리 방법

Java나 Python과 같이 비교적 최근에 개발된 언어는 기본 기능 중 하나여서

최근에 개발된 언어만 사용해본 분들은 GC의 위력을 체감하지 못할 수 있습니다.

C, C++과 같은 언어는 메모리를 직접 관리해야 하는 언어인데

매번 malloc, new를 통해 객체를 생성하고 free, delete로 객체를 해제해야 했기 때문에

개발자들을 아주 피곤하게 만들었습니다.

그러다 GC를 갖춘 언어로 넘어가게 되자, 개발자의 삶은 180도 달라지게 되는데

다 쓴 객체를 알아서 회수해가니 더 이상 메모리 관리에 심혈을 기울일 필요가 없게 된 것입니다.

그렇다면 이렇게 잘못된 결론을 내릴 수 있습니다.

> GC를 갖춘 언어를 사용하면 더 이상 메모리 관리에 신경을 쓰지 않아도 되는구나!

과연 그럴까요?

GC를 피해 메모리 누수를 일으키는 주범 3가지를 살펴보겠습니다.

## 2. 메모리 누수를 일으키는 주범

### 2-1. 메모리 직접 관리

```java
  public class Stack {
    ...
    public void push(Object e) {
      elements[size++] = e;
    }

    public Object pop() {
      if(size == 0)
        throw new EmptyStackException();

      return elements[--size];
    }
    ...
  }
```

해당 코드는 Stack의 핵심 연산인 push와 pop입니다.

여기서 메모리 누수가 일어나는 부분은 어디일까요?

바로, pop 연산에서 일어납니다.

Stack의 가장 윗부분에 위치한 영역을 size 변수를 통한 비활성 영역으로 변경할 뿐,

실제로 가장 윗부분에 위치한 객체가 GC의 대상이 되지 않기 때문인 것이죠.

그렇다면 이 문제는 어떻게 해결할 수 있을까요?

가장 쉬운 방법은 비활성 영역으로 변경할 때 가장 윗부분에 위치한 객체를 null 처리해주면 됩니다.

```java
  public Object pop() {
    if(size == 0)
      throw new EmptyStackException();

    Object result = elements[--size];
    elements[size] = null;
    return result;
  }
```

다 쓴 참조를 null 처리하면 메모리 누수를 방지할 뿐만 아니라 다른 이점도 있습니다.

만약, 기존의 코드에서 비활성 영역의 데이터를 실수로 접근을 하게 되면

비활성 영역이지만 실제로 데이터가 있기 때문에 잘못된 일을 수행할 수 있습니다.

하지만, null 처리를 한 후 비활성 영역에 접근을 하게 되면 NullPointerException을 던지며 종료하므로

조기에 프로그램 오류를 발견할 수 있습니다.

그렇다고 모든 객체를 쓰자마자 일일이 null 처리하는 것은 프로그램을 필요 이상으로 지저분하게 만듭니다.

그래서 가장 좋은 방법은 아래와 같이 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것입니다.

```java
  public class Stack<E> extends Vector<E> {
    ...
    public E push(E item) {
        addElement(item);

        return item;
    }

    public synchronized E pop() {
        E       obj;              // 지역 변수 선언
        int     len = size();

        obj = peek();             
        removeElementAt(len - 1);

        return obj;               // 반환과 동시에 참조 해제 
    }
    ...
  }
```

### 2-2. Cache

캐시 역시 메모리 누수를 일으키는 주범입니다.

객체 참조를 캐시에 넣고 난 후, 그 객체를 다 쓴 뒤로도 한참 놔두는 일을 자주 접할 수 있습니다.

해결 방법은 여러 가지가 있을 수 있는데 그 중 하나는 WeakHashMap을 사용해 캐시를 만드는 것입니다.

Hashmap과 WeakHashMap으로 만든 예시를 통해 비교해보겠습니다.

Hashmap의 경우, key에 해당하는 객체가 더이상 사용되지 않아도 해당 내용은 삭제되지 않습니다.

```java
/* HashMap 사용 */
  static class Person {
      public String toString(){
          return "name";
      }
  }

  public static void main(String[] args) throws InterruptedException {
      Map<Person, String> cache = new HashMap<>();
      Person a = new Person();
      Person b = new Person();

      // Puts an entry into HashMap
      cache.put(a, "AA");
      cache.put(b, "BB");

      System.out.println(cache);

      // AA Person is not used
      a = null;

      // Garbage collector is called
      System.gc();

      // Thread sleeps for 4 sec
      Thread.sleep(4000);

      System.out.println(cache);
  }
```

```
  {name=BB, name=AA}
  {name=BB, name=AA}
```

반면 WeakHashMap의 경우, key에 해당하는 객체가 더이상 사용되지 않는다고 판단되면 제거됩니다.

```java
/* WeakHashMap 사용 */
  static class Person {
      public String toString(){
          return "name";
      }
  }

  public static void main(String[] args) throws InterruptedException {
      Map<Person, String> cache = new WeakHashMap<>();
      Person a = new Person();
      Person b = new Person();

      // Puts an entry into WeakHashMap
      cache.put(a, "AA");
      cache.put(b, "BB");

      System.out.println(cache);

      // AA Person is not used
      a = null;

      // Garbage collector is called
      System.gc();

      // Thread sleeps for 4 sec
      Thread.sleep(4000);

      System.out.println(cache);
  }
```

```
  {name=BB, name=AA}
  {name=BB}
```

이렇게 WeakHashMap을 통해 메모리 누수를 방지하는 방법이 있습니다.

하지만, 캐시를 만들 때 보통 캐시 엔트리의 유효 기간을 정확히 정의하기 어렵습니다.

그래서 보통 시간이 지날수록 엔트리의 가치를 떨어뜨리는 방식을 사용하는데요.

이럴 때는 ScheduledThreadPoolExecutor와 같은 백그라운드 스레드나,

LinkedHashMap처럼 캐시에 새 엔트리를 추가할 때 부수 작업으로 수행하는 방법이 있습니다.

```java
  void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
  }
```

### 2-3. Listener, Callback

메모리 누수의 세 번째 주범은 리스너와 콜백입니다.

리스너와 콜백은 비슷하지만 차이점이 있습니다.

> 콜백(Callback) : 이벤트가 발생하면 특정 메소드를 호출해 알려준다.
>
> 리스너(Listener) : 이벤트가 발생하면 연결된 리스너(핸들러)들에게 이벤트를 전달한다.

다시 돌아가서, 클라이언트가 리스너나 콜백을 등록만 하고 명확히 해지하지 않는다면

리스너나 콜백들은 계속 쌓여만 갈 것입니다.

이 때, 해결방안으로 리스너나 콜백을 약한 참조(WeakReference)로 저장하면 

GC가 사용하지 않는 콜백과 리스너들을 수거해 메모리 누수를 방지할 수 있습니다.

## 3. 정리

이번 포스트에서는 메모리 누수를 일으키는 사례 3가지를 살펴보았습니다.

메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있습니다.

어떤 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 합니다.

그래서 사례와 함께 알아보았던 예방법을 통해 메모리 누수가 일어나지 않도록 사전에 차단해봅시다.