---
name: security-audit
description: 基于 OWASP Top 10 和安全编码实践的 Java 安全清单。适用于 Spring、Quarkus、Jakarta EE 和纯 Java。当审查代码安全、发布前或用户询问漏洞时使用。
---

# Security Audit 技能

基于 OWASP Top 10 和安全编码实践的 Java 应用程序安全清单。

## 何时使用
- 安全代码审查
- 生产发布之前
- 用户询问 "security"、"vulnerability"、"OWASP"
- 审查认证/授权代码
- 检查注入漏洞

---

## OWASP Top 10 快速参考

| # | Risk | Java 缓解措施 |
|---|------|-----------------|
| A01 | Broken Access Control | 基于角色的检查、默认拒绝 |
| A02 | Cryptographic Failures | 使用强算法、无硬编码密钥 |
| A03 | Injection | 参数化查询、输入验证 |
| A04 | Insecure Design | 威胁建模、安全默认值 |
| A05 | Security Misconfiguration | 禁用 debug、安全头 |
| A06 | Vulnerable Components | 依赖扫描、更新 |
| A07 | Authentication Failures | 强密码、MFA、会话管理 |
| A08 | Data Integrity Failures | 验证签名、安全反序列化 |
| A09 | Logging Failures | 记录安全事件、无敏感数据 |
| A10 | SSRF | 验证 URL、允许列表域名 |

---

## 输入验证（所有框架）

### Bean Validation (JSR 380)

适用于 Spring、Quarkus、Jakarta EE 和独立使用。

```java
// ✅ 好：在边界验证
public class CreateUserRequest {

    @NotNull(message = "Username is required")
    @Size(min = 3, max = 50, message = "Username must be 3-50 characters")
    @Pattern(regexp = "^[a-zA-Z0-9_]+$", message = "Username can only contain letters, numbers, underscore")
    private String username;

    @NotNull
    @Email(message = "Invalid email format")
    private String email;

    @NotNull
    @Size(min = 8, max = 100)
    @Pattern(regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d).*$",
             message = "Password must contain uppercase, lowercase, and number")
    private String password;

    @Min(value = 0, message = "Age cannot be negative")
    @Max(value = 150, message = "Invalid age")
    private Integer age;
}

// Controller/Resource - 触发验证
public Response createUser(@Valid CreateUserRequest request) {
    // request 已经验证
}
```

### 自定义验证器

```java
// 自定义注解
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = SafeHtmlValidator.class)
public @interface SafeHtml {
    String message() default "Contains unsafe HTML";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// 验证器实现
public class SafeHtmlValidator implements ConstraintValidator<SafeHtml, String> {

    private static final Pattern DANGEROUS_PATTERN = Pattern.compile(
        "<script|javascript:|on\\w+\\s*=", Pattern.CASE_INSENSITIVE
    );

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null) return true;
        return !DANGEROUS_PATTERN.matcher(value).find();
    }
}
```

### 允许列表 vs 拒绝列表

```java
// ❌ 坏：拒绝列表（攻击者找到绕过方法）
if (input.contains("<script>")) {
    throw new ValidationException("Invalid input");
}

// ✅ 好：允许列表（仅允许已知良好的）
private static final Pattern SAFE_NAME = Pattern.compile("^[a-zA-Z\\s'-]{1,100}$");

if (!SAFE_NAME.matcher(input).matches()) {
    throw new ValidationException("Invalid name format");
}
```

---

## SQL 注入防护

### JPA/Hibernate（所有框架）

```java
// ✅ 好：参数化查询
@Query("SELECT u FROM User u WHERE u.email = :email")
Optional<User> findByEmail(@Param("email") String email);

// ✅ 好：Criteria API
CriteriaBuilder cb = entityManager.getCriteriaBuilder();
CriteriaQuery<User> query = cb.createQuery(User.class);
Root<User> user = query.from(User.class);
query.where(cb.equal(user.get("email"), email));  // 安全

// ✅ 好：命名参数
TypedQuery<User> query = entityManager.createQuery(
    "SELECT u FROM User u WHERE u.status = :status", User.class);
query.setParameter("status", status);  // 安全

// ❌ 坏：字符串拼接
String jpql = "SELECT u FROM User u WHERE u.email = '" + email + "'";  // 易受攻击！
```

### 原生查询

```java
// ✅ 好：参数化原生查询
@Query(value = "SELECT * FROM users WHERE email = ?1", nativeQuery = true)
User findByEmailNative(String email);

// ❌ 坏：拼接的原生查询
String sql = "SELECT * FROM users WHERE email = '" + email + "'";  // 易受攻击！
```

### JDBC（纯 Java）

```java
// ✅ 好：PreparedStatement
String sql = "SELECT * FROM users WHERE email = ? AND status = ?";
try (PreparedStatement stmt = connection.prepareStatement(sql)) {
    stmt.setString(1, email);
    stmt.setString(2, status);
    ResultSet rs = stmt.executeQuery();
}

// ❌ 坏：带拼接的 Statement
String sql = "SELECT * FROM users WHERE email = '" + email + "'";  // 易受攻击！
Statement stmt = connection.createStatement();
stmt.executeQuery(sql);
```

---

## XSS 防护

### 输出编码

```java
// ✅ 好：使用模板引擎的自动转义

// Thymeleaf - 默认自动转义
<p th:text="${userInput}">...</p>  // 安全

// 要显示 HTML（危险，小心使用）：
<p th:utext="${trustedHtml}">...</p>  // 仅用于可信内容！

// ✅ 好：需要时手动编码
import org.owasp.encoder.Encode;

String safe = Encode.forHtml(userInput);
String safeJs = Encode.forJavaScript(userInput);
String safeUrl = Encode.forUriComponent(userInput);
```

**OWASP Encoder 的 Maven 依赖：**
```xml
<dependency>
    <groupId>org.owasp.encoder</groupId>
    <artifactId>encoder</artifactId>
    <version>1.2.3</version>
</dependency>
```

### 内容安全策略

```java
// 添加 CSP 头以防止内联脚本

// Spring Boot
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.headers(headers -> headers
            .contentSecurityPolicy(csp -> csp
                .policyDirectives("default-src 'self'; script-src 'self'; style-src 'self'")
            )
        );
        return http.build();
    }
}

// Servlet Filter（适用于所有地方）
@WebFilter("/*")
public class SecurityHeadersFilter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        HttpServletResponse response = (HttpServletResponse) res;
        response.setHeader("Content-Security-Policy", "default-src 'self'");
        response.setHeader("X-Content-Type-Options", "nosniff");
        response.setHeader("X-Frame-Options", "DENY");
        response.setHeader("X-XSS-Protection", "1; mode=block");
        chain.doFilter(req, res);
    }
}
```

---

## CSRF 防护

### Spring Security

```java
// 默认为浏览器客户端启用 CSRF
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // 对于使用 JWT 的 REST API（无状态）- 可以禁用 CSRF
            .csrf(csrf -> csrf.disable())

            // 对于带会话的浏览器应用 - 保持启用 CSRF
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            );
        return http.build();
    }
}
```

### Quarkus

```properties
# application.properties
quarkus.http.csrf.enabled=true
quarkus.http.csrf.cookie-name=XSRF-TOKEN
```

---

## 认证与授权

### 密码存储

```java
// ✅ 好：使用 BCrypt 或 Argon2
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.argon2.Argon2PasswordEncoder;

// BCrypt（广泛支持）
PasswordEncoder encoder = new BCryptPasswordEncoder(12);  // strength 12
String hash = encoder.encode(rawPassword);
boolean matches = encoder.matches(rawPassword, hash);

// Argon2（推荐用于新项目）
PasswordEncoder encoder = Argon2PasswordEncoder.defaultsForSpringSecurity_v5_8();
String hash = encoder.encode(rawPassword);

// ❌ 坏：MD5、SHA1、SHA256 不加盐
String hash = DigestUtils.md5Hex(password);  // 永远不要用于密码！
```

### 授权检查

```java
// ✅ 好：在 service 层检查授权
@Service
public class DocumentService {

    public Document getDocument(Long documentId, User currentUser) {
        Document doc = documentRepository.findById(documentId)
            .orElseThrow(() -> new NotFoundException("Document not found"));

        // 授权检查
        if (!doc.getOwnerId().equals(currentUser.getId()) &&
            !currentUser.hasRole("ADMIN")) {
            throw new AccessDeniedException("Not authorized to access this document");
        }

        return doc;
    }
}

// ❌ 坏：仅在 controller 层检查，信任用户输入
@GetMapping("/documents/{id}")
public Document getDocument(@PathVariable Long id) {
    return documentRepository.findById(id).orElseThrow();  // 无授权检查！
}
```

### Spring Security 注解

```java
@PreAuthorize("hasRole('ADMIN')")
public void adminOnly() { }

@PreAuthorize("hasRole('USER') and #userId == authentication.principal.id")
public void ownDataOnly(Long userId) { }

@PreAuthorize("@authService.canAccess(#documentId, authentication)")
public Document getDocument(Long documentId) { }
```

---

## 密钥管理

### 永远不要硬编码密钥

```java
// ❌ 坏：硬编码密钥
private static final String API_KEY = "sk-1234567890abcdef";
private static final String DB_PASSWORD = "admin123";

// ✅ 好：环境变量
String apiKey = System.getenv("API_KEY");

// ✅ 好：外部配置
@Value("${api.key}")
private String apiKey;

// ✅ 好：密钥管理器
@Autowired
private SecretsManager secretsManager;
String apiKey = secretsManager.getSecret("api-key");
```

### 配置文件

```yaml
# ✅ 好：引用环境变量
spring:
  datasource:
    password: ${DB_PASSWORD}

api:
  key: ${API_KEY}

# ❌ 坏：在 application.yml 中硬编码
spring:
  datasource:
    password: admin123  # 永远不要！
```

### .gitignore

```gitignore
# 永远不要提交这些
.env
*.pem
*.key
*credentials*
*secret*
application-local.yml
```

---

## 安全反序列化

### 避免 Java 序列化

```java
// ❌ 危险：Java ObjectInputStream
ObjectInputStream ois = new ObjectInputStream(untrustedInput);
Object obj = ois.readObject();  // 远程代码执行风险！

// ✅ 好：使用 JSON 与 Jackson
ObjectMapper mapper = new ObjectMapper();
// 禁用危险功能
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
mapper.activateDefaultTyping(
    LaissezFaireSubTypeValidator.instance,
    ObjectMapper.DefaultTyping.NON_FINAL
);  // 小心多态类型！

User user = mapper.readValue(json, User.class);
```

### Jackson 安全

```java
// ✅ 安全配置 Jackson
@Configuration
public class JacksonConfig {

    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();

        // 防止未知属性利用
        mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);

        // 不允许 JSON 中的类类型（防止小工具攻击）
        mapper.deactivateDefaultTyping();

        return mapper;
    }
}
```

---

## 依赖安全

### OWASP Dependency Check

**Maven:**
```xml
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>9.0.7</version>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <failBuildOnCVSS>7</failBuildOnCVSS>  <!-- 高严重性时失败 -->
    </configuration>
</plugin>
```

**运行：**
```bash
mvn dependency-check:check
# 报告：target/dependency-check-report.html
```

### 保持依赖更新

```bash
# 检查更新
mvn versions:display-dependency-updates

# 更新到最新
mvn versions:use-latest-releases
```

---

## 安全头

### 推荐的头

| Header | Value | Purpose |
|--------|-------|---------|
| `Content-Security-Policy` | `default-src 'self'` | 防止 XSS |
| `X-Content-Type-Options` | `nosniff` | 防止 MIME 嗅探 |
| `X-Frame-Options` | `DENY` | 防止点击劫持 |
| `Strict-Transport-Security` | `max-age=31536000` | 强制 HTTPS |
| `X-XSS-Protection` | `1; mode=block` | 传统 XSS 过滤器 |

### Spring Boot 配置

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.headers(headers -> headers
        .contentSecurityPolicy(csp -> csp.policyDirectives("default-src 'self'"))
        .frameOptions(frame -> frame.deny())
        .httpStrictTransportSecurity(hsts -> hsts.maxAgeInSeconds(31536000))
        .contentTypeOptions(Customizer.withDefaults())
    );
    return http.build();
}
```

---

## 记录安全事件

```java
// ✅ 记录安全相关事件
log.info("User login successful", kv("userId", userId), kv("ip", clientIp));
log.warn("Failed login attempt", kv("username", username), kv("ip", clientIp), kv("attempt", attemptCount));
log.warn("Access denied", kv("userId", userId), kv("resource", resourceId), kv("action", action));
log.error("Authentication failure", kv("reason", reason), kv("ip", clientIp));

// ❌ 永远不要记录敏感数据
log.info("Login: user={}, password={}", username, password);  // 永远不要！
log.debug("Request body: {}", requestWithCreditCard);  // 永远不要！
```

---

## 安全清单

### 代码审查

- [ ] 使用允许列表模式验证输入
- [ ] SQL 查询使用参数（无拼接）
- [ ] 为上下文编码输出（HTML、JS、URL）
- [ ] 在 service 层检查授权
- [ ] 无硬编码密钥
- [ ] 密码使用 BCrypt/Argon2 哈希
- [ ] 不记录敏感数据
- [ ] 启用 CSRF 保护（对于浏览器应用）

### 配置

- [ ] 强制 HTTPS
- [ ] 配置安全头
- [ ] 生产环境中禁用 debug/dev 功能
- [ ] 更改默认凭据
- [ ] 错误消息不泄露内部细节

### 依赖

- [ ] 无已知漏洞（OWASP 检查）
- [ ] 依赖是最新的
- [ ] 删除不必要的依赖

---

## 相关技能

- `java-code-review` - 通用代码审查
- `maven-dependency-audit` - 依赖漏洞扫描
- `logging-patterns` - 安全日志实践
