# 装饰器模式

装饰器模式的目的是在不改变原有业务逻辑的基础上对业务进行扩展。

装饰器模式一般是在业务的迭代中演化使用的。比如有一个需求是这样的：查询显示书本的信息。

我们的代码是这样实现的：

书籍类：

```java
/**
 * 书籍类
 */
@Data
public class Book {
    private String bookName;
    private Double price;
}
```

接口：

```java
public interface BookService {
    /**获取书籍列表*/
    public List<Book> getBooks();
}
```

实现类：

```java
public class BookServiceImpl implements BookService {
    @Override
    public List<Book> getBooks() {
        Book book1 = new Book();
        book1.setBookName("设计模式");
        book1.setPrice(20.0);
        Book book2 = new Book();
        book2.setBookName("java编程思想");
        book2.setPrice(80.0);
        List<Book> bookList = Arrays.asList(book1,book2);
        return bookList;
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        BookService bookService = new BookServiceImpl();
        List<Book> books = bookService.getBooks();
        books.forEach(System.out::println);
    }
}
```

输出：

```less
Book(bookName=设计模式, price=20.0)
Book(bookName=java编程思想, price=80.0)
```

现在需求有所改动，要求书本打五折，不管是根据软件设计的开闭原则，还是说其他地方也在调用bookServiceImpl的实现，最好不要修改原有的业务逻辑。这时候我们就可以用装饰器模式进行扩展。

打五折的实现类：

```java
public class BookHalfDisCount implements BookService {
    private BookService bookService;

    public BookHalfDisCount(BookService bookService){
        this.bookService = bookService;
    }
    @Override
    public List<Book> getBooks() {
        List<Book> books = bookService.getBooks();
        books.forEach( book -> {
            book.setPrice(book.getPrice()*0.5);
        });
        return books;
    }
}
```

客户端这样调用：

```java
public class Client {
    public static void main(String[] args) {
        BookService bookService = new BookServiceImpl();
        List<Book> books = bookService.getBooks();
        books.forEach(System.out::println);

        // 打五折输出
        bookService = new BookHalfDisCount(new BookServiceImpl());
        List<Book> books1 = bookService.getBooks();
        books1.forEach(System.out::println);
    }
}
```

输出：

```less
Book(bookName=设计模式, price=20.0)
Book(bookName=java编程思想, price=80.0)
Book(bookName=设计模式, price=10.0)
Book(bookName=java编程思想, price=40.0)
```

如果需求在来一个打八折是同样的方案。这里原有的BookServiceImpl就是装饰的对象。BookHalfDisCount是装饰器。

但是考虑到如果在有一个打八折的，我们可以把装饰器提出来一个抽象类，因为每个装饰器都需要注入被装饰的对象。所以代码改进为：

装饰器抽象类：

```java
public abstract class AbstractBookDisCount implements BookService {
    protected BookService bookService;

    public AbstractBookDisCount(BookService bookService){
        this.bookService = bookService;
    }
}
```

打五折的装饰器

```java
public class BookHalfDisCount extends AbstractBookDisCount {

    public BookHalfDisCount(BookService bookService){
        super(bookService);
    }
    @Override
    public List<Book> getBooks() {
        List<Book> books = bookService.getBooks();
        books.forEach( book -> {
            book.setPrice(book.getPrice()*0.5);
        });
        return books;
    }
}
```

客户端的调用不变。

Java的IO流就是金典的装饰器模式