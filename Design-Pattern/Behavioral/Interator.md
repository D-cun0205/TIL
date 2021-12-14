### 이터레이터 패턴

---

다양한 자료구조 접근 방법은 다르며 이를 캡슐화 하여 동일한 방법으로 쉽게 접근하는 패턴  

    Aggregate : 집합체 정의
    ConcreteAggregate : Aggregate 구현체
    Iterator : 집합체 요소들에게 접근하기 위한 인터페이스 정의
    ConcreteIterator : Iterator 구현체

```java
public interface Aggregate {
    public abstract Iterator createIterator();
}

public class Book {
    private String name;

    public Book(String name) { this.name = name; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}

public class AggregateImplements implements Aggregate {
    private Book[] books;
    private int num = 0;

    public AggregateImplements(int idx) {
        books = new Book[idx];
    }

    public Book getBook(int idx) {
        return books[idx];
    }

    public void addBook(Book book) {
        this.books[num] = book;
        num++;
    }

    public int getLength() {
        return num;
    }

    @Override
    public Iterator createIterator() {
        return new BookIterator(this);
    }
}

public class BookIterator implements Iterator {

    private AggregateImplements aggregateImplements;
    private int idx;

    public BookIterator(AggregateImplements aggregateImplements) {
        this.aggregateImplements = aggregateImplements;
        idx = 0;
    }

    @Override
    public boolean hasNext() {
        if(idx < aggregateImplements.getLength())
            return true;
        else
            return false;
    }

    @Override
    public Object next() {
        Book book = this.aggregateImplements.getBook(idx);
        idx++;
        return book;
    }
}

public class Main {
    public static void main(String[] args) {
        AggregateImplements aggregateImplements = new AggregateImplements(5);
        aggregateImplements.addBook(new Book("스프링 프레임워크"));
        aggregateImplements.addBook(new Book("스프링 부트"));
        aggregateImplements.addBook(new Book("자바"));

        Iterator bookIterator = new BookIterator(aggregateImplements);
        while (bookIterator.hasNext()) {
            Book book = (Book) bookIterator.next();
            System.out.println(book.getName());
        }
    }
}

/**
 출력 결과 :
 스프링 프레임워크
 스프링 부트
 자바
 */
```