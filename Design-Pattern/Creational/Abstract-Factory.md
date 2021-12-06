### 추상 팩토리 패턴

---

상호작용 하는 객체들을 묶은 팩토리 클래스를 생성하고 조건에 따라 팩토리 클래스를 사용하여 객체 생성

     최상위 계층의 구분이 명확할 때 하위 객체 생성을 팩토리 메소드 패턴처럼 세분화 하지않고 하위 객체들을 캡슐화 하여 일관되게 생성 

```java
public class FactoryOfCarFactory {
    public void createCar(String type) {
        CarFactory carFactory = null;
        switch (type) {
            case "Benz" :
                carFactory = new BenzCarFactory();
                break;
            case "Audi" :
                carFactory = new AudiCarFactory();
                break;
        }
        carFactory.createTire();
        carFactory.createWheel();
    }
}

public interface CarFactory {
    public Tire createTire();
    public Wheel createWheel();
}

public class BenzCarFactory implements CarFactory {
    @Override
    public BenzTire createTire() {
        return new BenzTire();
    }

    @Override
    public BenzWheel createWheel() {
        return new BenzWheel();
    }
}

public class AudiCarFactory implements CarFactory {
    @Override
    public AudiTire createTire() {
        return new AudiTire();
    }

    @Override
    public AudiWheel createWheel() {
        return new AudiWheel();
    }
}

public interface Tire {}

public class BenzTire implements Tire {
    public BenzTire() {
        System.out.println("벤츠 타이어 생성");
    }
}

public class AudiTire implements Tire {
    public AudiTire() {
        System.out.println("아우디 타이어 생성");
    }
}

public interface Wheel {}

public class BenzWheel implements Wheel {
    public BenzWheel() {
        System.out.println("벤츠 휠 생성");
    }
}

public class AudiWheel implements Wheel {
    public AudiWheel() {
        System.out.println("아우디 휠 생성");
    }
}

public class Consumer {
    public static void main(String[] args) {
        FactoryOfCarFactory factory = new FactoryOfCarFactory();
        factory.createCar("Benz");
    }
}

/**
 출력 결과 :
 벤츠 타이어 생성
 벤츠 휠 생성
 */
```