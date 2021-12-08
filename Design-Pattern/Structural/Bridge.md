### 브릿지 패턴

---

    추상화를 구현으로부터 분리하여 각각 독립적으로 변화할 수 있도록 하는 패턴

```java
public interface Color {
    void paintColor();
}

public class RedColor implements Color {
    @Override
    public void paintColor() {
        System.out.println("Red");
    }
}

public class GreenColor implements Color {
    @Override
    public void paintColor() {
        System.out.println("Green");
    }
}

public abstract class Shape {
    protected Color color;

    public Shape(Color color) {
        this.color = color;
    }

    abstract public void paintColor();
}

public class Triangle extends Shape {
    public Triangle(Color color) {
        super(color);
    }

    @Override
    public void paintColor() {
        System.out.println("Triangle Paint Color");
        color.paintColor();
    }
}

public class Circle extends Shape {
    public Circle(Color color) {
        super(color);
    }

    @Override
    public void paintColor() {
        System.out.println("Circle Paint Color");
        color.paintColor();
    }
}

public class Test {
    public static void main(String[] args) {
        Triangle triangle = new Triangle(new RedColor());
        triangle.paintColor();
        Circle circle = new Circle(new GreenColor());
        circle.paintColor();
    }
}

/** 
 출력 결과:
 Triangle Paint Color
 Red
 Circle Paint Color
 Green
 */
```