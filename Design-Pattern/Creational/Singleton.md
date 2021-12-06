### 싱글톤 패턴

---

어플리케이션 런타임시 메모리에 하나만 생성 보장과 다른 메모리에서 접근 가능

    디폴트 생성자의 접근제한자를 private으로 선언하여 직접 생성하지 못하도록 제한
    getInstance()를 통해서 생성될 경우 동일한 인스턴스 보장

```java
public class SingletonObject {

    static SingletonObject singletonObject = null;

    private SingletonObject(){}

    public static SingletonObject getInstance() {
        if(singletonObject == null) {
            singletonObject = new SingletonObject();
        }
        return singletonObject;
    }
}

public class SingletonTest {
    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            SingletonObject singletonObject = SingletonObject.getInstance();
            System.out.println(singletonObject.toString());
        }
    }
}

/**
 출력 결과 :
 SingletonObject@42a57993
 SingletonObject@42a57993
 SingletonObject@42a57993
 SingletonObject@42a57993
 SingletonObject@42a57993
 */
```