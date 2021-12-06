### 프로토 타입 패턴

---

프로토 타입의 의미는 특정 제품을 만드는 과정에서 임시로 만드는 것

    이미 객체가 존재할 때 사용되며 clone 메소드를 이용하여 생성
    생성하는 객체는 반드시 clone() 에 대해 정의되어 있어야 한다
    동일한 객체에 대해 잦은 수정이 있는 경우 필요한 만큼 생성자로 생성하지 않고
    객체를 복사하여 수정 작업을 진행할 수 있다

```java
public class Item implements Cloneable {
    private String itemName;
    private int count;
    private int price;

    public String getItemName() {
        return itemName;
    }

    public void setItemName(String itemName) {
        this.itemName = itemName;
    }

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public Item(String itemName, int count, int price) {
        this.itemName = itemName;
        this.count = count;
        this.price = price;
    }

    @Override
    public String toString() {
        return "Item {" + "itemName='" + itemName + '\'' +  ", count=" + count +  ", price=" + price +  '}';
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return this;
    }
}

public class ItemMain {
    public static void main(String[] args) throws CloneNotSupportedException {
        Item item = new Item("맥북", 1, 2500000);
        Item itemClone = (Item) item.clone();
        System.out.println("클론 객체 : 실제 객체의 값 변경 전");
        System.out.println(itemClone.toString());
        item.setCount(2);
        item.setPrice(5000000);
        System.out.println("실제 객체");
        System.out.println(item.toString());
        System.out.println("클론 객체 : 실제 객체의 값 변경 후");
        System.out.println(itemClone.toString());
    }
}

/**
 출력 결과 :
 클론 객체 : 실제 객체의 값 변경 전
 Item {itemName='맥북', count=1, price=2500000}
 실제 객체
 Item {itemName='맥북', count=2, price=5000000}
 클론 객체 : 실제 객체의 값 변경 후
 Item {itemName='맥북', count=2, price=5000000}
 */
```