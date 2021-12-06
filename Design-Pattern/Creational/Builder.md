### 빌더 패턴

---

복잡한 객체 생성과 표현을 분리하는 패턴

    빌더 패턴을 사용하여 필수 필드에 대해서는 생성자에게 필드 생성 책임을 줄 수 있으며
    필수가 아닌 필드에 대해서는 빌드를 통해 생성하거나 생성하지 않을 수 있고
    참조는 객체, 생성은 빌더에 책임을 구분하여 개발자 실수에 의한 불변성을 막을 수 있다

```java
public class Coin {
    private String name;
    private int count;
    private int price;

    public String getName() {
        return name;
    }

    public int getCount() {
        return count;
    }

    public int getPrice() {
        return price;
    }

    public Coin(CoinBuilder coinBuilder) {
        this.name = coinBuilder.name;
        this.count = coinBuilder.count;
        this.price = coinBuilder.price;
    }

    @Override
    public String toString() {
        return "Coin {" + "name='" + name + '\'' +  ", count=" + count +  ", price=" + price +  '}';
    }

    public static class CoinBuilder {
        private String name;
        private int count;
        private int price;

        public CoinBuilder(String name, int count, int price) {
            this.name = name;
            this.count = count;
            this.price = price;
        }

        public Coin build() {
            return new Coin(this);
        }
    }
}

public class CoinMain {
    public static void main(String[] args) {
        Coin coin = new Coin.CoinBuilder("비트코인", 1, 3500).build();
        System.out.println(coin.toString());
    }
}

/**
 출력 결과 :
 Coin {name='비트코인', count=1, price=3500}
 */
```