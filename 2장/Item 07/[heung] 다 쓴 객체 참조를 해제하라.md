Effective Java의 일곱 번째 아이템 "다 쓴 객체 참조를 해제하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 0. 들어가며

자바는 다 쓴 객체를 **GC(Garbage Collector)**가 알아서 회수해 갑니다. 그래서 자칫 메모리 관리에 더 이상 신경 쓰지 않아도 된다고 오해할 수 있는데, 이는 절대 사실이 아닙니다. 가비지 컬렉션 언어에서는 의도치 않게 객체를 살려두는 '**메모리 누수**'를 찾기가 아주 까다롭습니다. 객체 참조 하나를 살려두면 GC는 그 객체뿐 아니라 그 객체가 참조하는 모든 객체(그리고 또 그 객체들이 참조하는 모든 객체)를 회수해가지 못합니다. 때문에 단 몇 개의 객체가 매우 많은 객체를 회수하지 못하게 할 수 있고 잠재적으로 성능에 악영향을 줄 수 있습니다. 

본문에서 다 쓴 참조(앞으로 다시 쓰지 않을 참조)가 해제되지 않아 메모리 누수가 발생하는 예와 해결 방법을 알아보겠습니다.

<br>

## 1. 메모리 누수 예

CG가 알아서 다 쓴 객체를 회수해가는데 왜 메모리 누수가 발생하는 걸까요? 원인은 프로그래머가 작성한 클래스에 있습니다. 어떤 **클래스가 자기 메모리를 직접 관리**한다면, GC는 해당 메모리가 더 이상 쓸 일이 없는 참조라는 것을 인식하지 못하게 될 수 있습니다. 따라서 여전히 다 쓴 참조가 해제되지 않고 메모리 누수가 발생하는 것입니다. 세 가지 예제를 통해 자세히 알아보겠습니다.

### 1.1 스택

스택은 아시다시피 후입선출 방식의 자료구조입니다. java.util 패키지에서 구현체를 제공해 주지만, 메모리 누수가 발생하는 상황을 만들기 위해 직접 구현했습니다.

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
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }
  
    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

위 예제에서는 스택에 원소가 쌓였다가(push) 꺼내질 때(pop) 메모리 누수가 일어납니다. 더 이상 사용되지 않을 꺼내진 객체들을 GC는 회수하지 않습니다. 그 이유는 스택이 해당 객체들의 다 쓴 참조를 여전히 가지고 있기 때문입니다. 위의 예제에서는 elements 배열의 '활성 영역' 밖의 참조들이 모두 여기에 해당합니다. 여기서 활성 영역은 size보다 작은 인덱스를 말합니다.

이를 해결하기 위한 방법은 간단합니다. 해당 **참조를 다 썼을 때 null 처리(참조 해제)**해주면 됩니다. 다음은 pop 메서드를 개선한 코드입니다.

```java
public Object pop() {
  if (size == 0)
    throw new EmptyStackException();
  Object result = elements[--size];
  elements[size] = null; // 다 쓴 참조 해제
  return result;
}
```

예제의 Stack 클래스는 왜 null 처리가 필요한 걸까요? 이유는 elements 배열로 저장소 풀을 만들어 원소들을 관리(자기 메모리를 직접 관리)하기 때문입니다. 배열의 활성 영역에 속한 원소들이 사용되고 비활성 영역은 쓰이지 않습니다. 문제는 GC가 이 사실을 알 길이 없다는 것입니다. **GC의 관점에서는 비활성 영역에서 참조하는 객체도 똑같이 유효한 객체입니다.** 비활성 영역의 객체가 더 이상 쓸모없다는 것은 프로그래머만 아는 사실입니다. 때문에 프로그래머는 비활성 영역이 되는 순간 null 처리해서 해당 객체는 더 이상 쓰지 않을 것임을 GC에 알려야 합니다.

<br>

### 1.2 캐시

캐시 역시 메모리 누수를 일으키는 주범입니다. 객체 참조를 캐시에 넣고 나서, 이 사실을 까맣게 잊은 채 그 객체를 다 쓴 뒤로도 한참을 그냥 놔두는 일을 자주 접할 수 있습니다. 다음은 캐시를 사용한 예제입니다.

```java
public class CacheKey {

    private Integer value;

    private LocalDateTime created;

    public CacheKey(Integer value) {
        this.value = value;
        this.created = LocalDateTime.now();
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        CacheKey cacheKey = (CacheKey) o;
        return value.equals(cacheKey.value);
    }

    @Override
    public int hashCode() {
        return Objects.hash(value);
    }
}
```

```java
public class Post {

    private Long id;

    private String title;

    private String content;
}
```

CacheKey 클래스는 캐시의 key로 사용되고 Post 클래스는 캐싱된 value로 사용됩니다. 

```java
public class PostRepository {

    private Map<CacheKey, Post> cache;

    public PostRepository() {
        this.cache = new HashMap<>();
    }

    public Post getPostById(CacheKey key) {
        if (cache.containsKey(key)) {
            return cache.get(key);
        } else {
            // TODO DB에서 읽어오거나 REST API를 통해 Post 검색
            Post post = new Post(); // 검색된 Post
            cache.put(key, post);
            return post;
        }
    }

    public Map<CacheKey, Post> getCache() {
        return cache;
    }
}
```

Repository는 실제 캐싱이 되는 부분입니다. getPostById 메서드는 캐시에서 해당 key가 이미 존재하는지 확인합니다. 존재하지 않으면 DB나 다른 API를 통해 Post를 얻고 캐시에 저장합니다. 여기서 캐시는 HashMap을 사용했습니다.

다음은 Post를 조회하는 클라이언트 코드입니다. 

```java
PostRepository postRepository = new PostRepository();
CacheKey cacheKey = new CacheKey(1);
CacheKey newCacheKey = new CacheKey(1);

Post post1 = postRepository.getPostById(cacheKey);
Post post2 = postRepository.getPostById(newCacheKey); // post1과 post2는 같은 인스턴스 (캐시)

System.out.println(postRepository.getCache().size());

System.out.println("run GC");
System.gc(); // GC 동작
System.out.println("wait");
Thread.sleep(1000);

System.out.println(postRepository.getCache().size());

/* 실행 결과
cache size : 1
run GC
wait
cache size : 1
*/
```

처음 getPostById 메서드를 실행했을 때, CacheKey(1)을 key로 하여 새로운 Post 참조가 캐싱됩니다. 그리고 두 번째 getPostById에서 역시 동일한 값을 가지는 key로 조회하였기 때문에 캐싱된 Post 참조가 반환됩니다. 캐시에는 하나의 참조가 캐싱된 것이기 때문에 size는 1입니다.

다음, 명시적으로 GC를 실행시켰습니다. 그리고 다시 캐시 size를 확인해 보면 당연히 1인 것을 알 수 있습니다. 아무런 문제가 없어 보입니다. 그런데 **만약 CacheKey(1)를 더 이상 쓸 일이 없고 프로그래머가 이를 잊었다면 어떨까요?** 캐싱되어 있는 참조가 계속 남아서 메모리를 차지할 것입니다. 

이를 해결하는 방법은 여러 가지인데, 위의 예제처럼 운 좋게 캐시 외부에서 key를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요한 상황이라면 **WeakHashMap**을 사용해 캐시를 만드는 것을 권장합니다. 다 쓴 엔트리는 그 즉시 자동으로 제거될 것입니다. 단, WeakHashMap은 이러한 상황에서만 유용하다는 사실을 기억해야 합니다.

```java
public class PostRepository {

    private Map<CacheKey, Post> cache;

    public PostRepository() {
        this.cache = new WeakHashMap<>(); // HashMap -> WeakHashMap
    }

    public Post getPostById(CacheKey key) {
        if (cache.containsKey(key)) {
            return cache.get(key);
        } else {
            // TODO DB에서 읽어오거나 REST API를 통해 Post 검색
            Post post = new Post();
            cache.put(key, post);
            return post;
        }
    }

    public Map<CacheKey, Post> getCache() {
        return cache;
    }
}
```

```java
PostRepository postRepository = new PostRepository();
CacheKey cacheKey = new CacheKey(1);
CacheKey newCacheKey = new CacheKey(1);

Post post1 = postRepository.getPostById(cacheKey);
Post post2 = postRepository.getPostById(newCacheKey);

System.out.println("cache size : " + postRepository.getCache().size());

cacheKey = null; // 참조 해제

System.out.println("run GC");
System.gc();
System.out.println("wait");
Thread.sleep(1000);

System.out.println("cache size : " + postRepository.getCache().size());

/* 실행 결과
cache size : 1
run GC
wait
cache size : 0
*/
```

클라이언트 코드를 보면 `cacheKey = null;`가 추가되었습니다. 캐싱되어 있는 참조가 더 이상 쓸 일이 없기 때문에, null로 처리해 주어 참조를 해제하는 부분입니다. WeakHashMap는 다 쓴 엔트리를 제거한다고 했습니다. 따라서 GC가 동작할 때 캐싱되어 있는 참조 중 사용되지 않는 참조가 제거됩니다. 때문에 실행 결과는 GC가 동작한 후의 size가 0이 나오게 된 것입니다.

<br>

캐시를 만들 때 보통은 캐시 엔트리의 유효 기간을 정확히 정의하기 어렵기 때문에 시간이 지날수록 엔트리의 가치를 떨어뜨리는 방식을 흔히 사용합니다. 이런 방식에서는 쓰지 않는 엔트리를 이따금 청소해 줘야 합니다. (Scheduled ThreadPoolExecutor 같은) 백그라운드 스레드를 활용하거나 캐시에 새 엔트리를 추가할 때 부수 작업으로 수행하는 방법이 있습니다. LinkedHashMap은 removeEldestEntry 메서드를 써서 후자의 방식으로 처리합니다. 더 복잡한 캐시를 만들고 싶다면 java.lang.ref 패키지를 직접 활용해야 합니다.

<br>

### 1.3 리스너와 콜백

리스너와 콜백도 캐시처럼 메모리 어딘가에 저장되어야 합니다. 클라이언트가 리스너 또는 콜백을 등록만 하고 명확히 해지하지 않는다면, 뭔가 조치해 주지 않는 한 계속 쌓여갈 것입니다. 이럴 때 콜백을 약한 참조(weak reference)로 저장하면 GC가 즉시 수거해갑니다. 예를 들어 위에서 언급했던 WeakHashMap에 key로 저장하면 됩니다. 로직이 동일하기 때문에 예제 코드는 생략하겠습니다.

<br>

## 2. 핵심 정리

책에서 소개한 메모리 누수의 예는 스택, 캐시, 리스너와 콜백 이렇게 세 가지이다. 그리고 해결 방법으로는 직접 null 처리, 특정한 자료구조 사용, 백그라운드 스레드 사용 이렇게 세 가지가 있다.

메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다. 이런 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 한다. 그래서 이런 종류의 문제는 예방법을 익혀두는 것이 매우 중요하다.

<br>

## 3. Related Posts

- 변수의 범위 (Item 57)

