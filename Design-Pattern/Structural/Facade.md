### 퍼사드 패턴

---

    서브 객체들의 기능을 통합한 인터페이스를 제공하여 복잡한 시스템을 단순화 하는 패턴

```java
public interface Race {
    void start();
}

public class Lamborghini implements Race {
    @Override
    public void start() {
        System.out.println("람보르기니 출발 : 흐ㅏ후아ㅏㅘ어ㅗ아아앙ㅇ!!!!!!");
    }
}

public class Benz implements Race {
    @Override
    public void start() {
        System.out.println("벤츠 출발 : 부와아아아아앙!!!!!!!!!!!");
    }
}

public class Volvo implements Race {
    @Override
    public void start() {
        System.out.println("볼보 출발 : 부우우웅~");
    }
}

public class StartPacade {
    Lamborghini lamborghini;
    Benz benz;
    Volvo volvo;

    public StartPacade() {
        lamborghini = new Lamborghini();
        benz = new Benz();
        volvo = new Volvo();
    }

    public void start() {
        lamborghini.start();
        benz.start();
        volvo.start();
    }
}

public class Start {
    public static void main(String[] args) {
        StartPacade pacade = new StartPacade();
        pacade.start();
    }
}

/**
 출력 결과 :
 람보르기니 출발 : 흐ㅏ후아ㅏㅘ어ㅗ아아앙ㅇ!!!!!!
 벤츠 출발 : 부와아아아아앙!!!!!!!!!!!
 볼보 출발 : 부우우웅~
 */
```