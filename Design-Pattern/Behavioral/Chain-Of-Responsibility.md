### 책임 연쇄 패턴

---

    클래스가 작업에 대한 책임을 모두 가지지 않고 분할하여 결합을 느슨하게 하는 패턴
    특정 단위의 작업을 요청할 때 작업이 처리되는 시점(작업의 시작 ~ 끝)까지 반복 

```java
public enum OrderLv {
    BOTTOM (1), TIRE (2), TOP (3);

    private final int lvCode;

    OrderLv(int lvCode) { this.lvCode = lvCode; }
    public int getLvCode() { return this.lvCode; }
}

public class Car {
    private OrderLv orderLv;

    public Car(OrderLv orderLv) {
        this.orderLv = orderLv;
    }

    public OrderLv getOrderLv() { return orderLv; }
    public void setOrderLv(OrderLv orderLv) { this.orderLv = orderLv; }
}

public interface AssembleCar {
    void assembleDetail(Car car);
    void nextAssemble(Car car);
}

public class BottomAssemble implements AssembleCar {
    private AssembleCar assembleCar;

    public BottomAssemble(TireAssemble tireAssemble) {
        this.assembleCar = tireAssemble;
    }

    @Override
    public void assembleDetail(Car car) {
        System.out.println("여기서는 하체 조립");
    }

    @Override
    public void nextAssemble(Car car) {
        if (car.getOrderLv().getLvCode() == 1) {
            assembleDetail(car);
            car.setOrderLv(OrderLv.TIRE);
        }
        assembleCar.nextAssemble(car);
    }
}

public class TireAssemble implements AssembleCar {
    private AssembleCar assembleCar;

    public TireAssemble(TopAssemble topAssemble) {
        this.assembleCar = topAssemble;
    }

    @Override
    public void assembleDetail(Car car) {
        System.out.println("여기서는 타이어 조립");
    }

    @Override
    public void nextAssemble(Car car) {
        if (car.getOrderLv().getLvCode() == 2) {
            assembleDetail(car);
            car.setOrderLv(OrderLv.TOP);
        }
        assembleCar.nextAssemble(car);
    }
}

public class TopAssemble implements AssembleCar {

    private AssembleCar assembleCar;

    @Override
    public void assembleDetail(Car car) {
        System.out.println("여기서는 상체 조립");
    }

    @Override
    public void nextAssemble(Car car) {
        if (car.getOrderLv().getLvCode() == 3) {
            assembleDetail(car);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        AssembleCar assembleCar = new BottomAssemble(new TireAssemble(new TopAssemble()));
        System.out.println("첫번째 차 주문");
        assembleCar.nextAssemble(new Car(OrderLv.BOTTOM));
        System.out.println("두번째 차 주문(하체 조립은 되어 있음)");
        assembleCar.nextAssemble(new Car(OrderLv.TIRE));
    }
}

/*
 출력 결과 : 
 첫번째 차 주문
 여기서는 하체 조립
 여기서는 타이어 조립
 여기서는 상체 조립
 두번째 차 주문(하체 조립은 되어 있음)
 여기서는 타이어 조립
 여기서는 상체 조립
 */

```