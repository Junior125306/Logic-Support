# Logic语言快速入门指南

## 什么是Logic语言？

Logic是一种基于Java的解释性脚本语言，专为企业级业务快速开发和动态扩展而设计。它在AF SystemV4架构中扮演着核心业务逻辑处理的角色。

### 核心特点

- **与Java深度集成** - 支持无缝互操作，可以直接调用Java类和方法
- **轻量化语法** - 融合了JavaScript和Python的语法风格，学习成本低
- **解释执行机制** - 支持动态加载和即时执行，适合快速迭代
- **模块化与扩展性** - 适合业务逻辑的快速扩展与变更
- **企业级特性** - 内置数据库操作、插件系统、异常处理等企业级功能

## 环境准备

### 开发工具支持

1. **VS Code扩展**：安装"Logic Support"扩展获得完整的IDE支持
   - 语法高亮
   - 代码片段
   - 括号匹配
   - 自动补全

2. **Context7支持**：通过Context7在任何AI编辑器中获得Logic语法帮助

### 基本要求

- Java运行环境
- AF SystemV4框架支持
- 文本编辑器（推荐VS Code）

## 第一个Logic脚本

让我们从一个简单的例子开始：

```logic
// hello-world.logic
// 这是你的第一个Logic脚本

// 参数验证
validate {
    name: {
        required: true,
        message: "姓名不能为空"
    }
},

// 获取入参
userName = data.name,

// 简单的条件判断
userName == "世界" : (
    greeting = "Hello, World!"
), (
    greeting = "你好, {userName}!"
),

// 记录日志
log.info("生成问候语: {greeting}"),

// 返回结果
return {
    success: true,
    message: greeting,
    timestamp: dateTools.getNow2()
}
```

### 运行说明

1. 将代码保存为 `.logic` 文件到 `src/main/resources/{模块}/logics/` 目录
2. 在对应的 `logic.xml` 中注册：`<logic alias="别名" path="文件名.logic" />`
3. 在 `module.xml` 中注册模块：`<module name="模块名"/>`
4. 构建部署后通过REST API调用：`POST /logic/别名`

## 核心语法规则

### 1. 表达式结束符

**重要**：每个表达式必须以逗号(`,`)结束，除了return语句

```logic
name = "张三",        // ✅ 正确
age = 30,            // ✅ 正确
isAdmin = true,      // ✅ 正确

return result        // ✅ return语句不需要逗号
```

### 2. 条件表达式

条件表达式必须有true和false两个分支，即使false分支为空也要写`null`

```logic
// ✅ 正确的条件表达式
age > 18 : (
    status = "成年人"
), (
    status = "未成年人"
),

// ✅ 空的false分支也要写null
isVip : (
    discount = 0.8
), null,
```

### 3. 参数访问

使用 `data.fieldName` 访问传入的参数

```logic
// 获取传入的参数
userId = data.userId,
userName = data.userName,
userAge = data.age,
```

### 4. 数据类型

```logic
// 数字
count = 10,
price = 15.99,

// 字符串
name = "张三",
message = "欢迎使用Logic",

// 布尔值
isActive = true,
isDeleted = false,

// 数组
numbers = [1, 2, 3, 4, 5],
names = ["张三", "李四", "王五"],

// 对象
user = {
    name: "张三",
    age: 30,
    email: "zhangsan@example.com"
},
```

## 常用功能示例

### 数据库操作

```logic
// 查询用户信息
user = entity.getById("t_user", data.userId),

// 保存或更新数据
saveData = {
    f_name: data.name,
    f_age: data.age,
    f_status: 1
},
result = entity.partialSave("t_user", saveData),

// 执行SQL查询
users = sql.querySQL("getUserList", "
    SELECT * FROM t_user 
    WHERE f_status = 1 
    AND f_age > {data.minAge}
"),
```

### 循环操作

```logic
// 遍历数组
items = [1, 2, 3, 4, 5],
items.each(
    log.info("当前元素: {row}"),
    log.info("索引: {rowIndex}")
),

// 范围循环
(1, 10).each(
    log.info("数字: {row}")
),
```

### 异常处理

```logic
try {
    // 可能出错的代码
    result = entity.getById("t_user", data.userId),
    assert result != null
} catch (Exception e) {
    log.error("查询用户失败: {e}"),
    throw {
        msg: "用户不存在",
        status: 404
    }
}
```

## 开发最佳实践

### 1. 标准Logic结构

```logic
// 1. 参数验证
validate {
    userId: {
        required: true,
        message: "用户ID不能为空"
    }
},

// 2. 获取参数
userId = data.userId,

// 3. 业务逻辑处理
user = entity.getById("t_user", userId),
user != null : null, (
    throw "用户不存在"
),

// 4. 返回结果
return {
    success: true,
    data: user
}
```

### 2. 错误处理

```logic
// 参数验证
data.name != null && data.name != "" : null, (
    throw "姓名不能为空"
),

// 数据库操作错误处理
try {
    result = entity.partialSave("t_user", saveData)
} catch (Exception e) {
    log.error("保存用户失败: {e}"),
    throw "系统错误，请稍后重试"
}
```

### 3. 日志记录

```logic
// 记录关键步骤
log.info("开始处理用户注册, userId: {data.userId}"),

// 记录业务逻辑
log.info("用户验证通过, 开始保存数据"),

// 记录结果
log.info("用户注册成功, 生成ID: {result.id}"),
```

## 下一步学习

- 📖 [语法参考大全](syntax-reference.md) - 详细的语法规则和特性
- 🗃️ [数据库操作指南](database-operations.md) - 完整的数据库操作方法
- 🔌 [插件系统详解](plugin-system.md) - 使用和开发插件
- 🚀 [高级特性](advanced-features.md) - Lambda表达式、异步操作等
- 💡 [最佳实践](best-practices.md) - 企业级开发规范
- 🔧 [故障排除](troubleshooting.md) - 常见问题和解决方案

## 获取帮助

- **VS Code用户**：安装Logic Support扩展，享受完整的IDE支持
- **AI编辑器用户**：通过Context7获得实时的Logic语法帮助
- **示例代码**：查看 `examples/` 目录中的完整示例
- **代码片段**：在VS Code中输入关键词快速插入代码模板
