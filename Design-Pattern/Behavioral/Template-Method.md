### 템플릿 메소드 패턴

---

    많은 클래스에서 공통적으로 사용할 메소드가 명확할 때 추상 클래스로 템플릿 메소드를 제공하며
    추상 클래스를 상속받은 클래스는 훅 메소드를 이용하여 유연하게 확장할 수 있도록 한다 

```java
public abstract class Car {
    public void autoStart() {
        start();
        accelerator();
        dynamicSelect();
    }

    public abstract void start();

    public void accelerator() {
        System.out.println("악셀레이터 밟기");
    }

    public abstract void dynamicSelect();
}

public class Benz extends Car {
    @Override
    public void start() {
        System.out.println("Benz Start");
    }

    @Override
    public void dynamicSelect() {
        System.out.println("Change Mode : Sport");
    }
}

public class Hyundai extends Car {
    @Override
    public void start() {
        System.out.println("Hyundai Start");
    }

    @Override
    public void dynamicSelect() {
        System.out.println("Change Mode : Comfort");
    }
}

public class Driver {
    public static void main(String[] args) {
        Car benz = new Benz();
        benz.autoStart();
        Car hyundai = new Hyundai();
        hyundai.autoStart();
    }
}

/**
 출력 결과 :
 Benz Start
 악셀레이터 밟기
 Change Mode : Sport
 Hyundai Start
 악셀레이터 밟기
 Change Mode : Comfort
 */
```