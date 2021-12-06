### 어댑터 패턴

---

관계가 없는 서로 다른 인터페이스를 어댑터 클래스를 사용하여 관계를 생성

    어댑터 패턴을 이용하면 기존 소스코드의 수정 없이 관계를 맺을 수 있다
    관계를 맺기위한 클래스를 생성하여 연결해야 한다

```java
public interface GrandCar {
    void exhaustSound();
}

public class Lamborghini implements GrandCar {
    @Override
    public void exhaustSound() {
        System.out.println("부와아아아아앙");
    }
}

public interface CuteCar {
    void exhaustSound();
}

public class Tico implements CuteCar {
    @Override
    public void exhaustSound() {
        System.out.println("부우웅");
    }
}

public class AdapterCar implements CuteCar {

    GrandCar grandCar;

    public AdapterCar(GrandCar grandCar) {
        this.grandCar = grandCar;
    }

    @Override
    public void exhaustSound() {
        grandCar.exhaustSound();
    }
}

public class Driver {
    public static void main(String[] args) {
        CuteCar tico = new Tico();
        System.out.println("티코 출발");
        tico.exhaustSound();
        GrandCar lamborghini = new Lamborghini();
        System.out.println("람보르기니 출발");
        lamborghini.exhaustSound();
        //CuteCar로 Tico를 바로 생산하지 않고
        //AdapterCar 를 통해서 CuteCar 를 GrandCar 로 재정의
        //재정의 시 GrandCar의 구현체(Lamborghini)로 메소드를 재정의 
        //tuningTico를 생산
        CuteCar tuningTico = new AdapterCar(lamborghini);
        System.out.println("튜닝한 티코 출발");
        tuningTico.exhaustSound();
    }
}

/**
 출력 결과 :
 티코 출발
 부우웅
 toto
 람보르기니 출발
 부와아아아아앙
 튜닝한 티코 출발
 부와아아아아앙 
 */
```