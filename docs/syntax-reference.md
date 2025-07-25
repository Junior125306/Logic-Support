# Logic语法参考大全

## 核心语法规则

### 表达式结束符

Logic中每个表达式必须以英文逗号(`,`)结束，这是Logic语法的基本规则，类似于Java或JavaScript中的分号(`;`)。

```javascript
// ✅ 正确的语法
name = "张三",
age = 30,
isAdmin = true,

// ❌ 错误：缺少逗号
name = "张三"
age = 30

// ⚠️ 特殊：return语句可以省略逗号
return result
```

**注意事项：**
- 忘记添加逗号是最常见的语法错误
- 块语句(如条件语句、循环等)内的最后一个表达式也需要逗号
- 只有return语句可以省略结束逗号

## 数据类型

### 基本数据类型

```javascript
// 数字类型 (number)
count = 10,
price = 15.99,
negative = -5,

// 字符串类型 (string)
name = "张三",
message = "欢迎使用Logic",
template = $用户{name}的年龄是{age}$,    // 支持变量插值

// 布尔类型 (boolean)
isActive = true,
isDeleted = false,

// 空值 (null)
emptyValue = null,
```

### 复合数据类型

```javascript
// 数组 (array)
numbers = [1, 2, 3, 4, 5],
names = ["张三", "李四", "王五"],
mixed = [1, "hello", true, null],

// 对象 (object)
user = {
    name: "张三",
    age: 30,
    email: "zhangsan@example.com",
    address: {
        city: "北京",
        district: "朝阳区"
    }
},
```

### 数据访问

```javascript
// 数组访问
firstNumber = numbers[0],
lastName = names[2],

// 对象属性访问
userName = user.name,
userAge = user.age,
userCity = user.address.city,

// 参数访问
userId = data.userId,
userName = data.name,
```

## 变量和赋值

### 变量声明和赋值

```javascript
// 基本赋值
name = "张三",
age = 25,

// 从对象中赋值
user = {name: "李四", age: 30},
userName = user.name,

// 从数组中赋值
scores = [85, 90, 88],
firstScore = scores[0],

// 从参数中赋值
inputName = data.name,
inputAge = data.age,
```

### 变量命名规则

```javascript
// ✅ 推荐的变量命名
userName = "张三",
userAge = 30,
isActive = true,
maxRetryCount = 3,

// ❌ 避免使用关键字
data = "some value",    // data是内置参数对象
log = "message",        // log是内置日志对象
```

## 条件表达式

### 基本条件语句

Logic使用三元表达式的变体语法：`条件 : (真分支), (假分支)`

```javascript
// 基本条件判断
age > 18 : (
    status = "成年人"
), (
    status = "未成年人"
),

// 空的假分支必须写null
isVip : (
    discount = 0.8
), null,

// 复杂条件
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

### 多条件判断

```javascript
// 多个独立条件
age = data.age,
age > 18 : (
    status = "成年",
    canVote = true
),
age > 60 : (
    status = "老年",
    hasDiscount = true
), (
    status = "青年",
    hasDiscount = false
),

// 复合条件
age >= 18 && score >= 60 : (
    result = "合格"
), (
    result = "不合格"
),
```

### 条件操作符

```javascript
// 比较操作符
a == b,     // 等于
a != b,     // 不等于
a > b,      // 大于
a < b,      // 小于
a >= b,     // 大于等于
a <= b,     // 小于等于

// 逻辑操作符
a && b,     // 逻辑与
a || b,     // 逻辑或
!a,         // 逻辑非

// 示例
(age >= 18 && score >= 60) || isVip : (
    access = "allowed"
), (
    access = "denied"
),
```

## 循环操作

### each循环

```javascript
// 遍历数组
items = [1, 2, 3, 4, 5],
items.each(
    log.info("当前元素: " + row),          // row: 当前元素
    log.info("索引: " + rowIndex)         // rowIndex: 索引
),

// 遍历对象
user = {name: "张三", age: 30, city: "北京"},
user.each(
    log.info("键: " + rowKey),            // rowKey: 对象的键
    log.info("值: " + row)                // row: 对象的值
),
```

### 范围循环

```javascript
// 数字范围循环
(1, 10).each(
    log.info("数字: " + row)
),

// 带控制的循环
(0, 100).each(
    row == 50 : (
        log.info("到达中点"),
        continue                          // 继续下一次循环
    ), null,
    
    row == 80 : (
        log.info("达到阈值，退出循环"),
        break                             // 跳出循环
    ), null,
    
    log.info("当前值: " + row)
),
```

### 循环控制

```javascript
// continue - 跳过当前迭代
numbers = [1, 2, 3, 4, 5],
numbers.each(
    row % 2 == 0 : (
        continue                          // 跳过偶数
    ), null,
    log.info("奇数: " + row)
),

// break - 终止循环
scores = [85, 90, 75, 95, 88],
scores.each(
    row < 80 : (
        log.info("发现低分: " + row),
        break                             // 发现低分就退出
    ), null,
    log.info("正常分数: " + row)
),
```

## 参数验证

### validate块语法

```javascript
// 基本参数验证
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
    }
},
```

### 验证规则

```javascript
validate {
    // 必填参数
    userId: {
        required: true,
        message: "用户ID是必需的"
    },
    
    // 可选参数with默认值
    pageSize: {
        default: 10
    },
    
    // 可选参数，无默认值
    sortBy: {
        required: false
    },
    
    // 复杂验证（需要在业务逻辑中实现）
    password: {
        required: true,
        message: "密码不能为空"
    }
},

// 自定义验证逻辑
commonTools.isNotEmpty(data.password) : null, (
    throw "密码不能为空"
),

data.password.length >= 6 : null, (
    throw "密码长度至少6位"
),
```

## 异常处理

### try-catch语法

```javascript
try {
    // 可能出错的代码
    user = entity.getById("t_user", data.userId),
    assert user != null,
    result = someRiskyOperation(user)
} catch (Exception e) {
    // 异常处理
    log.error("操作失败: " + e),
    throw {
        msg: "系统错误，请稍后重试",
        status: 500
    }
},
```

### 断言 (assert)

```javascript
// 基本断言
assert user != null,
assert age > 0,
assert commonTools.isNotEmpty(data.name),

// 在条件中使用断言
try {
    assert data.age >= 18
} catch (Exception e) {
    throw "年龄必须大于等于18岁"
},
```

### 抛出异常

```javascript
// 抛出字符串异常
throw "用户名不能为空",

// 抛出对象异常
throw {
    msg: "权限不足",
    status: 403,
    code: "ACCESS_DENIED"
},

// 条件抛出异常
data.role != "admin" : (
    throw {
        msg: "需要管理员权限",
        status: 403
    }
), null,
```

## 字符串操作

### 字符串定义

```javascript
// 双引号字符串
name = "张三",
message = "欢迎使用Logic语言",

// 模板字符串 (使用$符号)
greeting = $你好，{name}！今天是{dateTools.format(dateTools.now(), "yyyy-MM-dd")}$,

// 转义字符
quoted = "他说：\"Logic很简单\"",
price = "\$199.99",
```

### 字符串操作

```javascript
// 字符串连接
fullName = firstName + " " + lastName,

// 字符串比较
name1 == name2,
name1 != name2,

// 字符串长度（通过插件）
nameLength = name.length,         // 需要相应插件支持
```

## 注释规范

```javascript
// 单行注释 - 用于简短说明
name = "张三",        // 行末注释

/*
多行注释
用于详细说明和文档
*/

// TODO 注释 - 标记待完成的工作
// TODO: 添加参数验证

// BUG 注释 - 标记已知问题
// BUG: 这里可能存在并发问题

// FIXME 注释 - 标记需要修复的代码
// FIXME: 优化查询性能
```

## 函数式特性

### Lambda表达式

```javascript
// 定义Lambda函数
processor = (data) => {
    // 可以读取外部变量，但不能修改
    log.info("处理数据: " + data.name),
    return data.value * 2
},

// 调用Lambda函数
result = processor.apply({name: "test", value: 10}),

// 带外部变量的Lambda
multiplier = 3,
calculator = (input) => {
    // 可以访问外部的multiplier变量
    return input.value * multiplier
},
```

### 函数作为参数

```javascript
// 定义处理函数
dataProcessor = (item) => {
    return {
        id: item.id,
        processedAt: dateTools.now(),
        result: item.value * 2
    }
},

// 在循环中使用函数
items = [{id: 1, value: 10}, {id: 2, value: 20}],
processedItems = [],
items.each(
    processedItem = dataProcessor.apply(row),
    processedItems.push(processedItem)
),
```

## 内置对象和关键字

### 系统内置对象

```javascript
// data - 入参对象
userId = data.userId,
userName = data.name,

// log - 日志对象
log.info("信息日志"),
log.error("错误日志"),
log.warn("警告日志"),
log.debug("调试日志"),

// entity - 实体操作对象
user = entity.getById("t_user", userId),
result = entity.partialSave("t_user", userData),

// sql - SQL操作对象
users = sql.querySQL("queryName", "SELECT * FROM t_user"),
sql.execute("updateName", "UPDATE t_user SET name = 'new'"),
```

### 循环变量

```javascript
items.each(
    // row - 当前循环的元素
    log.info("当前元素: " + row),
    
    // rowIndex - 当前元素的索引（从0开始）
    log.info("索引: " + rowIndex),
    
    // rowKey - 对象循环时的键名
    log.info("键名: " + rowKey)
),
```

### 保留关键字

避免使用以下关键字作为变量名：
- `data` - 入参对象
- `log` - 日志对象
- `entity` - 实体操作对象
- `sql` - SQL操作对象
- `logic` - Logic调用对象
- `row`, `rowIndex`, `rowKey` - 循环变量
- `validate` - 参数验证块
- `try`, `catch` - 异常处理
- `throw` - 抛出异常
- `assert` - 断言
- `return` - 返回语句
- `break`, `continue` - 循环控制

## 语法最佳实践

### 1. 代码格式化

```javascript
// ✅ 推荐的格式
validate {
    name: {
        required: true,
        message: "姓名不能为空"
    }
},

userName = data.name,
userAge = data.age,

userAge >= 18 : (
    status = "成年人",
    canVote = true
), (
    status = "未成年人",
    canVote = false
),

return {
    success: true,
    data: {
        name: userName,
        status: status,
        canVote: canVote
    }
}
```

### 2. 错误避免

```javascript
// ❌ 常见错误
name = "张三"          // 缺少逗号
age > 18 : (           // 缺少else分支
    result = "成年"
),

// ✅ 正确写法
name = "张三",         // 添加逗号
age > 18 : (           // 添加else分支
    result = "成年"
), (
    result = "未成年"
),
```

### 3. 可读性提升

```javascript
// 使用有意义的变量名
currentUser = data.user,
isUserActive = currentUser.status == 1,
hasPermission = currentUser.role == "admin",

// 添加注释说明复杂逻辑
// 检查用户权限：必须是激活的管理员用户
isUserActive && hasPermission : (
    access = "granted"
), (
    access = "denied"
),
```

这个语法参考涵盖了Logic语言的所有核心语法特性。配合VS Code扩展的语法高亮和代码片段功能，可以大大提高Logic开发效率。