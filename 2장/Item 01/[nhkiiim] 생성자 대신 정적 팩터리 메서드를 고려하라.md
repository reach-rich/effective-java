## 1. public 생성자로 객체 생성하기

```java
public class Character {
    protected int intelligence, strength, hitPoint, magicPoint;

    // public 생성자
    public Character(int intelligence, int strength, int hitPoint, int magicPoint) {
        this.intelligence = intelligence;   // 지능
        this.strength = strength;           // 힘
        this.hitPoint = hitPoint;           // HP
        this.magicPoint = magicPoint;       // MP
    }
}
```

```java
public class Mage extends Character {
    // public 생성자
    public Mage() {
        super(15, 5, 10, 15);
    }
}
```
```java
public class Warrior extends Character {
    // public 생성자
    public Warrior() {
        super(5, 15, 20, 3);
    }
}
```

<br>

## 2. static factory method로 객체 생성하기

```java
public class Character {

    private int intelligence, strength, hitPoint, magicPoint;

    private Character(int intelligence, int strength, int hitPoint, int magicPoint) {
        this.intelligence = intelligence;   // 지능
        this.strength = strength;           // 힘
        this.hitPoint = hitPoint;           // HP
        this.magicPoint = magicPoint;       // MP
    }

    // '전사 캐릭터'를 생성하는 정적 팩토리 메소드
    public static Character newWarrior() { // 
        return new Character(5, 15, 20, 3);     // 인스턴스를 생성 후 반환
    }

    // '마법사 캐릭터'를 생성하는 정적 팩토리 메소드
    public static Character newMage() {
        return new Character(15, 5, 10, 15);    // 인스턴스를 생성 후 반환
    }
}
```

<br>

## 3. 정적 팩토리 메서드가 생성자보다 좋은 이유
- 항상 정적 팩토리 메서드가 좋은 건 아님!

1) 이름을 가질 수 있다
```java
Character warrior = Character.newWarrior();
Character mage = Character.newMage();
```
2) 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다
3) 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다
4) 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다
5) 클라이언트를 구현체로부터 분리해준다
### 참고
https://jaeseongdev.github.io/development/2021/01/05/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C_%EC%9E%90%EB%B0%94_%EC%95%84%EC%9D%B4%ED%85%9C_1/
