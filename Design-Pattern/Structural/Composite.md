### 컴포지트 패턴

---

    단일 객체와 객체들을 가지는 집합 객체를 같은 타입으로 취급하고 트리 구조로 개체들을 엮는다
    Composite, Leaf 클래스는 Component를 구현하여 같은 타입으로 인해 트리구조로 생성 가능하다
    

```java
public interface Component {
    void getName();
}

public class Composite implements Component{
    private String name;
    private List<Component> childList;

    @Override
    public void getName() {
        System.out.println("Component : " + name);
    }

    public void addLeaf(Component component) {
        childList.add(component);
    }
}

public class Leaf implements Component {
    private String name;

    @Override
    public void getName() {
        System.out.println("Leaf : " + name);
    }
}

public class Client {
    public static void main(String[] args) {
        Composite composite = new Composite();
        composite.addLeaf(new Leaf());
        composite.addLeaf(new Leaf());
        composite.addLeaf(new Composite());
        Composite upperComposite = new Composite();
        upperComposite.addLeaf(composite);
    }
}
```