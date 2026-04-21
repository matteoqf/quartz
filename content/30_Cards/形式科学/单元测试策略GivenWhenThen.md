---
id: unit-test-given-when-then
title: 单元测试策略 Given-When-Then
type: procedure
tags:
  - 单元测试
  - 测试策略
  - 软件工程
  - 形式科学
created: 2026-04-21
modified: 2026-04-21
aliases:
  - GWT模式
  - 行为驱动测试
---

# 单元测试策略：Given-When-Then

## 概述

**Given-When-Then (GWT)** 是一种结构化的单元测试编写模式，源自行为驱动开发（BDD）方法论。它将测试用例划分为三个清晰的部分：

| 阶段 | 作用 | 典型内容 |
|------|------|----------|
| **Given** | 测试前置条件 | 输入数据、模拟对象、初始状态 |
| **When** | 执行被测操作 | 调用方法、触发行为 |
| **Then** | 验证预期结果 | 断言、返回值、状态变化 |

## 核心原则

### 1. 单测单一行为
每个测试方法只验证一个明确的behavior，避免过度耦合。

### 2. 清晰的三段式结构
```
Given: 设定测试环境
When:  执行操作
Then:  验证结果
```

### 3. 可读性与可维护性
测试代码应像自然语言描述的行为规格说明。

---

## 实施步骤

### 步骤1：明确测试目标

在编写测试前，确定：
- 被测试的方法或函数
- 期望的行为是什么
- 边界条件和异常场景

### 步骤2：构建 Given 阶段

准备测试所需的一切前置条件：

```python
# Python示例
def test_add_item_to_empty_cart():
    # Given: 购物车为空
    cart = ShoppingCart()
    product = Product(id=1, name="Book", price=29.99)
```

### 步骤3：执行 When 阶段

执行实际的操作：

```python
    # When: 添加商品到购物车
    cart.add_item(product, quantity=2)
```

### 步骤4：验证 Then 阶段

使用断言验证预期结果：

```python
    # Then: 购物车包含该商品且数量正确
    assert cart.item_count == 1
    assert cart.get_item(product.id).quantity == 2
    assert cart.total_price == 59.98
```

---

## 完整示例

### Java/JUnit 示例

```java
@Test
void shouldCalculateTotalPriceWithDiscount() {
    // Given: 一个购物车，包含两件商品
    ShoppingCart cart = new ShoppingCart();
    Product book = new Product("Book", 100.0);
    Product pen = new Product("Pen", 20.0);
    cart.addItem(book);
    cart.addItem(pen);
    
    // And: 有一张10%折扣券
    Discount coupon = new PercentageDiscount(0.1);
    
    // When: 应用折扣并计算总价
    double total = cart.calculateTotal(coupon);
    
    // Then: 总价应为108 (110 * 0.9)
    assertEquals(108.0, total, 0.01);
}
```

### Python/pytest 示例

```python
def test_user_registration_with_valid_email():
    # Given: 用户提供的有效注册信息
    valid_user_data = {
        "username": "testuser",
        "email": "test@example.com",
        "password": "SecurePass123"
    }
    user_repository = MockUserRepository()
    email_service = MockEmailService()
    registration_service = RegistrationService(user_repository, email_service)
    
    # When: 用户提交注册表单
    result = registration_service.register(valid_user_data)
    
    # Then: 注册成功，用户被创建，验证邮件已发送
    assert result.success is True
    assert result.user.username == "testuser"
    assert result.user.is_verified is False
    email_service.verify_sent_to("test@example.com")
```

---

## 进阶模式

### 参数化测试 (Parameterized Tests)

减少重复代码，对多组数据执行相同验证逻辑：

```java
@ParameterizedTest
@CsvSource({
    "1, 2, 3",
    "0, 0, 0",
    "-1, 1, 0",
    "100, 200, 300"
})
void shouldAddTwoNumbers(int a, int b, int expected) {
    // When
    int result = calculator.add(a, b);
    // Then
    assertEquals(expected, result);
}
```

### 测试嵌套 (Nested Tests)

使用嵌套类组织相关测试场景：

```java
@DisplayName("用户登录测试")
class UserLoginTest {
    
    @Nested
    @DisplayName("成功场景")
    class SuccessScenarios {
        // Given/When/Then tests
    }
    
    @Nested
    @DisplayName("失败场景")
    class FailureScenarios {
        // Given/When/Then tests
    }
}
```

---

## 常见反模式

| 反模式 | 问题 | 改进方式 |
|--------|------|----------|
| **Given-When-Then混用** | 结构不清 | 明确分隔每个阶段 |
| **过度模拟** | 测试与实现耦合 | 仅模拟外部依赖 |
| **断言过多** | 测试不单一 | 拆分为多个测试 |
| **魔法数字** | 可读性差 | 使用命名常量 |

---

## 工具支持

### Java生态
- **JUnit 5**: 原生支持嵌套测试、参数化测试
- **AssertJ**: 流畅断言API
- **Mockito**: 模拟框架

### Python生态
- **pytest**: 参数化、fixtures、标记
- **unittest**: 标准库支持
- **pytest-mock**: 模拟支持

### JavaScript/TypeScript生态
- **Jest**: 内置模拟、快照测试
- **Mocha/Chai**: BDD风格断言
- **Vitest**: 现代化替代方案

---

## 总结

**Given-When-Then** 模式通过清晰的结构化划分：
- 提升测试代码可读性
- 便于测试意图的快速理解
- 支持团队协作与知识传递
- 降低测试维护成本

采用此策略时，应坚持**单一行为验证**原则，合理使用参数化和嵌套测试，并注意避免过度模拟导致的脆弱性。

---

## 相关资源

- [[单元测试原则]]
- [[测试金字塔]]
- [[Mock对象使用规范]]
- [[行为驱动开发BDD]]
