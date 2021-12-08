### 프록시 패턴

---

    실행 하려는 인터페이스의 구현체에 대리자 역할의 구현체를 생성하여 실행
    프록시(대리자 역할의 구현체)를 통해서 흐름을 전달하고 데이터 조작은 하지 않는다

```java
public interface ServiceInterface {
    void serviceCall();
}

public class Service implements ServiceInterface {
    @Override
    public void serviceCall() {
        System.out.println("서비스 호출");
    }
}

public class Proxy implements ServiceInterface {
    ServiceInterface serviceInterface;

    @Override
    public void serviceCall() {
        serviceInterface = new Service();
        serviceInterface.serviceCall();
    }
}

public class Main {
    public static void main(String[] args) {
        ServiceInterface proxyServiceInterface = new Proxy();
        proxyServiceInterface.serviceCall();
    }
}

/**
 출력 결과 :
 서비스 호출
 */
```