---
name: kotlin-code-review
description: 基于项目 kotlin-standard 规范对 Kotlin 代码进行全面审查，涵盖变量、类型、类与对象、数据类、集合、函数、泛型、协程、元编程等方面，结合代码上下文提供详细的改进建议。触发词：kotlin代码审查、审查kotlin代码、检查kotlin代码、kotlin code review、review kotlin code
---

# Kotlin 代码审查

## 概述

本 skill 提供基于项目 kotlin-standard 规范的 Kotlin 代码审查能力。审查涵盖以下维度：

1. **变量使用** - val/var 选择、作用域、延迟初始化、lateinit 使用
2. **类型安全** - 空安全、平台类型、类型别名、类型推断
3. **类与对象** - 对象创建模式、密封类、属性设计
4. **数据类** - 不可变性、copy 使用、解构
5. **集合操作** - 不可变集合、函数式扩展、Sequence 使用
6. **函数设计** - 契约、具名参数、操作符、高阶函数、SAM 转换
7. **泛型** - 类型擦除、装箱行为、类型实化、型变
8. **协程** - 纯函数、作用域、取消处理、并发安全、异常处理
9. **元编程** - 反射成本、代码生成、KSP 优先
10. **最佳实践** - 性能优化、代码风格、惯用写法

## 审查工作流程

### 1. 理解代码上下文
- 分析文件的职责和在项目中的位置
- 识别代码的业务逻辑和技术架构
- 了解相关的依赖和调用关系

### 2. 执行规范检查
按照以下优先级进行审查：
- **P0 严重问题**：可能导致 crash、内存泄漏、数据不一致的问题
- **P1 重要问题**：违反核心规范、影响性能或可维护性的问题
- **P2 建议**：代码风格、可读性改进建议

### 3. 提供改进方案
- 指出具体的问题代码位置（行号）
- 说明违反了哪条规范
- 提供正例代码示例
- 解释为什么这样改进

### 4. 生成审查报告
使用结构化格式输出审查结果（见输出格式部分）

## 审查维度详解

### 1. 变量使用规范

#### 应当检查的要点：
- ✅ 优先使用 `val` 而不是 `var`
- ✅ 变量作用域最小化
- ✅ 高开销初始化使用 `by lazy`
- ✅ 谨慎使用 `lateinit`，禁用 `isInitialized`

#### 常见问题：
```kotlin
// ❌ 反例
var count = 0  // 应该用 val
class Manager {
    var result = 0  // 作用域过大
}

// ✅ 正例
val count = 0
class Manager {
    fun calculate(): Int {
        val result = 0  // 局部变量
        return result
    }
}
```

### 2. 类型安全规范

#### 应当检查的要点：
- ✅ 使用 `T?`、`?.`、`?:` 处理空值
- ✅ 避免使用 `!!` 非空断言
- ✅ 消除平台类型风险
- ✅ 使用 `typealias` 简化复杂类型
- ✅ 不明确时显式声明类型
- ✅ 公共 API 避免暴露推断类型

#### 常见问题：
```kotlin
// ❌ 反例
val name = person!!.address!!.city  // 使用 !!
fun getData(): Any  // 返回 Any

// ✅ 正例
val name = person?.address?.city ?: "Unknown"
fun getData(): List<String>  // 明确类型
```

### 3. 类与对象规范

#### 应当检查的要点：
- ✅ 优先使用主构造函数和默认参数
- ✅ 复杂场景使用工厂函数
- ✅ 复杂状态使用密封类替代枚举
- ✅ 属性应代表状态而非行为

#### 常见问题：
```kotlin
// ❌ 反例
class User {
    val temperature: Double
        get() = fetchFromAPI()  // 属性不应有副作用
}

// ✅ 正例
class User {
    fun fetchTemperature(): Double = fetchFromAPI()
}
```

### 4. 数据类规范

#### 应当检查的要点：
- ✅ 优先考虑普通类，确需纯数据存储才用数据类
- ✅ 避免可变属性
- ✅ 避免复杂类型属性（函数类型、可变集合）
- ✅ 谨慎使用 `copy()`（浅复制）
- ✅ 正确使用解构（限制在小作用域、3个属性以内）

#### 常见问题：
```kotlin
// ❌ 反例
data class User(
    var name: String,  // 可变属性
    val tags: MutableList<String>,  // 可变集合
    val onLogin: () -> Unit  // 函数类型
)

// ✅ 正例
data class User(
    val name: String,
    val tags: List<String>
)
```

### 5. 集合操作规范

#### 应当检查的要点：
- ✅ 优先使用不可变集合
- ✅ 使用函数式扩展（map、filter、reduce）
- ✅ 避免在 `forEach` 中使用 `return`
- ✅ 大数据集或链式操作考虑使用 `Sequence`
- ✅ 不在外部 API 暴露可变集合

#### 常见问题：
```kotlin
// ❌ 反例
fun getUsers(): MutableList<User>  // 暴露可变集合
numbers.forEach { if (it > 5) return }  // forEach 中 return

// ✅ 正例
fun getUsers(): List<User>
numbers.firstOrNull { it > 5 }
```

### 6. 函数设计规范

#### 应当检查的要点：
- ✅ 使用 `require` 和 `check` 强制执行契约
- ✅ 多参数、同类型参数、布尔参数使用具名参数
- ✅ 操作符行为与名称一致
- ✅ 正确理解高阶函数内联（inline、noinline、crossinline）
- ✅ 正确处理 SAM 转换（注册/反注册一致性）

#### 常见问题：
```kotlin
// ❌ 反例
fun setAge(age: Int) { this.age = age }  // 缺少前置条件检查
resizeImage(1024, 768, 1.5)  // 参数含义不明

// ✅ 正例
fun setAge(age: Int) {
    require(age > 0) { "Age must be positive" }
    this.age = age
}
resizeImage(width = 1024, height = 768, scaleFactor = 1.5)
```

### 7. 泛型规范

#### 应当检查的要点：
- ✅ 理解泛型类型擦除
- ✅ 注意基本类型装箱
- ✅ 内联函数使用 `reified` 避免类型擦除
- ✅ 使用型变（out/in）确保容器类型安全

#### 常见问题：
```kotlin
// ❌ 反例
fun <T> isInstanceOf(value: Any): Boolean {
    return value is T  // 编译错误
}

// ✅ 正例
inline fun <reified T> isInstanceOf(value: Any): Boolean {
    return value is T
}
```

### 8. 协程规范

#### 应当检查的要点：
- ✅ 尽量编写纯函数，避免共享状态
- ✅ 避免使用 `GlobalScope`
- ✅ 正确处理协作式取消（不捕获 CancellationException）
- ✅ 避免使用线程锁，使用 `Mutex`
- ✅ 正确处理异常（通过返回值而非抛出）

#### 常见问题：
```kotlin
// ❌ 反例
GlobalScope.launch { }  // 生命周期不受控
synchronized(lock) { delay(100) }  // 协程中使用线程锁

// ✅ 正例
viewModelScope.launch { }
mutex.withLock { }
```

### 9. 元编程规范

#### 应当检查的要点：
- ✅ 注意类引用实例不唯一
- ✅ 理解反射成本，避免性能敏感场景使用
- ✅ 优先使用代码生成而非反射
- ✅ 优先使用 KSP 而非 KAPT

#### 常见问题：
```kotlin
// ❌ 反例
synchronized(Worker::class) { }  // 类引用可能不唯一
val kClass = Class.forName(name).kotlin  // 启动时使用 Kotlin 反射

// ✅ 正例
synchronized(this) { }
val jClass = Class.forName(name)  // 使用 Java 反射
```

### 10. 最佳实践

#### 应当检查的要点：
- ✅ 优先使用不可变数据
- ✅ 使用默认参数而非重载
- ✅ 使用类型别名简化复杂类型
- ✅ 优先使用条件表达式
- ✅ 优先使用高阶函数而非循环
- ✅ 避免滥用 `lateinit`
- ✅ 使用 `data` 关键字定义数据类
- ✅ 使用安全转换 `as?`
- ✅ 优先使用字符串模板
- ✅ 返回 `Flow` 不声明为 `suspend`
- ✅ 减少带标签表达式
- ✅ 冗余空检查用 `run {}`
- ✅ 多次重复字符串提前声明
- ✅ 优先使用 Kotlin 定义的数组
- ✅ 减少 `ForEachOnRange`
- ✅ 减少 `SpreadOperator`
- ✅ 使用下标操作替代 get/set
- ✅ 不显式调用 `gc`
- ✅ 合理使用 `inline` 优化 lambda
- ✅ 集合多操作符考虑 `asSequence`

## 审查输出格式

```markdown
# Kotlin 代码审查报告

**文件**：`<文件路径>`
**审查时间**：<时间>
**总体评估**：优秀/良好/需改进

---

## 严重问题（P0）🔴

### 🔴 [严重] <问题标题>
**位置**：第 X 行
**问题**：<问题描述>
**违反规范**：<具体规范条目>
**风险**：<可能导致的后果>
**建议**：
\`\`\`kotlin
// 改进后的代码
\`\`\`

---

## 重要问题（P1）🟡

### 🟡 [重要] <问题标题>
**位置**：第 X 行
**问题**：<问题描述>
**违反规范**：<具体规范条目>
**建议**：
\`\`\`kotlin
// 改进后的代码
\`\`\`

---

## 建议（P2）🔵

### 🔵 [建议] <问题标题>
**位置**：第 X 行
**问题**：<问题描述>
**建议**：<改进建议>

---

## 正面反馈 ✅

### ✅ [好的实践] <标题>
**位置**：第 X 行
**原因**：<为什么这是好的实践>

---

## 总结

### 优点
- <列出代码的优点>

### 改进领域
- <列出需要改进的方面>

### 建议
<总体改进建议和优先级>
```

## 使用示例

### 示例 1：审查单个文件
```
用户：kotlin代码审查 UserManager.kt

助手：[执行完整审查流程，生成结构化报告]
```

### 示例 2：审查特定方面
```
用户：检查这个文件的协程使用是否符合规范

助手：[重点审查协程相关代码]
```

### 示例 3：审查代码片段
```
用户：帮我review这段代码：
\`\`\`kotlin
fun getData() = GlobalScope.launch {
    val result = api.fetch()!!
    return result
}
\`\`\`

助手：[针对代码片段进行审查]
```

## 参考资源

本 skill 基于以下规范文档：
- `references/variable.md` - 变量使用规范
- `references/type.md` - 类型安全规范
- `references/class_and_object.md` - 类与对象规范
- `references/data_class.md` - 数据类规范
- `references/collections.md` - 集合操作规范
- `references/function.md` - 函数设计规范
- `references/generics.md` - 泛型规范
- `references/coroutines.md` - 协程规范
- `references/metaprogramming.md` - 元编程规范
- `references/kotlin_best_practices.md` - Kotlin 最佳实践

## 注意事项

1. **上下文优先**：始终结合代码的实际业务场景和项目架构进行审查
2. **平衡取舍**：某些规范在特定场景下可能需要权衡（如性能 vs 可读性）
3. **渐进改进**：对于大型遗留代码，建议分优先级逐步改进
4. **团队共识**：审查建议应与团队编码规范保持一致
5. **工具辅助**：建议配合 detekt、ktlint 等静态分析工具使用
