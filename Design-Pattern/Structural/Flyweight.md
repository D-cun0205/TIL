### 플라이웨이트 패턴

---

    특정 객체의 인스턴스를 사용하여 여러개의 인스턴스를 제공하는 패턴
    특정 값을 객체 생성 단위로 분기하여 객체의 생성을 제한하고 동일한 인스턴스를 보장한다 

```java
public interface Shape {
    void draw();
}

public class Circle implements Shape {
    private String color;

    public Circle(String color) {
        this.color = color;
    }

    @Override
    public void draw() {
        System.out.println("Create Circle! Color : " + color);
    }
}

public class ShapeFactory {
    private static final HashMap<String, Circle> circleMap = new HashMap<>();

    public static Shape getCircle(String color) {
        Circle circle = circleMap.get(color);
        if (circle == null) {
            System.out.println("객체 생성");
            circle = new Circle(color);
            circleMap.put(color, circle);
        }
        return circle;
    }
}

public class Artist {
    public static void main(String[] args) {
        String[] colors = {"Purple", "White", "Black"};
        for (int i = 0; i < 5; i++) {
            Circle circle = (Circle) ShapeFactory.getCircle(colors[(int) (Math.random()*3)]);
            System.out.println(circle);
            circle.draw();
        }
    }
}

/**
 출력 결과 :
 객체 생성
 Circle@75b84c92
 Create Circle! Color : Black
 객체 생성
 Circle@6bc7c054
 Create Circle! Color : Purple
 객체 생성
 Circle@232204a1
 Create Circle! Color : White
 Circle@6bc7c054
 Create Circle! Color : Purple
 Circle@75b84c92
 Create Circle! Color : Black
 */
```