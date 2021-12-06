### 팩토리 메소드 패턴

---

분기에 따른 객체의 생성을 직접하지 않고 팩토리라는 클래스에 위임하여 객체 생성  
* 팩토리 : 객체를 생성하는 공장


    ClassA 에서 직접 TypeFactory 의 메소드를 선언하여 사용할 수 있지만
    분기에 따른 객체 생성이 많아지면 동일한 소스코드가 클래스마다 추가된다
    그러므로 팩토리 메소드 패턴을 사용하여 불필요한 소스코드를 줄일 수 있다

```java
public abstract class Type {}

public class TypeA extends Type {
    public TypeA() {
        System.out.println("Type A 클래스 생성");
    }
}

public class TypeB extends Type {
    public TypeB() {
        System.out.println("Type B 클래스 생성");
    }
}

public class ClassA {
    public Type createType(String type) {
        TypeFactory factory = new TypeFactory();
        return factory.createType(type);
    }
}

public class TypeFactory {
    public Type createType(String type) {
        Type returnType = null;
        switch (type) {
            case "A" :
                returnType = new TypeA();
                break;
            case "B" :
                returnType = new TypeB();
                break;
        }
        return returnType;
    }
}

public class MainTest {
    public static void main(String[] args) {
        ClassA classA = new ClassA();
        classA.createType("B");
    }
}

/** 
 출력 결과 :
 Type B 클래스 생성
 */
```

