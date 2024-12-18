# 과도한 동기화는 피하라

과도한 동기화는 성능을 떨어뜨리고, 교착상태에 빠뜨리고, 심지어 예측할 수 없는 동작을 낳기도 한다. 

<br>

## 1) 동기화 블록 안에서 외계인 메서드 호출
응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안 된다.
* 동기화된 영역을 포함한 클래스 관점에서 클라이언트가 넘겨준 메서드는 무슨 일을 할지 알지 못하며 통제할 수도 없다.
* 때문에 클라이언트에 따라 동기화된 영역은 예외를 일으키거나, 교착상태에 빠지거나, 데이터를 훼손할 수도 있다.
```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized(observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element); // notifyElementAdded 호출
        return result;
    }
}
```
위는 어떤 집합(Set)을 감싼 래퍼 클래스다. 이 클래스의 클라이언트는 집합에 원소가 추가되면 알람을 받을 수 있다. (관찰자 패턴)
SetObserver는 구조적으로 BiConsumer<ObservableSet<E>, E>와 똑같다.
```jav
@FunctionalInterface
public interface SetObserver<E> {
    // ObservableSet에 원소가 추가되면 호출된다.
    void added(ObservableSet<E> set, E element);
}

```
아래 프로그램은 0부터 99까지 잡합에 추가된 정숫값을 출력하다가, 그 값이 23이면 자기 자신을 제거한다.

```java
public static void main(String[] args) {
    ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
    set.addObserver(new SetObserver<>() {
        public void added(ObservableSet<Integer> s, Integer e) {
            System.out.println(e);
            if (e == 23)
                s.removeObserver(this);
        }
    });

    for (int i = 0; i < 100; i++)
        set.add(i;)
}
```
이 프로그램은 23까지 출력한다은 ConcurrentModificationException을 던진다. 
관찰자의 added 메서드 호출이 일어난 시점이 notifyElementAdded가 관찰자들의 리스트를 순회하는 도중이기 때문이다. 
리스트에서 원소를 제거하려 하는데, 마침 지금은 이 리스트를 순회하는 도중이다. 즉, 허용하지 않는 동작이다.

<br>

## 2) 쓸데없이 백그라운드 스레드를 사용하는 관찰자

아래 프로그램은 구독해지를 하는 관찰자를 작성하는데, removeObserver를 직접 호출하지 않고 실행자 서비스를 사용해 다른 스레드한테 부탁한다.
```java
public static void main(String[] args) {
    ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

    set.addObserver(new SetObserver<>() {
        public void added(ObservableSet<Integer> s, Integer e) {
            System.out.println(e);
            if (e == 23) {
                ExecutorService exec = Executors.newSingleThreadExecutor();
                try {
                    // 여기서 lock이 발생한다. (메인 스레드는 작업을 기리고 있음)
                    // 특정 태스크가 완료되기를 기다린다. (submit의 get 메서드)
                    exec.submit(() -> s.removeObserver(this)).get();
                } catch (ExecutionException | InterruptedException ex) {
                    throw new AssertionError(ex);
                } finally {
                    exec.shutdown();
                }
            }
        }
    })

    for (int i = 0; i < 100; i++) {
        set.add(i);
    }
}
```
실행하면 예외는 나지 않지만 교착상태에 빠진다. 
메인스레드가 이미 관찰자에 대해 락을 쥐고 있기 때문에 백그라운드 스레드가 s.removeObserver를 호출하여 락을 얻을 수 없다. 
그와 동시에 메인스레드는 백그라운드 스레드가 관찰자를 제거하기만을 기다리는 중이다.

<br>

## 3) 해결 방법
```java
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized (observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot) {
        observer.added(this, element);
    }
}
```
```java
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers)
        observer.added(this, element);
}
```



