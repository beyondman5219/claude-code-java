---
name: design-patterns
description: 常见设计模式及 Java 示例（Factory、Builder、Strategy、Observer、Decorator 等）。当用户询问 "implement pattern" (Factory, Builder, Strategy, Observer, Decorator, etc.). Use when user asks "implement pattern", "use factory", "strategy pattern", or when designing extensible components.
---

# Design Patterns 技能

带有现代示例的 Java 实用设计模式参考。

## 何时使用
- 用户要求实现特定模式
- 设计可扩展/灵活的组件
- 重构僵化的代码结构
- 代码审查建议使用模式

---

## 快速参考：何时使用什么

| 问题 | 模式 |
|---------|---------|
| 复杂对象构造 | **Builder** |
| 在不指定类的情况下创建对象 | **Factory** |
| 多种算法，运行时交换 | **Strategy** |
| 在不更改类的情况下添加行为 | **Decorator** |
| 通知多个对象更改 | **Observer** |
| 确保单个实例 | **Singleton** |
| 转换不兼容的接口 | **Adapter** |
| 定义算法骨架 | **Template Method** |

---

## 创建型模式

### Builder

**使用场景：** 对象有许多参数，其中一些是可选的。

```java
// ❌ 伸缩构造函数反模式
public class User {
    public User(String name) { }
    public User(String name, String email) { }
    public User(String name, String email, int age) { }
    public User(String name, String email, int age, String phone) { }
    // ... 构造函数爆炸
}

// ✅ Builder 模式
public class User {
    private final String name;      // 必需
    private final String email;     // 必需
    private final int age;          // 可选
    private final String phone;     // 可选
    private final String address;   // 可选

    private User(Builder builder) {
        this.name = builder.name;
        this.email = builder.email;
        this.age = builder.age;
        this.phone = builder.phone;
        this.address = builder.address;
    }

    public static Builder builder(String name, String email) {
        return new Builder(name, email);
    }

    public static class Builder {
        // 必需
        private final String name;
        private final String email;
        // 带默认值的可选
        private int age = 0;
        private String phone = "";
        private String address = "";

        private Builder(String name, String email) {
            this.name = name;
            this.email = email;
        }

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public Builder phone(String phone) {
            this.phone = phone;
            return this;
        }

        public Builder address(String address) {
            this.address = address;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }
}

// 使用
User user = User.builder("John", "john@example.com")
    .age(30)
    .phone("+1234567890")
    .build();
```

**使用 Lombok：**
```java
@Builder
@Getter
public class User {
    private final String name;
    private final String email;
    @Builder.Default private int age = 0;
    private String phone;
}
```

---

### Factory Method

**使用场景：** 需要在不指定确切类的情况下创建对象。

```java
// ✅ Factory Method 模式
public interface Notification {
    void send(String message);
}

public class EmailNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

public class SmsNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}

public class PushNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("Push: " + message);
    }
}

// Factory
public class NotificationFactory {

    public static Notification create(String type) {
        return switch (type.toUpperCase()) {
            case "EMAIL" -> new EmailNotification();
            case "SMS" -> new SmsNotification();
            case "PUSH" -> new PushNotification();
            default -> throw new IllegalArgumentException("Unknown type: " + type);
        };
    }
}

// 使用
Notification notification = NotificationFactory.create("EMAIL");
notification.send("Hello!");
```

**使用 Spring（推荐）：**
```java
public interface NotificationSender {
    void send(String message);
    String getType();
}

@Component
public class EmailSender implements NotificationSender {
    @Override public void send(String message) { /* ... */ }
    @Override public String getType() { return "EMAIL"; }
}

@Component
public class SmsSender implements NotificationSender {
    @Override public void send(String message) { /* ... */ }
    @Override public String getType() { return "SMS"; }
}

@Component
public class NotificationFactory {
    private final Map<String, NotificationSender> senders;

    public NotificationFactory(List<NotificationSender> senderList) {
        this.senders = senderList.stream()
            .collect(Collectors.toMap(
                NotificationSender::getType,
                Function.identity()
            ));
    }

    public NotificationSender getSender(String type) {
        return Optional.ofNullable(senders.get(type))
            .orElseThrow(() -> new IllegalArgumentException("Unknown: " + type));
    }
}
```

---

### Singleton

**使用场景：** 只需要一个实例（谨慎使用！）。

```java
// ✅ 现代 singleton（基于枚举，线程安全）
public enum DatabaseConnection {
    INSTANCE;

    private Connection connection;

    DatabaseConnection() {
        // 初始化连接
    }

    public Connection getConnection() {
        return connection;
    }
}

// 使用
Connection conn = DatabaseConnection.INSTANCE.getConnection();
```

**使用 Spring（推荐）：**
```java
@Component  // 默认 scope 是 singleton
public class DatabaseConnection {
    // Spring 管理单个实例
}
```

**警告：** Singleton 可能有问题：
- 难以测试（全局状态）
- 隐藏的依赖
- 考虑改用依赖注入

---

## 行为型模式

### Strategy

**使用场景：** 同一操作的多种算法，需要在运行时交换。

```java
// ✅ Strategy 模式
public interface PaymentStrategy {
    void pay(BigDecimal amount);
}

public class CreditCardPayment implements PaymentStrategy {
    private final String cardNumber;

    public CreditCardPayment(String cardNumber) {
        this.cardNumber = cardNumber;
    }

    @Override
    public void pay(BigDecimal amount) {
        System.out.println("Paid " + amount + " with card " + cardNumber);
    }
}

public class PayPalPayment implements PaymentStrategy {
    private final String email;

    public PayPalPayment(String email) {
        this.email = email;
    }

    @Override
    public void pay(BigDecimal amount) {
        System.out.println("Paid " + amount + " via PayPal: " + email);
    }
}

public class CryptoPayment implements PaymentStrategy {
    private final String walletAddress;

    public CryptoPayment(String walletAddress) {
        this.walletAddress = walletAddress;
    }

    @Override
    public void pay(BigDecimal amount) {
        System.out.println("Paid " + amount + " to wallet: " + walletAddress);
    }
}

// Context
public class ShoppingCart {
    private PaymentStrategy paymentStrategy;

    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }

    public void checkout(BigDecimal total) {
        paymentStrategy.pay(total);
    }
}

// 使用
ShoppingCart cart = new ShoppingCart();
cart.setPaymentStrategy(new CreditCardPayment("4111-1111-1111-1111"));
cart.checkout(new BigDecimal("99.99"));

// 在运行时更改策略
cart.setPaymentStrategy(new PayPalPayment("user@example.com"));
cart.checkout(new BigDecimal("49.99"));
```

**使用 Java 8+（函数式）：**
```java
// 策略作为函数式接口
@FunctionalInterface
public interface PaymentStrategy {
    void pay(BigDecimal amount);
}

// 使用 lambdas
PaymentStrategy creditCard = amount ->
    System.out.println("Card payment: " + amount);

PaymentStrategy paypal = amount ->
    System.out.println("PayPal payment: " + amount);

cart.setPaymentStrategy(creditCard);
```

---

### Observer

**使用场景：** 对象需要被通知另一个对象的更改。

```java
// ✅ Observer 模式（现代 Java）
public interface OrderObserver {
    void onOrderPlaced(Order order);
}

public class OrderService {
    private final List<OrderObserver> observers = new ArrayList<>();

    public void addObserver(OrderObserver observer) {
        observers.add(observer);
    }

    public void removeObserver(OrderObserver observer) {
        observers.remove(observer);
    }

    public void placeOrder(Order order) {
        // 处理订单
        saveOrder(order);

        // 通知所有观察者
        observers.forEach(observer -> observer.onOrderPlaced(order));
    }
}

// 观察者
public class InventoryService implements OrderObserver {
    @Override
    public void onOrderPlaced(Order order) {
        // 减少库存
        order.getItems().forEach(item ->
            reduceStock(item.getProductId(), item.getQuantity())
        );
    }
}

public class EmailNotificationService implements OrderObserver {
    @Override
    public void onOrderPlaced(Order order) {
        sendConfirmationEmail(order.getCustomerEmail(), order);
    }
}

public class AnalyticsService implements OrderObserver {
    @Override
    public void onOrderPlaced(Order order) {
        trackOrderEvent(order);
    }
}

// 设置
OrderService orderService = new OrderService();
orderService.addObserver(new InventoryService());
orderService.addObserver(new EmailNotificationService());
orderService.addObserver(new AnalyticsService());
```

**使用 Spring Events（推荐）：**
```java
// Event
public record OrderPlacedEvent(Order order) {}

// Publisher
@Service
public class OrderService {
    private final ApplicationEventPublisher eventPublisher;

    public void placeOrder(Order order) {
        saveOrder(order);
        eventPublisher.publishEvent(new OrderPlacedEvent(order));
    }
}

// Listeners（观察者）
@Component
public class InventoryListener {
    @EventListener
    public void handleOrderPlaced(OrderPlacedEvent event) {
        // 减少库存
    }
}

@Component
public class EmailListener {
    @EventListener
    public void handleOrderPlaced(OrderPlacedEvent event) {
        // 发送邮件
    }

    @EventListener
    @Async  // 异步处理
    public void handleOrderPlacedAsync(OrderPlacedEvent event) {
        // 异步发送邮件
    }
}
```

---

### Template Method

**使用场景：** 定义算法骨架，让子类填充步骤。

```java
// ✅ Template Method 模式
public abstract class DataProcessor {

    // 模板方法 - 定义算法
    public final void process() {
        readData();
        processData();
        writeData();
        if (shouldNotify()) {
            notifyCompletion();
        }
    }

    // 子类要实现的步骤
    protected abstract void readData();
    protected abstract void processData();
    protected abstract void writeData();

    // Hook - 可选覆盖
    protected boolean shouldNotify() {
        return true;
    }

    protected void notifyCompletion() {
        System.out.println("Processing completed!");
    }
}

public class CsvDataProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading CSV file...");
    }

    @Override
    protected void processData() {
        System.out.println("Processing CSV data...");
    }

    @Override
    protected void writeData() {
        System.out.println("Writing to database...");
    }
}

public class ApiDataProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Fetching from API...");
    }

    @Override
    protected void processData() {
        System.out.println("Transforming API response...");
    }

    @Override
    protected void writeData() {
        System.out.println("Writing to cache...");
    }

    @Override
    protected boolean shouldNotify() {
        return false;  // 覆盖 hook
    }
}

// 使用
DataProcessor csvProcessor = new CsvDataProcessor();
csvProcessor.process();

DataProcessor apiProcessor = new ApiDataProcessor();
apiProcessor.process();
```

---

## 结构型模式

### Decorator

**使用场景：** 在不修改现有类的情况下动态添加行为。

```java
// ✅ Decorator 模式
public interface Coffee {
    String getDescription();
    BigDecimal getCost();
}

public class SimpleCoffee implements Coffee {
    @Override
    public String getDescription() {
        return "Coffee";
    }

    @Override
    public BigDecimal getCost() {
        return new BigDecimal("2.00");
    }
}

// 基础 decorator
public abstract class CoffeeDecorator implements Coffee {
    protected final Coffee coffee;

    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }

    @Override
    public String getDescription() {
        return coffee.getDescription();
    }

    @Override
    public BigDecimal getCost() {
        return coffee.getCost();
    }
}

// 具体 decorators
public class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }

    @Override
    public BigDecimal getCost() {
        return coffee.getCost().add(new BigDecimal("0.50"));
    }
}

public class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Sugar";
    }

    @Override
    public BigDecimal getCost() {
        return coffee.getCost().add(new BigDecimal("0.20"));
    }
}

public class WhippedCreamDecorator extends CoffeeDecorator {
    public WhippedCreamDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Whipped Cream";
    }

    @Override
    public BigDecimal getCost() {
        return coffee.getCost().add(new BigDecimal("0.70"));
    }
}

// 使用 - 组合 decorators
Coffee coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);
coffee = new WhippedCreamDecorator(coffee);

System.out.println(coffee.getDescription());  // Coffee, Milk, Sugar, Whipped Cream
System.out.println(coffee.getCost());         // 3.40
```

**Java I/O 使用 Decorator：**
```java
// Java 中的经典示例
BufferedReader reader = new BufferedReader(
    new InputStreamReader(
        new FileInputStream("file.txt")
    )
);
```

---

### Adapter

**使用场景：** 使不兼容的接口一起工作。

```java
// ✅ Adapter 模式

// 我们的代码使用的现有接口
public interface MediaPlayer {
    void play(String filename);
}

// 遗留/第三方接口
public class LegacyAudioPlayer {
    public void playMp3(String filename) {
        System.out.println("Playing MP3: " + filename);
    }
}

public class AdvancedVideoPlayer {
    public void playMp4(String filename) {
        System.out.println("Playing MP4: " + filename);
    }

    public void playAvi(String filename) {
        System.out.println("Playing AVI: " + filename);
    }
}

// Adapters
public class Mp3PlayerAdapter implements MediaPlayer {
    private final LegacyAudioPlayer legacyPlayer = new LegacyAudioPlayer();

    @Override
    public void play(String filename) {
        legacyPlayer.playMp3(filename);
    }
}

public class VideoPlayerAdapter implements MediaPlayer {
    private final AdvancedVideoPlayer videoPlayer = new AdvancedVideoPlayer();

    @Override
    public void play(String filename) {
        if (filename.endsWith(".mp4")) {
            videoPlayer.playMp4(filename);
        } else if (filename.endsWith(".avi")) {
            videoPlayer.playAvi(filename);
        }
    }
}

// 使用
MediaPlayer mp3Player = new Mp3PlayerAdapter();
mp3Player.play("song.mp3");

MediaPlayer videoPlayer = new VideoPlayerAdapter();
videoPlayer.play("movie.mp4");
```

---

## 模式选择指南

| 情况 | 考虑 |
|-----------|----------|
| 对象创建复杂 | Builder、Factory |
| 需要动态添加功能 | Decorator |
| 算法的多种实现 | Strategy |
| 对状态变化做出反应 | Observer |
| 与遗留代码集成 | Adapter |
| 通用算法，步骤不同 | Template Method |
| 需要单个实例 | Singleton（谨慎使用）|

---

## 要避免的反模式

| 反模式 | 问题 | 更好的方法 |
|--------------|---------|-----------------|
| Singleton 滥用 | 全局状态，难以测试 | 依赖注入 |
| Factory 到处用 | 过度工程 | 如果类型已知，使用简单的 `new` |
| 深层 decorator 链 | 难以调试 | 保持链简短，考虑组合 |
| Observer 与许多事件 | 意大利面式通知 | 事件总线，清晰的事件层次结构 |

---

## 相关技能

- `solid-principles` - 模式帮助实现的设计原则
- `clean-code` - 代码级最佳实践
- `spring-boot-patterns` - Spring 特定实现
