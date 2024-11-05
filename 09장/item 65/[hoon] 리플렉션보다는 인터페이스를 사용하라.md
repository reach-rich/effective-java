## 1. 들어가기

리플렉션을 사용하면 프로그램 임의의 클래스에 접근할 수 있습니다.

리플렉션을 통해 클래스의 생성자, 메서드, 필드 인스턴스를 가져올 수 있고

이 인스턴스들로는 클래스의 멤버 이름, 필드 타입, 메서드 시그니처를 알 수 있습니다.

하지만 리플렉션을 남용하면 문제가 생길 수 있습니다.

그럼 어떤 문제가 발생하는지 살펴보겠습니다.

## 2. 리플렉션의 위험

1. 컴파일 타임 타입 검사가 주는 이점을 누릴 수 없다.

   만약 개발자가 리플렉션 기능을 써서 존재하지 않는 혹은 접근할 수 없는 메서드를 호출하려 시도하면

   런타임 오류가 발생할 수 있습니다.

2. 리플렉션을 이용하면 코드가 지저분하고 장황해진다.

   리플렉션은 코드를 읽기 힘들게 만들어 가독성을 떨어뜨릴 수 있습니다.

3. 성능이 떨어진다.

   리플렉션을 통한 메서드 호출은 일반 메서드 호출보다 느립니다.

## 3. 대안

그럼 리플렉션을 어떻게 다른 방법으로 처리할 수 있을까요?

너무나도 단점이 명백한 리플렉션이기 때문에 제한된 형태로만 사용하는 방법을 선택할 수 있습니다.

그래서 리플렉션을 인스턴스 생성에만 사용하고 인스턴스를 인터페이스나 상위 클래스로 참조해 사용합니다.

```java
   public static void main(String[] args) {

      // 클래스 이름을 Class 객체로 변환
      Class<? extneds Set<String>> cl = null;

      try {
         cl = (Class<? extends Set<String>>) Class.forName(args[0]);
      } catch (ClassNotFoundException e) {
         fatalError("클래스를 찾을 수 없습니다");
      }

      // 생성자를 얻는다
      Constructor<? extends Set<String>> cons = null;

      try {
         cons = cl.getDeclaredConstructor();
      } catch (NoSuchMethodException e){
         fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
      }

      // 집합 인스턴스를 생성
      Set<String> s = null;

      try {
         s = cons.newInstance();
      } catch (IllegalAccessException e) {
         fatalError("생성자에 접근할 수 없습니다.");
      } catch (InstantitaionException e) {
         fatalError("클래스를 인스턴스화할 수 없습니다.");
      } catch (InvocationTargetException e) {
         fatalError("생성자가 예외를 던졌습니다.");
      } catch (ClassCastException e) {
         fatalError("Set을 구현하지 않은 클래스입니다.");
      }

      // 생성한 집합 사용
      s.addAll(Arrays.asList(args).subList(1, args.length));
      System.out.println(s);
   }

   private static void fatalError(String msg) {
      System.err.println(msg);
      Ststem.exit(1);
   }
```

하지만 이렇게 대안으로 만든 예도 두 가지의 단점이 있습니다.

1. 6가지의 런타임 예외를 던진다.

2. 클래스 이름만으로 인스턴스를 생성해내기 위해 많은 코드를 작성한다.

즉, 최대한 리플렉션을 사용하지 않는 것이 좋습니다.

## 4. 리플렉션을 사용해야 하는 특수 케이스

드물긴 하지만 리플렉션을 사용하는 것이 적합한 경우도 있는데

런타임에 존재하지 않을 수 있는 클래스, 메서드, 필드와의 의존성을 관리할 때는 리플렉션이 적합합니다.

버전이 여러 개 존재하는 외부 패키지를 다룰 때 주로 오래된 버전만을 지원하도록 컴파일한 후,

이후 버전의 클래스와 메서드 등은 리플렉션으로 접근하는 방식입니다.

## 5. 정리

이번 포스트는 리플렉션을 왜 지양해야 하고 대안 방법을 알아보았습니다.

리플렉션은 단점이 많고 가독성을 떨어뜨리므로 사용을 지양해야 하지만

꼭 사용해야 한다면 되도록 객체 생성에만 사용하고

적절한 인터페이스나 컴파일 타임에 알 수 있는 상위 클래스로 형변환해서 사용해야 합니다.