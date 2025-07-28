# Logic语法完整参考

## 核心语法规则

### 表达式结束符

- 每个表达式必须以逗号(`,`)结束
- return语句例外，可省略逗号

### 条件表达式

- 格式：`条件 : (真分支), (假分支)`
- 必须提供true和false两个分支
- 空分支写`null`

### 参数访问

- 使用`data.fieldName`访问入参

## 数据类型

### 基本类型

```logic
// 数字
count = 10,
price = 15.99,

// 字符串
name = "张三",
template = $用户{name}的年龄是{age}$,

// 布尔
isActive = true,

// 空值
value = null,
```

### 复合类型

```logic
// 数组
numbers = [1, 2, 3],
mixed = [1, "hello", true],

// 对象
user = {
    name: "张三",
    age: 30,
    address: {
        city: "北京"
    }
},
```

### 数据访问

```logic
// 数组访问
first = numbers[0],

// 对象属性访问
userName = user.name,
city = user.address.city,

// 参数访问
userId = data.userId,
```

## 条件逻辑详解

### 基本条件

```logic
age > 18 : (
    status = "成年人"
), (
    status = "未成年人"
),

// 空分支
isVip : (
    discount = 0.8
), null,
```

### 多条件判断

```logic
score >= 90 : (
    level = "优秀"
), score >= 80 : (
    level = "良好"
), score >= 60 : (
    level = "及格"
), (
    level = "不及格"
),
```

### 条件操作符

```logic
// 比较：==, !=, >, <, >=, <=
// 逻辑：&&, ||, !

(age >= 18 && score >= 60) || isVip : (
    access = "allowed"
), (
    access = "denied"
),
```

## 循环操作

### 数组循环

```logic
items = [1, 2, 3],
items.each(
    log.info("元素: {row}"),        // row: 当前元素
    log.info("索引: {rowIndex}")     // rowIndex: 索引
),
```

### 对象循环

```logic
user = {name: "张三", age: 30},
user.each(
    log.info("键: {rowKey}"),        // rowKey: 键名
    log.info("值: {row}")            // row: 值
),
```

### 范围循环

```logic
(1, 10).each(
    log.info("数字: {row}")
),
```

### 循环控制

```logic
items.each(
    row == 5 : (
        continue    // 跳过当前迭代
    ), null,
    
    row == 8 : (
        break      // 终止循环
    ), null,
    
    log.info("处理: {row}")
),
```

## 参数验证

### validate块

```logic
validate {
    name: {
        required: true,
        message: "姓名不能为空"
    },
    age: {
        required: true,
        message: "年龄不能为空"
    },
    email: {
        default: "no-email@example.com"
    },
    sortBy: {
        required: false
    }
},
```

## 异常处理

### try-catch

```logic
try {
    user = entity.getById("t_user", data.userId),
    assert user != null,
    result = operation(user)
} catch (Exception e) {
    log.error("操作失败: " + e),
    throw {
        msg: "系统错误",
        status: 500
    }
},
```

### 断言

```logic
assert user != null,
assert age > 0,
assert data.name != null && data.name != "",
```

### 抛出异常

```logic
// 字符串异常
throw "用户名不能为空",

// 对象异常
throw {
    msg: "权限不足",
    status: 403,
    code: "ACCESS_DENIED"
},
```

## Lambda表达式

### 定义和调用

```logic
// 定义Lambda
processor = (data) => {
    log.info("处理: {data.name}"),
    return data.value * 2
},

// 调用Lambda
result = processor.apply({name: "test", value: 10}),

// 带外部变量的Lambda
multiplier = 3,
calculator = (input) => {
    return input.value * multiplier  // 可访问外部变量
},
```

## 字符串操作

### 字符串定义

```logic
// 基本字符串
name = "张三",

// 模板字符串
greeting = $你好，{name}！今天是{dateTools.getNow()}$,

// 转义字符
quoted = "他说：\"Logic很简单\"",
```

### 字符串操作详解

```logic
// 连接
fullName = "{firstName} {lastName}",

// 比较
name1 == name2,
name1 != name2,
```

## 内置对象

### 系统对象

```logic
// data - 入参对象
userId = data.userId,

// log - 日志对象
log.info("信息"),
log.error("错误"),
log.warn("警告"),
log.debug("调试"),

// entity - 实体操作
user = entity.getById("t_user", userId),
result = entity.partialSave("t_user", userData),
entity.deleteById("t_user", userId),

// sql - SQL操作
users = sql.querySQL("queryName", "SELECT * FROM t_user WHERE id = {data.id}"),
sql.execute("updateName", "UPDATE t_user SET name = {data.name}"),

// logic - Logic调用
result = logic.run("otherLogic", parameters),
logic.runAsync("asyncLogic", parameters),
logic.remoteRun("service", "remoteLogic", parameters),
```

### 循环变量

```logic
items.each(
    // row - 当前元素
    // rowIndex - 索引（从0开始）
    // rowKey - 对象键名（对象循环）
),
```

## 标准代码模式

### 基本Logic结构

```logic
// 1. 参数验证
validate {
    userId: {
        required: true,
        message: "用户ID不能为空"
    }
},

// 2. 参数获取
userId = data.userId,

// 3. 业务逻辑
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

### 错误处理模式

```logic
// 参数检查
data.name != null && data.name != "" : null, (
    throw "姓名不能为空"
),

// 数据库操作错误处理
try {
    result = entity.partialSave("t_user", saveData)
} catch (Exception e) {
    log.error("保存失败: {e}"),
    throw "系统错误，请稍后重试"
}
```

## 保留关键字

避免使用以下关键字作为变量名：

- `data`, `log`, `entity`, `sql`, `logic` - 内置对象
- `row`, `rowIndex`, `rowKey` - 循环变量
- `validate` - 参数验证块
- `try`, `catch`, `throw`, `assert` - 异常处理
- `return` - 返回语句
- `break`, `continue` - 循环控制
