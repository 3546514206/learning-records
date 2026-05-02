---
name: mockito-junit
description: |
  Mockito + JUnit 5 单元测试编写指南。
  TRIGGER when: 编写单元测试、编写测试用例、mock 对象、stub 方法、验证交互、
  测试 service 层、测试异常场景、测试 void 方法、参数化测试、
  Spring Boot 测试集成、使用 @Mock @InjectMocks @Spy 注解。
  DO NOT TRIGGER when: 纯前端测试、集成测试（需要真实数据库）、端到端测试。
license: MIT
metadata:
  category: testing
  version: "1.0.0"
  sources:
    - Mockito Official Documentation (site.mockito.org)
    - JUnit 5 User Guide (junit.org/junit5)
    - Mockito GitHub Repository (github.com/mockito/mockito)
    - Spring Boot Testing Documentation
    - Kent Beck - Test Driven Development
    - Martin Fowler - Refactoring
---

# Mockito-JUnit 单元测试编写指南

## 1. 概述

本 skill 用于指导 AI Agent 编写高质量的单元测试代码，基于 Mockito 和 JUnit 5 框架。

## 2. 核心原则

- 测试应具有**独立性**、**可重复性**、**可读性**
- 每个测试方法应聚焦于单一行为验证
- 合理使用 mock 隔离被测单元的依赖
- 测试代码与生产代码同等重要

## 3. 常用注解

| 注解 | 用途 |
|------|------|
| `@Test` | 标记测试方法 (`org.junit.jupiter.api.Test`，**注意不是 `org.junit.Test` JUnit 4**) |
| `@BeforeEach` | 每个测试前执行 |
| `@AfterEach` | 每个测试后执行 |
| `@BeforeAll` | 所有测试前执行一次 |
| `@AfterAll` | 所有测试后执行一次 |
| `@Mock` | 创建 Mock 对象 |
| `@Spy` | 创建 Spy 对象 |
| `@InjectMocks` | 注入 Mock 对象到被测对象 |
| `@ExtendWith(MockitoExtension.class)` | 启用 Mockito 扩展 |
| `@DisplayName` | 自定义测试显示名称 |
| `@ParameterizedTest` | 参数化测试 |
| `@CsvSource` | CSV 格式参数源 |

## 4. Mockito 核心用法

### 4.1 Mock 创建与配置

```java
// 方式一：@Mock 注解 + @ExtendWith(MockitoExtension.class)
@Mock
private UserRepository userRepository;

@InjectMocks
private UserService userService;

// 方式二：手动创建
List<String> mockList = mock(List.class);

// 方式三：@Spy 注解 - 部分 mock，真实调用方法
@Spy
private List<String> spyList = new ArrayList<>();
```

### 4.2 Argument Matchers 参数匹配器

```java
// 精确匹配
when(mockList.get(0)).thenReturn("first");

// 使用匹配器
when(mockList.get(anyInt())).thenReturn("any");
when(mockList.set(anyInt(), anyString())).thenReturn("old");

// 特定匹配
when(mockList.get(eq(0))).thenReturn("first");

// nullable 和 isNull
when(stringProcessor.process(isNull())).thenReturn("null");
when(stringProcessor.process(nullable(String.class))).thenReturn("empty");

// 组合匹配
when(mockList.get(argThat(i -> i > 5))).thenReturn("greater than 5");

// anyList, anySet, anyMap
when(mapper.map(anyMap())).thenReturn("mapped");
```

### 4.3 行为配置

```java
// 设定返回值
when(mockList.get(0)).thenReturn("first");
when(mockList.get(1)).thenReturn("second");

// 连续调用返回不同值
when(mockList.get(0)).thenReturn("first")
                    .thenReturn("second")
                    .thenThrow(new RuntimeException("exhausted"));

// 无限调用返回相同值
when(mockList.size()).thenReturn(10);

// 模拟异常
when(mockList.get(0)).thenThrow(new RuntimeException("error"));

// 使用 thenAnswer 自定义逻辑
when(mockList.get(anyInt())).thenAnswer(invocation -> {
    int index = invocation.getArgument(0);
    return "element_" + index;
});

// do 系列（适合 void 方法或 spy）
doNothing().when(mockList).clear();
doThrow(new RuntimeException("error")).when(mockList).add(anyString());
doAnswer(invocation -> {
    System.out.println("Called with: " + invocation.getArgument(0));
    return null;
}).when(mockList).add(anyString());

// 延迟配置
when(mockList.size()).thenAnswer(invocation -> {
    Thread.sleep(100);
    return 10;
});
```

### 4.4 验证调用

```java
// 验证方法被调用
verify(mockList).add("one");

// 验证调用次数
verify(mockList, times(2)).add("one");
verify(mockList, never()).clear();
verify(mockList, atLeastOnce()).add("one");
verify(mockList, atMost(3)).add("one");

// 验证调用顺序
InOrder inOrder = inOrder(mockList);
inOrder.verify(mockList).add("first");
inOrder.verify(mockList).add("second");

// 验证无其他调用
verifyNoMoreInteractions(mockList);
verifyNoInteractions(mockList);

// 验证没有更多调用（宽松验证）
verify(mockList, atLeastOnce()).add("one");

// 使用 getArguments 获取调用参数
verify(mockList).add(argThat(s -> s.length() > 3));
```

### 4.5 Spy 用法

```java
// Spy 会真实调用对象方法，除非配置了 stub
List<String> spyList = spy(new ArrayList<>());

// 部分 mock - 推荐使用 doReturn/when 方式避免真实调用
doReturn("mocked").when(spyList).get(0);
when(spyList.size()).thenReturn(5);

// 不执行真实方法的正确方式
doNothing().when(spyList).clear();
doThrow(new RuntimeException()).when(spyList).add(null);

// 真实调用后再验证
spyList.add("one");
verify(spyList).add("one");
assertEquals(1, spyList.size());
```

## 5. 断言与假设

### 5.1 JUnit 5 断言

```java
import static org.junit.jupiter.api.Assertions.*;
import static org.junit.jupiter.api.Assumptions.assumeTrue;

// 基本断言
assertEquals(4, calculator.add(2, 2));
assertNotNull(user);
assertTrue(list.isEmpty());
assertFalse(list.isEmpty());

// 异常断言
assertThrows(ArithmeticException.class, () -> calculator.divide(1, 0));

// 惰性消息（避免字符串拼接开销）
assertEquals("expected", actual, () -> "message with " + expensiveOperation());

// 分组断言（一个失败不影响其他）
assertAll("user group",
    () -> assertNotNull(user.getName()),
    () -> assertEquals("John", user.getName()),
    () -> assertTrue(user.getAge() > 0)
);

// 异常详细断言
RuntimeException exception = assertThrows(RuntimeException.class, () -> {
    throw new RuntimeException("message");
});
assertEquals("message", exception.getMessage());

// 假设
assumeTrue("CI".equals(System.getenv("ENV")));
assumeFalse(user.isGuest());
```

## 6. 测试命名规范

```java
// 推荐：描述性命名，包含场景和期望
@Test
@DisplayName("should return user when findById with valid id")
void should_return_user_when_findById_by_valid_id() { }

@Test
@DisplayName("should throw UserNotFoundException when findById with non-existent id")
void should_throw_UserNotFoundException_when_findById_by_non_existent_id() { }

@Test
@DisplayName("should save user and return saved user")
void should_save_user_and_return_saved_user() { }

// 常用前缀
// should_... when_... - 描述行为
// given_... when_... then_... - BDD 风格
// test_... - 简单描述
```

## 7. 最佳实践

### 7.1 Given-When-Then 模式

```java
@Test
void should_calculate_total_price_with_discount() {
    // Given
    List<OrderItem> items = Arrays.asList(
        new OrderItem("p1", 2, 10.0),
        new OrderItem("p2", 1, 20.0)
    );
    UserService userService = new UserService(userRepository);
    when(userRepository.findDiscount()).thenReturn(0.1);

    // When
    double total = userService.calculateTotalWithDiscount(items);

    // Then
    assertEquals(36.0, total, 0.01);
    verify(userRepository).findDiscount();
}
```

### 7.2 测试粒度

```java
// 好：单一职责测试
@Test
void should_return_empty_list_when_no_users_exist() { }

@Test
void should_return_users_when_users_exist() { }

@Test
void should_throw_exception_when_save_user_with_null_name() { }

// 不好：一个测试验证太多
@Test
void testUserService() {
    // 太多场景混在一起
}
```

### 7.3 Mock 最佳实践

```java
// 好：只 mock 必要的依赖
@Mock
private UserRepository userRepository;

@InjectMocks
private UserService userService;

// 不好：mock 整个类而影响可读性
when(userRepository.anyMethod()).thenReturn(...);
```

### 7.4 常见反模式

- **过度 mock**：mock 一切，测试失去意义
- **断言不足**：只验证返回值，不验证交互
- **测试间耦合**：共享可变状态
- **magic numbers**：硬编码值无解释
- **忽视边界条件**：不测试空值、边界值

## 8. 常见测试场景模板

### 8.1 测试 Service 层

```java
@ExtendWith(MockitoExtension.class)
@DisplayName("UserService Tests")
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private EmailService emailService;

    @InjectMocks
    private UserService userService;

    @Test
    @DisplayName("should return user when findById with valid id")
    void should_return_user_when_findById_by_valid_id() {
        // Given
        User expectedUser = new User(1L, "John", "john@example.com");
        when(userRepository.findById(1L)).thenReturn(expectedUser);

        // When
        User result = userService.findById(1L);

        // Then
        assertNotNull(result);
        assertEquals("John", result.getName());
        assertEquals("john@example.com", result.getEmail());
        verify(userRepository).findById(1L);
        verifyNoInteractions(emailService);
    }

    @Test
    @DisplayName("should throw UserNotFoundException when findById with non-existent id")
    void should_throw_UserNotFoundException_when_findById_by_non_existent_id() {
        // Given
        when(userRepository.findById(999L)).thenReturn(null);

        // When & Then
        UserNotFoundException exception = assertThrows(
            UserNotFoundException.class,
            () -> userService.findById(999L)
        );
        assertEquals("User not found with id: 999", exception.getMessage());
        verify(userRepository).findById(999L);
    }

    @Test
    @DisplayName("should save user and send welcome email")
    void should_save_user_and_send_welcome_email() {
        // Given
        User user = new User(null, "John", "john@example.com");
        User savedUser = new User(1L, "John", "john@example.com");
        when(userRepository.save(user)).thenReturn(savedUser);
        doNothing().when(emailService).sendWelcomeEmail(any(User.class));

        // When
        User result = userService.register(user);

        // Then
        assertNotNull(result);
        assertEquals(1L, result.getId());
        verify(userRepository).save(user);
        verify(emailService).sendWelcomeEmail(savedUser);
    }
}
```

### 8.2 测试 void 方法

```java
@Test
@DisplayName("should clear all users")
void should_clear_all_users() {
    // Given
    doNothing().when(userRepository).deleteAll();

    // When
    userService.clearAll();

    // Then
    verify(userRepository, times(1)).deleteAll();
}

@Test
@DisplayName("should throw exception when deleteById with null id")
void should_throw_exception_when_deleteById_with_null_id() {
    // Given
    doThrow(new IllegalArgumentException("id cannot be null"))
        .when(userRepository).deleteById(null);

    // When & Then
    assertThrows(IllegalArgumentException.class,
        () -> userService.deleteById(null));
}
```

### 8.3 测试返回值的方法

```java
@Test
@DisplayName("should return all active users")
void should_return_all_active_users() {
    // Given
    List<User> activeUsers = Arrays.asList(
        new User(1L, "John", true),
        new User(2L, "Jane", true)
    );
    when(userRepository.findByActive(true)).thenReturn(activeUsers);

    // When
    List<User> result = userService.getActiveUsers();

    // Then
    assertEquals(2, result.size());
    assertTrue(result.stream().allMatch(User::isActive));
    verify(userRepository).findByActive(true);
}
```

### 8.4 测试异常场景

```java
@Test
@DisplayName("should throw exception when user name is blank")
void should_throw_exception_when_user_name_is_blank() {
    // Given
    User invalidUser = new User(1L, "   ", "email@test.com");
    when(userRepository.existsByEmail("email@test.com")).thenReturn(false);

    // When & Then
    ValidationException exception = assertThrows(ValidationException.class,
        () -> userService.createUser(invalidUser));
    assertTrue(exception.getMessage().contains("name"));
}

@Test
@DisplayName("should throw exception when email already exists")
void should_throw_exception_when_email_already_exists() {
    // Given
    User user = new User(null, "John", "existing@test.com");
    when(userRepository.existsByEmail("existing@test.com")).thenReturn(true);

    // When & Then
    assertThrows(EmailAlreadyExistsException.class,
        () -> userService.createUser(user));
    verify(userRepository).existsByEmail("existing@test.com");
}
```

### 8.5 测试带有回调/异步的方法

```java
@Test
@DisplayName("should call callback with result")
void should_call_callback_with_result() {
    // Given
    User user = new User(1L, "John", "john@example.com");
    ArgumentCaptor<UserCallback> callbackCaptor = ArgumentCaptor.forClass(UserCallback.class);
    when(userRepository.findByIdAsync(1L)).thenAnswer(invocation -> {
        UserCallback callback = invocation.getArgument(0);
        callback.onSuccess(user);
        return null;
    });

    // When
    userService.findByIdAsync(1L, result -> {
        assertEquals(user, result);
    });

    // Then
    verify(userRepository).findByIdAsync(1L);
}
```

### 8.6 参数化测试

```java
@ParameterizedTest
@CsvSource({
    "1, 2, 3",
    "0, 0, 0",
    "-1, 1, 0",
    "10, 20, 30"
})
@DisplayName("should add two numbers correctly")
void should_add_two_numbers_correctly(int a, int b, int expected) {
    assertEquals(expected, calculator.add(a, b));
}

@ParameterizedTest
@CsvSource({
    "john@example.com, true",
    "jane@example.com, true",
    "invalid, false"
})
@DisplayName("should validate email format")
void should_validate_email_format(String email, boolean expected) {
    assertEquals(expected, validator.isValidEmail(email));
}
```

## 9. Spring Boot 测试集成

```java
// 单元测试（使用 SpringExtension）
@ExtendWith(SpringExtension.class)
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    @DisplayName("should return user when GET /users/{id}")
    void should_return_user_when_get_user_by_id() throws Exception {
        // Given
        User user = new User(1L, "John", "john@example.com");
        when(userService.findById(1L)).thenReturn(user);

        // When & Then
        mockMvc.perform(get("/users/1")
                .accept(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("John"))
            .andExpect(jsonPath("$.email").value("john@example.com"));
    }
}

// 集成测试
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private UserRepository userRepository;

    @Test
    @DisplayName("should create and retrieve user")
    void should_create_and_retrieve_user() {
        // 先清理
        userRepository.deleteAll();

        // 创建用户
        User user = new User(null, "John", "john@example.com");
        User saved = userRepository.save(user);

        // 验证
        Optional<User> found = userRepository.findById(saved.getId());
        assertTrue(found.isPresent());
        assertEquals("John", found.get().getName());
    }
}
```

## 10. 测试辅助工具

### 10.1 懒加载断言

```java
@Test
void should_calculate_complex_result() {
    // 惰性消息只在断言失败时计算
    assertEquals(
        "expected_value",
        actual,
        () -> "Expected " + expensiveComputation() + " but got " + actual
    );
}
```

### 10.2 创建测试对象

```java
// 使用 builder 模式
@Test
void should_create_user_with_builder() {
    User user = User.builder()
        .name("John")
        .email("john@example.com")
        .age(25)
        .active(true)
        .build();

    when(userRepository.save(user)).thenReturn(user);
    User result = userService.createUser(user);
    assertNotNull(result);
}
```

## 11. 参考资源

- [Mockito 官方文档](https://site.mockito.org/)
- [JUnit 5 官方文档](https://junit.org/junit5/)
- [Mockito GitHub](https://github.com/mockito/mockito)
- [Spring Boot Testing](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-test.html)
