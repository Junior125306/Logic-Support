# Logic开发最佳实践

## 标准代码结构

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
currentTime = dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss"),

// 3. 业务逻辑验证
userId != null && userId != "" : null, (
    throw "用户ID不能为空"
),

// 4. 核心业务处理
user = entity.getById("t_user", userId),
user != null : null, (
    throw "用户不存在"
),

// 5. 返回结果
return {
    success: true,
    data: user
}
```

## 错误处理规范

### 参数验证

```logic
// 使用validate块
validate {
    name: {required: true, message: "姓名必填"},
    email: {required: true, message: "邮箱必填"}
},

// 业务逻辑验证
data.name != null && data.name != "" : null, (throw "姓名不能为空"),
data.age > 0 : null, (throw "年龄必须大于0"),
```

### 异常处理

```logic
try {
    user = entity.getById("t_user", data.userId),
    result = entity.partialSave("t_user", updateData)
} catch (Exception e) {
    log.error("操作失败: " + e),
    throw "系统错误，请稍后重试"
}
```

## 性能优化

### 数据库优化

```logic
// ✅ 使用参数化查询
users = sql.querySQL("query", "SELECT * FROM t_user WHERE id = {data.id}"),

// ✅ 避免循环中的数据库操作
usersWithProfiles = sql.querySQL("joinQuery", "
    SELECT u.*, p.avatar FROM t_user u
    LEFT JOIN t_profile p ON u.id = p.user_id
"),
```

### 条件判断优化

```logic
// ✅ 简单条件在前
(data.name != null && data.name != "") && data.age > 18 : (
    // 处理逻辑
), null,
```

## 日志记录

### 日志级别使用

```logic
// 关键业务操作
log.info("用户登录: " + user.name),

// 错误信息
log.error("数据保存失败: " + e),

// 调试信息
log.debug("处理参数: " + jsonTools.toJson(data)),

// 警告信息
log.warn("用户权限不足: " + user.id),
```

## 代码规范

### 命名规范

```logic
// ✅ 推荐命名
userName = data.name,
userAge = data.age,
isAdmin = user.role == "admin",
maxRetryCount = 3,

// ❌ 避免的命名
data = "some value",    // data是保留字
log = "message",        // log是保留字
```

### 代码组织

```logic
// 按功能分组
// 参数处理
userId = data.userId,
userName = data.name,

// 数据查询
user = entity.getById("t_user", userId),
profile = entity.getById("t_profile", user.profileId),

// 业务逻辑
isValidUser = user != null && user.status == 1,
canEdit = isValidUser && (user.createdBy == currentUserId || isAdmin),

// 返回结果
return {success: true, data: processedData}
```

## 安全实践

### SQL注入防护

```logic
// ✅ 使用参数化查询
users = sql.querySQL("safe", "SELECT * FROM t_user WHERE name = {data.name}"),

// ❌ 避免字符串拼接
// badQuery = "SELECT * FROM t_user WHERE name = '" + data.name + "'",
```

### 数据验证

```logic
// 输入验证
data.userId != null && data.userId != "" : null, (throw "用户ID不能为空"),
data.userId.length <= 32 : null, (throw "用户ID长度超限"),

// 权限检查
currentUser.role == "admin" : null, (throw "权限不足"),
```

## 常见模式

### 数据处理模式

```logic
// 标准CRUD模式
validate {id: {required: true}},
id = data.id,
record = entity.getById("t_table", id),
record != null : null, (throw "记录不存在"),
result = entity.partialSave("t_table", updateData),
return {success: true, data: result}
```

### 批量处理模式

```logic
items = data.items,
results = [],
errors = [],

items.each(
    try {
        result = entity.partialSave("t_table", row),
        results.push(result)
    } catch (Exception e) {
        errors.push({item: row, error: e})
    }
),

return {
    success: errors.length == 0,
    results: results,
    errors: errors
}
```

### 条件处理模式

```logic
// 多条件判断
status = data.status,
role = data.role,

status == "active" && role == "admin" : (
    // 管理员处理
    result = processAsAdmin()
), status == "active" : (
    // 普通用户处理
    result = processAsUser()
), (
    // 异常处理
    throw "用户状态异常"
)
```
