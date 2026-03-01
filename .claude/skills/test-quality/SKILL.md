---
name: test-quality
description: 使用 JUnit 5 和 AssertJ 断言编写高质量测试。当用户说 "add tests"、"write tests"、"improve test coverage" 或审查/创建 Java 代码的测试类时使用。
---

# Test Quality 技能 (JUnit 5 + AssertJ)

使用现代最佳实践为 Java 项目编写高质量、可维护的测试。

## 何时使用
- 编写新的测试类
- 审查/改进现有测试
- 用户要求 "add tests" / "improve test coverage"
- 代码审查提到缺少测试

## 框架偏好

### JUnit 5 (Jupiter)
```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Nested;
import static org.assertj.core.api.Assertions.*;
```

### AssertJ 优于标准断言
✅ **使用 AssertJ**：
```java
assertThat(plugin.getState())
    .as("Plugin should be started after initialization")
    .isEqualTo(PluginState.STARTED);

assertThat(plugins)
    .hasSize(3)
    .extracting(Plugin::getId)
    .containsExactly("plugin1", "plugin2", "plugin3");
```

❌ **避免 JUnit assertions**：
```java
assertEquals(PluginState.STARTED, plugin.getState()); // 可读性较差
assertTrue(plugins.size() == 3); // 失败描述不够详细
```

## 测试结构 (AAA 模式)

始终使用 Arrange-Act-Assert 模式：

```java
@Test
@DisplayName("Should load plugin from valid directory")
void shouldLoadPluginFromValidDirectory() {
    // Arrange - 设置测试数据和依赖
    Path pluginDir = Paths.get("test-plugins/valid-plugin");
    PluginLoader loader = new DefaultPluginLoader();

    // Act - 执行被测试的行为
    Plugin plugin = loader.load(pluginDir);

    // Assert - 验证结果
    assertThat(plugin)
        .isNotNull()
        .extracting(Plugin::getId, Plugin::getVersion)
        .containsExactly("test-plugin", "1.0.0");
}
```

## 命名约定

### 测试类名称
```java
// 被测试类：PluginManager
PluginManagerTest           // ✅ 简单、标准
PluginManagerShould         // ✅ BDD style（如果团队偏好）
TestPluginManager           // ❌ 避免
```

### 测试方法名称

**选项 1：should_expectedBehavior_when_condition**（描述性）
```java
@Test
void should_throwException_when_pluginDirectoryNotFound() { }

@Test
void should_returnEmptyList_when_noPluginsAvailable() { }

@Test
void should_loadPluginsInDependencyOrder_when_multipleDependencies() { }
```

**选项 2：自然语言 + @DisplayName**（代码更简洁）
```java
@Test
@DisplayName("Should load all plugins from directory")
void loadAllPlugins() { }

@Test
@DisplayName("Should throw exception when plugin descriptor is invalid")
void invalidPluginDescriptor() { }
```

## AssertJ 强大功能

### 集合断言
```java
// 基本集合检查
assertThat(plugins)
    .isNotEmpty()
    .hasSize(2)
    .doesNotContainNull();

// 高级过滤和提取
assertThat(plugins)
    .filteredOn(p -> p.getState() == PluginState.STARTED)
    .extracting(Plugin::getId)
    .containsExactlyInAnyOrder("plugin-a", "plugin-b");

// 所有元素匹配条件
assertThat(plugins)
    .allMatch(p -> p.getVersion() != null, "All plugins have version");
```

### 异常断言
```java
// 基本异常检查
assertThatThrownBy(() -> loader.load(invalidPath))
    .isInstanceOf(PluginException.class)
    .hasMessageContaining("Invalid plugin descriptor");

// 详细异常验证
assertThatThrownBy(() -> manager.startPlugin("missing-plugin"))
    .isInstanceOf(PluginException.class)
    .hasMessageContaining("Plugin not found")
    .hasCauseInstanceOf(IllegalArgumentException.class)
    .hasNoCause(); // 或验证原因链

// 使用 assertThatExceptionOfType（更易读）
assertThatExceptionOfType(PluginException.class)
    .isThrownBy(() -> loader.load(invalidPath))
    .withMessageContaining("Invalid")
    .withMessageMatching("Invalid .* descriptor");
```

### 对象断言
```java
// 提取并验证多个属性
assertThat(plugin)
    .isNotNull()
    .extracting("id", "version", "state")
    .containsExactly("my-plugin", "1.0", PluginState.STARTED);

// 使用方法引用（类型安全）
assertThat(plugin)
    .extracting(Plugin::getId, Plugin::getVersion, Plugin::getState)
    .containsExactly("my-plugin", "1.0", PluginState.STARTED);

// 逐字段比较
assertThat(actualPlugin)
    .usingRecursiveComparison()
    .isEqualTo(expectedPlugin);
```

### 软断言（多个检查）
```java
@Test
void shouldHaveValidPluginDescriptor() {
    SoftAssertions softly = new SoftAssertions();

    softly.assertThat(descriptor.getId())
        .as("Plugin ID")
        .isNotBlank()
        .matches("[a-z0-9-]+");

    softly.assertThat(descriptor.getVersion())
        .as("Plugin version")
        .matches("\\d+\\.\\d+\\.\\d+");

    softly.assertThat(descriptor.getDependencies())
        .as("Dependencies")
        .isNotNull()
        .doesNotContainNull();

    softly.assertAll(); // 所有断言都被评估，即使有些失败
}
```

### 字符串断言
```java
assertThat(errorMessage)
    .startsWith("Error:")
    .contains("plugin", "failed")
    .doesNotContain("success")
    .matches("Error: .* failed")
    .hasLineCount(3);
```

## 测试组织

### 嵌套测试以保持清晰
```java
@DisplayName("PluginManager")
class PluginManagerTest {

    private PluginManager manager;

    @BeforeEach
    void setUp() {
        manager = new DefaultPluginManager();
    }

    @Nested
    @DisplayName("when starting plugins")
    class WhenStartingPlugins {

        @Test
        @DisplayName("should start all plugins in dependency order")
        void shouldStartInDependencyOrder() {
            // 测试实现
        }

        @Test
        @DisplayName("should skip disabled plugins")
        void shouldSkipDisabledPlugins() {
            // 测试实现
        }

        @Test
        @DisplayName("should fail if circular dependency detected")
        void shouldFailOnCircularDependency() {
            // 测试实现
        }
    }

    @Nested
    @DisplayName("when stopping plugins")
    class WhenStoppingPlugins {

        @Test
        @DisplayName("should stop plugins in reverse dependency order")
        void shouldStopInReverseOrder() {
            // 测试实现
        }
    }
}
```

### 参数化测试
```java
@ParameterizedTest
@ValueSource(strings = {"1.0.0", "2.1.3", "10.0.0-SNAPSHOT"})
@DisplayName("Should accept valid semantic versions")
void shouldAcceptValidVersions(String version) {
    assertThat(VersionParser.parse(version))
        .isNotNull()
        .hasFieldOrPropertyWithValue("valid", true);
}

@ParameterizedTest
@CsvSource({
    "plugin-a, 1.0, STARTED",
    "plugin-b, 2.0, STOPPED",
    "plugin-c, 1.5, DISABLED"
})
@DisplayName("Should load plugin with expected state")
void shouldLoadPluginWithState(String id, String version, PluginState expectedState) {
    Plugin plugin = createPlugin(id, version);

    assertThat(plugin.getState()).isEqualTo(expectedState);
}

@ParameterizedTest
@MethodSource("invalidPluginDescriptors")
@DisplayName("Should reject invalid plugin descriptors")
void shouldRejectInvalidDescriptors(PluginDescriptor descriptor, String expectedError) {
    assertThatThrownBy(() -> validator.validate(descriptor))
        .hasMessageContaining(expectedError);
}

static Stream<Arguments> invalidPluginDescriptors() {
    return Stream.of(
        Arguments.of(descriptorWithoutId(), "Missing plugin ID"),
        Arguments.of(descriptorWithInvalidVersion(), "Invalid version format"),
        Arguments.of(descriptorWithEmptyId(), "Plugin ID cannot be empty")
    );
}
```

## 常见模式

### 使用 mock 测试（Mockito）
```java
@ExtendWith(MockitoExtension.class)
class PluginManagerTest {

    @Mock
    private PluginRepository repository;

    @Mock
    private PluginValidator validator;

    @InjectMocks
    private DefaultPluginManager manager;

    @Test
    @DisplayName("Should load plugins from repository")
    void shouldLoadPluginsFromRepository() {
        // Given
        List<PluginDescriptor> descriptors = List.of(
            createDescriptor("plugin1"),
            createDescriptor("plugin2")
        );
        when(repository.findAll()).thenReturn(descriptors);

        // When
        List<Plugin> plugins = manager.loadAll();

        // Then
        assertThat(plugins).hasSize(2);
        verify(repository).findAll();
        verify(validator, times(2)).validate(any(PluginDescriptor.class));
    }
}
```

### 使用 @BeforeEach 的测试夹具
```java
@BeforeEach
void setUp() throws IOException {
    // 为测试插件创建临时目录
    pluginDir = Files.createTempDirectory("test-plugins");

    // 使用测试配置初始化 plugin manager
    PluginConfig config = PluginConfig.builder()
        .pluginDirectory(pluginDir)
        .enableValidation(true)
        .build();

    pluginManager = new DefaultPluginManager(config);
}

@AfterEach
void tearDown() throws IOException {
    // 清理测试资源
    if (pluginManager != null) {
        pluginManager.stopAll();
    }
    if (pluginDir != null) {
        FileUtils.deleteDirectory(pluginDir.toFile());
    }
}
```

### 测试异步操作
```java
@Test
@DisplayName("Should complete async plugin loading")
void shouldCompleteAsyncLoading() {
    CompletableFuture<Plugin> future = manager.loadAsync(pluginPath);

    assertThat(future)
        .succeedsWithin(Duration.ofSeconds(5))
        .satisfies(plugin -> {
            assertThat(plugin.getState()).isEqualTo(PluginState.STARTED);
            assertThat(plugin.getId()).isNotBlank();
        });
}
```

## Token 优化

编写测试时：

### 1. 首先生成测试骨架
```java
// 阶段 1：将测试用例列为注释
// @Test void shouldLoadPlugin() { }
// @Test void shouldThrowExceptionForInvalidPlugin() { }
// @Test void shouldHandleMissingDependencies() { }
```

### 2. 增量实现
- 一次一个测试
- 每次后验证编译
- 运行测试以验证
- 如需要则重构

### 3. 重用模式
```java
// 将通用设置提取到辅助方法
private Plugin createTestPlugin(String id, String version) {
    return Plugin.builder()
        .id(id)
        .version(version)
        .build();
}
```

## 代码覆盖率指南

- **目标**：核心逻辑 80%+ 行覆盖率
- **重点**：业务逻辑、复杂算法、边缘情况
- **跳过**：简单的 getter/setter、POJO、生成的代码
- **测试**：快乐路径 + 错误条件 + 边界情况

### 测试什么
✅ **高优先级**：
- Public APIs
- 复杂的业务逻辑
- 错误处理
- 边缘情况和边界
- 集成点

❌ **低优先级**：
```java
// 简单的 getter/setter
public String getId() { return id; }
public void setId(String id) { this.id = id; }

// 没有逻辑的简单 POJO
public class PluginInfo {
    private String id;
    private String version;
    // ... 只有 getter/setter
}
```

## 反模式

❌ **避免**：
```java
// 1. 通用测试名称
@Test void test1() { }
@Test void testPlugin() { }

// 2. 测试实现细节
assertThat(plugin.internalState.flag).isTrue(); // 耦合到内部实现

// 3. 使用时间戳的脆弱断言
assertThat(message).isEqualTo("Error at 2024-01-26 10:30:15");

// 4. 多个不相关的断言
@Test void testEverything() {
    // 50 个不相关的断言
    assertThat(plugin.getId()).isNotNull();
    assertThat(manager.getCount()).isEqualTo(5);
    assertThat(config.isEnabled()).isTrue();
    // ... 混合多个关注点
}

// 5. 忽略异常
@Test void shouldFail() {
    try {
        loader.load(invalidPath);
        fail("Should have thrown exception");
    } catch (Exception e) {
        // 吞掉异常细节
    }
}
```

✅ **首选**：
```java
@Test
@DisplayName("Should reject plugin with missing dependencies")
void shouldRejectPluginWithMissingDependencies() {
    PluginDescriptor descriptor = PluginDescriptor.builder()
        .id("test-plugin")
        .dependencies(List.of("missing-dep"))
        .build();

    assertThatThrownBy(() -> manager.load(descriptor))
        .isInstanceOf(PluginException.class)
        .hasMessageContaining("Missing dependencies: missing-dep");
}
```

## 与覆盖率工具集成

### Maven 配置
```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.11</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### 测试生成后，建议：
```bash
# 运行测试并生成覆盖率
mvn clean test jacoco:report

# 查看覆盖率报告
open target/site/jacoco/index.html

# 检查覆盖率阈值
mvn verify # 如果低于配置的阈值则失败
```

## 快速参考

```java
// ===== 基本断言 =====
assertThat(value).isEqualTo(expected);
assertThat(value).isNotNull();
assertThat(value).isInstanceOf(String.class);
assertThat(number).isPositive().isGreaterThan(5);

// ===== 集合 =====
assertThat(list).hasSize(3);
assertThat(list).contains(item);
assertThat(list).containsExactly(item1, item2, item3);
assertThat(list).containsExactlyInAnyOrder(item2, item1, item3);
assertThat(list).doesNotContain(item);
assertThat(list).allMatch(predicate);

// ===== 字符串 =====
assertThat(str).isNotBlank();
assertThat(str).startsWith("prefix");
assertThat(str).endsWith("suffix");
assertThat(str).contains("substring");
assertThat(str).matches("regex\\d+");

// ===== 异常 =====
assertThatThrownBy(() -> code())
    .isInstanceOf(PluginException.class)
    .hasMessageContaining("error");

assertThatNoException().isThrownBy(() -> code());

// ===== 自定义描述 =====
assertThat(userId)
    .as("User ID should be positive")
    .isPositive();

// ===== 对象比较 =====
assertThat(actual)
    .usingRecursiveComparison()
    .ignoringFields("timestamp", "id")
    .isEqualTo(expected);
```

## 最佳实践总结

1. **所有断言使用 AssertJ**
2. **遵循 AAA 模式**（Arrange-Act-Assert）
3. **描述性名称** + @DisplayName
4. **每个测试一个概念**
5. **测试行为**，而非实现
6. **提取辅助方法**用于通用设置
7. **使用 @Nested** 进行逻辑分组
8. **参数化**相似测试
9. **软断言**用于多个检查
10. **覆盖率**关注业务逻辑，而非样板代码

## 参考资料

- [AssertJ Documentation](https://assertj.github.io/doc/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
