---
name: jpa-patterns
description: Spring 应用程序的 JPA/Hibernate 模式和常见陷阱（N+1、lazy loading、事务、查询）。当有 JPA 性能问题时使用 (N+1, lazy loading, transactions, queries). Use when user has JPA performance issues, LazyInitializationException, or asks about entity relationships and fetching strategies.
---

# JPA Patterns 技能

Spring 应用程序中 JPA/Hibernate 的最佳实践和常见陷阱。

## 何时使用
- 用户提到 "N+1 problem" / "too many queries"
- LazyInitializationException 错误
- 关于获取策略的问题（EAGER vs LAZY）
- 事务管理问题
- 实体关系设计
- 查询优化

---

## 快速参考：常见问题

| 问题 | 症状 | 解决方案 |
|---------|---------|----------|
| N+1 queries | 许多 SELECT 语句 | JOIN FETCH、@EntityGraph |
| LazyInitializationException | 事务外错误 | Open Session in View、DTO projection、JOIN FETCH |
| Slow queries | 性能问题 | 分页、projections、索引 |
| Dirty checking overhead | 更新慢 | 只读事务、DTOs |
| Lost updates | 并发修改 | 乐观锁（@Version）|

---

## N+1 问题

> JPA 性能杀手 #1

### 问题所在

```java
// ❌ 坏：N+1 查询
@Entity
public class Author {
    @Id private Long id;
    private String name;

    @OneToMany(mappedBy = "author", fetch = FetchType.LAZY)
    private List<Book> books;
}

// 这段看似无害的代码...
List<Author> authors = authorRepository.findAll();  // 1 个查询
for (Author author : authors) {
    System.out.println(author.getBooks().size());   // N 个查询！
}
// 结果：1 + N 个查询（如果有 100 个作者 = 101 个查询）
```

### 解决方案 1：JOIN FETCH (JPQL)

```java
// ✅ 好：使用 JOIN FETCH 的单个查询
public interface AuthorRepository extends JpaRepository<Author, Long> {

    @Query("SELECT a FROM Author a JOIN FETCH a.books")
    List<Author> findAllWithBooks();
}

// 使用 - 单个查询
List<Author> authors = authorRepository.findAllWithBooks();
```

### 解决方案 2：@EntityGraph

```java
// ✅ 好：用于声明式获取的 EntityGraph
public interface AuthorRepository extends JpaRepository<Author, Long> {

    @EntityGraph(attributePaths = {"books"})
    List<Author> findAll();

    // 或使用命名图
    @EntityGraph(value = "Author.withBooks")
    List<Author> findAllWithBooks();
}

// 在实体上定义命名图
@Entity
@NamedEntityGraph(
    name = "Author.withBooks",
    attributeNodes = @NamedAttributeNode("books")
)
public class Author {
    // ...
}
```

### 解决方案 3：批量获取

```java
// ✅ 好：批量获取（Hibernate 特定）
@Entity
public class Author {

    @OneToMany(mappedBy = "author")
    @BatchSize(size = 25)  // 每次获取 25 个
    private List<Book> books;
}

// 或在 application.properties 中全局设置
spring.jpa.properties.hibernate.default_batch_fetch_size=25
```

### 检测 N+1

```yaml
# 启用 SQL 日志以检测 N+1
spring:
  jpa:
    show-sql: true
    properties:
      hibernate:
        format_sql: true

logging:
  level:
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
```

---

## 懒加载

### FetchType 基础

```java
@Entity
public class Order {

    // LAZY：仅在访问时加载（集合的默认值）
    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
    private List<OrderItem> items;

    // EAGER：始终立即加载（@ManyToOne、@OneToOne 的默认值）
    @ManyToOne(fetch = FetchType.EAGER)  // ⚠️ 通常不好
    private Customer customer;
}
```

### 最佳实践：默认使用 LAZY

```java
// ✅ 好：始终使用 LAZY，需要时获取
@Entity
public class Order {

    @ManyToOne(fetch = FetchType.LAZY)  // 覆盖 EAGER 默认值
    private Customer customer;

    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
    private List<OrderItem> items;
}
```

### LazyInitializationException

```java
// ❌ 坏：在事务外访问懒加载字段
@Service
public class OrderService {

    public Order getOrder(Long id) {
        return orderRepository.findById(id).orElseThrow();
    }
}

// 在 controller 中（无事务）
Order order = orderService.getOrder(1L);
order.getItems().size();  // 💥 LazyInitializationException!
```

### LazyInitializationException 的解决方案

**解决方案 1：在查询中使用 JOIN FETCH**
```java
// ✅ 在查询中获取所需的关联
@Query("SELECT o FROM Order o JOIN FETCH o.items WHERE o.id = :id")
Optional<Order> findByIdWithItems(@Param("id") Long id);
```

**解决方案 2：在 service 方法上使用 @Transactional**
```java
// ✅ 在访问时保持事务打开
@Service
public class OrderService {

    @Transactional(readOnly = true)
    public OrderDTO getOrderWithItems(Long id) {
        Order order = orderRepository.findById(id).orElseThrow();
        // 在事务内访问
        int itemCount = order.getItems().size();
        return new OrderDTO(order, itemCount);
    }
}
```

**解决方案 3：DTO Projection（推荐）**
```java
// ✅ 最佳：只返回你需要的内容
public interface OrderSummary {
    Long getId();
    String getStatus();
    int getItemCount();
}

@Query("SELECT o.id as id, o.status as status, SIZE(o.items) as itemCount " +
       "FROM Order o WHERE o.id = :id")
Optional<OrderSummary> findOrderSummary(@Param("id") Long id);
```

**解决方案 4：Open Session in View（不推荐）**
```yaml
# 在视图渲染期间保持 session 打开
# ⚠️ 可能掩盖 N+1 问题，谨慎使用
spring:
  jpa:
    open-in-view: true  # 默认为 true
```

---

## 事务

### 基本事务管理

```java
@Service
public class OrderService {

    // 只读：已优化，无 dirty checking
    @Transactional(readOnly = true)
    public Order findById(Long id) {
        return orderRepository.findById(id).orElseThrow();
    }

    // 写入：带 dirty checking 的完整事务
    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        Order order = new Order();
        // ... 设置属性
        return orderRepository.save(order);
    }

    // 显式回滚
    @Transactional(rollbackFor = Exception.class)
    public void processPayment(Long orderId) throws PaymentException {
        // 在任何异常时回滚，不仅仅是 RuntimeException
    }
}
```

### 事务传播

```java
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;

    @Transactional
    public void placeOrder(Order order) {
        orderRepository.save(order);

        // REQUIRED（默认）：使用现有事务或创建新事务
        paymentService.processPayment(order);

        // 如果 paymentService 抛出异常，整个订单回滚
    }
}

@Service
public class PaymentService {

    // REQUIRES_NEW：始终创建新事务
    // 如果失败，订单仍可保存
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void processPayment(Order order) {
        // 独立事务
    }

    // MANDATORY：必须在现有事务中运行
    @Transactional(propagation = Propagation.MANDATORY)
    public void updatePaymentStatus(Order order) {
        // 如果不存在事务则抛出异常
    }
}
```

### 常见事务错误

```java
// ❌ 坏：从同一类调用 @Transactional 方法
@Service
public class OrderService {

    public void processOrder(Long id) {
        updateOrder(id);  // @Transactional 被忽略！
    }

    @Transactional
    public void updateOrder(Long id) {
        // 因为内部调用，事务未启动
    }
}

// ✅ 好：注入 self 或使用单独的 service
@Service
public class OrderService {

    @Autowired
    private OrderService self;  // 或使用单独的 service

    public void processOrder(Long id) {
        self.updateOrder(id);  // 现在事务正常工作
    }

    @Transactional
    public void updateOrder(Long id) {
        // 事务正确启动
    }
}
```

---

## 实体关系

### OneToMany / ManyToOne

```java
// ✅ 好：具有正确映射的双向关联
@Entity
public class Author {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Book> books = new ArrayList<>();

    // 用于双向同步的辅助方法
    public void addBook(Book book) {
        books.add(book);
        book.setAuthor(this);
    }

    public void removeBook(Book book) {
        books.remove(book);
        book.setAuthor(null);
    }
}

@Entity
public class Book {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "author_id")
    private Author author;
}
```

### ManyToMany

```java
// ✅ 好：ManyToMany 使用 Set（而不是 List）以避免重复
@Entity
public class Student {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();

    public void addCourse(Course course) {
        courses.add(course);
        course.getStudents().add(this);
    }

    public void removeCourse(Course course) {
        courses.remove(course);
        course.getStudents().remove(this);
    }
}

@Entity
public class Course {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();
}
```

### 实体的 equals() 和 hashCode()

```java
// ✅ 好：谨慎使用业务键或 ID
@Entity
public class Book {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NaturalId  // Hibernate 注解用于业务键
    @Column(unique = true, nullable = false)
    private String isbn;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Book book)) return false;
        return isbn != null && isbn.equals(book.isbn);
    }

    @Override
    public int hashCode() {
        return Objects.hash(isbn);  // 使用业务键，而非 ID
    }
}
```

---

## 查询优化

### 分页

```java
// ✅ 好：始终对大结果集进行分页
public interface OrderRepository extends JpaRepository<Order, Long> {

    Page<Order> findByStatus(OrderStatus status, Pageable pageable);

    // 带排序
    @Query("SELECT o FROM Order o WHERE o.status = :status")
    Page<Order> findByStatusSorted(
        @Param("status") OrderStatus status,
        Pageable pageable
    );
}

// 使用
Pageable pageable = PageRequest.of(0, 20, Sort.by("createdAt").descending());
Page<Order> orders = orderRepository.findByStatus(OrderStatus.PENDING, pageable);
```

### DTO Projections

```java
// ✅ 好：仅获取所需的列

// 基于接口的 projection
public interface OrderSummary {
    Long getId();
    String getCustomerName();
    BigDecimal getTotal();
}

@Query("SELECT o.id as id, o.customer.name as customerName, o.total as total " +
       "FROM Order o WHERE o.status = :status")
List<OrderSummary> findOrderSummaries(@Param("status") OrderStatus status);

// 基于类的 projection（DTO）
public record OrderDTO(Long id, String customerName, BigDecimal total) {}

@Query("SELECT new com.example.dto.OrderDTO(o.id, o.customer.name, o.total) " +
       "FROM Order o WHERE o.status = :status")
List<OrderDTO> findOrderDTOs(@Param("status") OrderStatus status);
```

### 批量操作

```java
// ✅ 好：批量更新而非加载实体
public interface OrderRepository extends JpaRepository<Order, Long> {

    @Modifying
    @Query("UPDATE Order o SET o.status = :status WHERE o.createdAt < :date")
    int updateOldOrdersStatus(
        @Param("status") OrderStatus status,
        @Param("date") LocalDateTime date
    );

    @Modifying
    @Query("DELETE FROM Order o WHERE o.status = :status AND o.createdAt < :date")
    int deleteOldOrders(
        @Param("status") OrderStatus status,
        @Param("date") LocalDateTime date
    );
}

// 使用
@Transactional
public void archiveOldOrders() {
    LocalDateTime threshold = LocalDateTime.now().minusYears(1);
    int updated = orderRepository.updateOldOrdersStatus(
        OrderStatus.ARCHIVED,
        threshold
    );
    log.info("Archived {} orders", updated);
}
```

---

## 乐观锁

### 防止丢失更新

```java
// ✅ 好：使用 @Version 进行乐观锁
@Entity
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Version
    private Long version;

    private OrderStatus status;
    private BigDecimal total;
}

// 当两个用户更新同一订单时：
// 用户 1：加载订单（version=1），修改，保存 → version 变为 2
// 用户 2：加载订单（version=1），修改，保存 → OptimisticLockException!
```

### 处理 OptimisticLockException

```java
@Service
public class OrderService {

    @Transactional
    public Order updateOrder(Long id, UpdateOrderRequest request) {
        try {
            Order order = orderRepository.findById(id).orElseThrow();
            order.setStatus(request.getStatus());
            return orderRepository.save(order);
        } catch (OptimisticLockException e) {
            throw new ConcurrentModificationException(
                "Order was modified by another user. Please refresh and try again."
            );
        }
    }

    // 或使用重试
    @Retryable(value = OptimisticLockException.class, maxAttempts = 3)
    @Transactional
    public Order updateOrderWithRetry(Long id, UpdateOrderRequest request) {
        Order order = orderRepository.findById(id).orElseThrow();
        order.setStatus(request.getStatus());
        return orderRepository.save(order);
    }
}
```

---

## 常见错误

### 1. 级联误用

```java
// ❌ 坏：在 @ManyToOne 上使用 CascadeType.ALL
@Entity
public class Book {
    @ManyToOne(cascade = CascadeType.ALL)  // 危险！
    private Author author;
}
// 删除一本书可能会删除作者！

// ✅ 好：仅从父级到子级级联
@Entity
public class Author {
    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Book> books;
}
```

### 2. 缺少索引

```java
// ❌ 坏：在非索引列上频繁查询
@Query("SELECT o FROM Order o WHERE o.customerEmail = :email")
List<Order> findByCustomerEmail(@Param("email") String email);

// ✅ 好：添加索引
@Entity
@Table(indexes = @Index(name = "idx_order_customer_email", columnList = "customerEmail"))
public class Order {
    private String customerEmail;
}
```

### 3. toString() 包含懒加载字段

```java
// ❌ 坏：toString 包含懒加载集合
@Entity
public class Author {
    @OneToMany(mappedBy = "author", fetch = FetchType.LAZY)
    private List<Book> books;

    @Override
    public String toString() {
        return "Author{id=" + id + ", books=" + books + "}";  // 触发懒加载！
    }
}

// ✅ 好：从 toString 中排除懒加载字段
@Override
public String toString() {
    return "Author{id=" + id + ", name='" + name + "'}";
}
```

---

## 性能清单

审查 JPA 代码时，检查：

- [ ] 无 N+1 查询（使用 JOIN FETCH 或 @EntityGraph）
- [ ] 默认 LAZY 获取（特别是 @ManyToOne）
- [ ] 大结果集的分页
- [ ] 只读查询的 DTO projections
- [ ] 批量更新/删除的批量操作
- [ ] 并发访问实体的 @Version
- [ ] 频繁查询列上的索引
- [ ] toString() 中无懒加载字段
- [ ] 适用时的只读事务

---

## 相关技能

- `spring-boot-patterns` - Spring Boot controller/service 模式
- `java-code-review` - 通用代码审查清单
- `clean-code` - 代码质量原则
