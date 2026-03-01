---
name: spring-boot-patterns
description: Spring Boot 应用程序的最佳实践和模式。当创建 controllers、services、repositories 或用户询问 Spring Boot 架构、REST APIs、异常处理或 JPA 模式时使用。
---

# Spring Boot Patterns 技能

Spring Boot 应用程序的最佳实践和模式。

## 何时使用
- 用户说 "create controller" / "add service" / "Spring Boot help"
- 审查 Spring Boot 代码
- 设置新的 Spring Boot 项目结构

## 项目结构

```
src/main/java/com/example/myapp/
├── MyAppApplication.java          # @SpringBootApplication
├── config/                        # 配置类
│   ├── SecurityConfig.java
│   └── WebConfig.java
├── controller/                    # REST controllers
│   └── UserController.java
├── service/                       # 业务逻辑
│   ├── UserService.java
│   └── impl/
│       └── UserServiceImpl.java
├── repository/                    # 数据访问
│   └── UserRepository.java
├── model/                         # Entities
│   └── User.java
├── dto/                           # Data transfer objects
│   ├── request/
│   │   └── CreateUserRequest.java
│   └── response/
│       └── UserResponse.java
├── exception/                     # 自定义异常
│   ├── ResourceNotFoundException.java
│   └── GlobalExceptionHandler.java
└── util/                          # 工具类
    └── DateUtils.java
```

---

## Controller 模式

### REST Controller 模板
```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor  // Lombok 用于构造器注入
public class UserController {

    private final UserService userService;

    @GetMapping
    public ResponseEntity<List<UserResponse>> getAll() {
        return ResponseEntity.ok(userService.findAll());
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getById(@PathVariable Long id) {
        return ResponseEntity.ok(userService.findById(id));
    }

    @PostMapping
    public ResponseEntity<UserResponse> create(
            @Valid @RequestBody CreateUserRequest request) {
        UserResponse created = userService.create(request);
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(created.getId())
            .toUri();
        return ResponseEntity.created(location).body(created);
    }

    @PutMapping("/{id}")
    public ResponseEntity<UserResponse> update(
            @PathVariable Long id,
            @Valid @RequestBody UpdateUserRequest request) {
        return ResponseEntity.ok(userService.update(id, request));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Controller 最佳实践

| Practice | Example |
|----------|---------|
| 版本化 API | `/api/v1/users` |
| 复数名词 | `/users` 而非 `/user` |
| HTTP 方法 | GET=读、POST=创建、PUT=更新、DELETE=删除 |
| 状态码 | 200=OK、201=Created、204=NoContent、404=NotFound |
| 验证 | 请求体上的 `@Valid` |

### ❌ 反模式
```java
// ❌ Controller 中的业务逻辑
@PostMapping
public User create(@RequestBody User user) {
    user.setCreatedAt(LocalDateTime.now());  // 逻辑属于 service
    return userRepository.save(user);         // 直接访问 repo
}

// ❌ 直接返回 entity（暴露内部实现）
@GetMapping("/{id}")
public User getById(@PathVariable Long id) {
    return userRepository.findById(id).get();
}
```

---

## Service 模式

### Service 接口 + 实现
```java
// Interface
public interface UserService {
    List<UserResponse> findAll();
    UserResponse findById(Long id);
    UserResponse create(CreateUserRequest request);
    UserResponse update(Long id, UpdateUserRequest request);
    void delete(Long id);
}

// Implementation
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)  // 默认只读
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;
    private final UserMapper userMapper;

    @Override
    public List<UserResponse> findAll() {
        return userRepository.findAll().stream()
            .map(userMapper::toResponse)
            .toList();
    }

    @Override
    public UserResponse findById(Long id) {
        return userRepository.findById(id)
            .map(userMapper::toResponse)
            .orElseThrow(() -> new ResourceNotFoundException("User", id));
    }

    @Override
    @Transactional  // 写事务
    public UserResponse create(CreateUserRequest request) {
        User user = userMapper.toEntity(request);
        User saved = userRepository.save(user);
        return userMapper.toResponse(saved);
    }

    @Override
    @Transactional
    public void delete(Long id) {
        if (!userRepository.existsById(id)) {
            throw new ResourceNotFoundException("User", id);
        }
        userRepository.deleteById(id);
    }
}
```

### Service 最佳实践

- Interface + Impl 以提高可测试性
- 类级别的 `@Transactional(readOnly = true)`
- 写方法使用 `@Transactional`
- 抛出领域异常，而非通用异常
- 使用 mappers（MapStruct）进行 entity ↔ DTO 转换

---

## Repository 模式

### JPA Repository
```java
public interface UserRepository extends JpaRepository<User, Long> {

    // 派生查询
    Optional<User> findByEmail(String email);

    List<User> findByActiveTrue();

    // 自定义查询
    @Query("SELECT u FROM User u WHERE u.department.id = :deptId")
    List<User> findByDepartmentId(@Param("deptId") Long departmentId);

    // Native query（谨慎使用）
    @Query(value = "SELECT * FROM users WHERE created_at > :date",
           nativeQuery = true)
    List<User> findRecentUsers(@Param("date") LocalDate date);

    // 存在性检查（比 findBy 更高效）
    boolean existsByEmail(String email);

    // 计数
    long countByActiveTrue();
}
```

### Repository 最佳实践

- 尽可能使用派生查询
- 单结果使用 `Optional`
- 存在性检查使用 `existsBy` 而非 `findBy`
- 避免原生查询，除非必要
- 使用 `@EntityGraph` 进行 fetch 优化

---

## DTO 模式

### Request/Response DTOs
```java
// 带验证的 Request DTO
public record CreateUserRequest(
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100)
    String name,

    @NotBlank
    @Email(message = "Invalid email format")
    String email,

    @NotNull
    @Min(18)
    Integer age
) {}

// Response DTO
public record UserResponse(
    Long id,
    String name,
    String email,
    LocalDateTime createdAt
) {}
```

### MapStruct Mapper
```java
@Mapper(componentModel = "spring")
public interface UserMapper {

    UserResponse toResponse(User entity);

    List<UserResponse> toResponseList(List<User> entities);

    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    User toEntity(CreateUserRequest request);
}
```

---

## 异常处理

### 自定义异常
```java
public class ResourceNotFoundException extends RuntimeException {

    public ResourceNotFoundException(String resource, Long id) {
        super(String.format("%s not found with id: %d", resource, id));
    }
}

public class BusinessException extends RuntimeException {

    private final String code;

    public BusinessException(String code, String message) {
        super(message);
        this.code = code;
    }
}
```

### 全局异常处理器
```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        log.warn("Resource not found: {}", ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("NOT_FOUND", ex.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
            MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .toList();
        return ResponseEntity.badRequest()
            .body(new ErrorResponse("VALIDATION_ERROR", errors.toString()));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        log.error("Unexpected error", ex);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred"));
    }
}

public record ErrorResponse(String code, String message) {}
```

---

## 配置模式

### Application Properties
```yaml
# application.yml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: ${DB_USER}
    password: ${DB_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: validate  # 生产环境中绝不要用 'create'！
    show-sql: false

app:
  jwt:
    secret: ${JWT_SECRET}
    expiration: 86400000
```

### Configuration Properties 类
```java
@Configuration
@ConfigurationProperties(prefix = "app.jwt")
@Validated
public class JwtProperties {

    @NotBlank
    private String secret;

    @Min(60000)
    private long expiration;

    // getters and setters
}
```

### Profile 特定配置
```
src/main/resources/
├── application.yml           # 通用配置
├── application-dev.yml       # 开发环境
├── application-test.yml      # 测试环境
└── application-prod.yml      # 生产环境
```

---

## 常用注解快速参考

| Annotation | 用途 |
|------------|---------|
| `@RestController` | REST controller（结合 @Controller + @ResponseBody） |
| `@Service` | 业务逻辑组件 |
| `@Repository` | 数据访问组件 |
| `@Configuration` | 配置类 |
| `@RequiredArgsConstructor` | Lombok：构造器注入 |
| `@Transactional` | 事务管理 |
| `@Valid` | 触发验证 |
| `@ConfigurationProperties` | 绑定属性到类 |
| `@Profile("dev")` | Profile 特定 bean |
| `@Scheduled` | 定时任务 |

---

## 测试模式

### Controller 测试（MockMvc）
```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void shouldReturnUser() throws Exception {
        when(userService.findById(1L))
            .thenReturn(new UserResponse(1L, "John", "john@example.com", null));

        mockMvc.perform(get("/api/v1/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("John"));
    }
}
```

### Service 测试
```java
@ExtendWith(MockitoExtension.class)
class UserServiceImplTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private UserMapper userMapper;

    @InjectMocks
    private UserServiceImpl userService;

    @Test
    void shouldThrowWhenUserNotFound() {
        when(userRepository.findById(1L)).thenReturn(Optional.empty());

        assertThatThrownBy(() -> userService.findById(1L))
            .isInstanceOf(ResourceNotFoundException.class);
    }
}
```

### 集成测试
```java
@SpringBootTest
@AutoConfigureMockMvc
@Testcontainers
class UserIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");

    @Autowired
    private MockMvc mockMvc;

    @Test
    void shouldCreateUser() throws Exception {
        mockMvc.perform(post("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"name": "John", "email": "john@example.com", "age": 25}
                    """))
            .andExpect(status().isCreated());
    }
}
```

---

## 快速参考卡

| Layer | 职责 | 注解 |
|-------|---------------|-------------|
| Controller | HTTP 处理、验证 | `@RestController`、`@Valid` |
| Service | 业务逻辑、事务 | `@Service`、`@Transactional` |
| Repository | 数据访问 | `@Repository`、extends `JpaRepository` |
| DTO | 数据传输 | 带验证注解的 Records |
| Config | 配置 | `@Configuration`、`@ConfigurationProperties` |
| Exception | 错误处理 | `@RestControllerAdvice` |
