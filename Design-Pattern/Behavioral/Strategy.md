### 전략 패턴

---

    유사한 행위를 캡슐화 하여 객체가 행위를 직접 수정하지 않고 전략만 변경하는 방법 
    1.행위 객체에 대한 클래스를 생성
    2.유사한 행위를 캡슐화 한 인터페이스 생성
    3.캡슐화 한 인터페이스를 구현하는 구현 객체 생성
    4.캡슐화한 구현 객체를 행위 객체에 전달하여 행위를 유연하게 확장

```java
public interface MovableStrategy {
    void move();
}

public class RailLoadStrategy implements MovableStrategy {
    @Override
    public void move() {
        System.out.println("선로를 통해 이동");
    }
}

public class LoadStrategy implements MovableStrategy {
    @Override
    public void move() {
        System.out.println("도로를 통해 이동");
    }
}

public class Moving {
    private MovableStrategy movableStrategy;

    public void setMovableStrategy(MovableStrategy movableStrategy) {
        this.movableStrategy = movableStrategy;
    }

    public void move() {
        movableStrategy.move();
    }
}

public class Bus extends Moving {}

public class Driver {
    public static void main(String[] args) {
        Moving bus = new Bus();
        bus.setMovableStrategy(new RailLoadStrategy()); //전략 변경
        bus.move();
    }
}

/**
 출력 결과 :
 선로를 통해 이동 
 */
```