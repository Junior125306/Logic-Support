# Logic数据库操作指南

数据库操作通过`entity`和`sql`两个内置对象实现。

## entity对象 - 实体操作

`entity`对象用于单表CRUD操作。

### 查询操作

#### getById - 根据ID查询

```logic
// 基本用法
user = entity.getById("t_user", data.userId),

// 检查查询结果
user != null : (
    log.info("找到用户: " + user.f_name)
), (
    throw "用户不存在"
),

// 获取用户的具体属性
userName = user.f_name,
userEmail = user.f_email,
userStatus = user.f_status,
```

### 保存操作

#### partialSave - 保存或更新

`partialSave`根据是否有id决定插入或更新。

```logic
// 新增用户
newUser = {
    f_name: data.name,
    f_email: data.email,
    f_phone: data.phone,
    f_status: 1,
    f_create_time: dateTools.getNow2(),
    f_creator: securityUtils.getCurrentUserId()
},
result = entity.partialSave("t_user", newUser),

// 获取新生成的ID
newUserId = result.id,
log.info("新用户ID: " + newUserId),
```

```logic
// 更新用户信息
updateData = {
    id: data.userId,              // 有ID表示更新操作
    f_name: data.name,
    f_email: data.email,
    f_create_time: dateTools.getNow2(),
    f_updater: securityUtils.getCurrentUserId()
},
result = entity.partialSave("t_user", updateData),

log.info("更新用户成功: " + data.userId),
```

### 删除操作

#### deleteById - 根据ID删除

```logic
// 基本删除操作
entity.deleteById("t_user", data.userId),
log.info("删除用户: " + data.userId),

```

## sql对象 - SQL查询

`sql`对象用于执行SQL语句，适合复杂查询和跨表操作。

### SQL查询操作

#### querySQL - 执行查询SQL

```logic
// 简单查询
users = sql.querySQL("getUserList", "
    SELECT id, f_name, f_email, f_status 
    FROM t_user 
    WHERE f_status = 1
    ORDER BY f_create_time DESC
"),

```

#### 参数化查询

```logic
// 使用参数防止SQL注入
usersByDept = sql.querySQL("getUsersByDept", "
    SELECT u.id, u.f_name, u.f_email, d.f_name as dept_name
    FROM t_user u
    LEFT JOIN t_department d ON u.f_dept_id = d.id
    WHERE u.f_dept_id = {data.deptId}
    AND u.f_status = 1
    ORDER BY u.f_name
"),

// 多参数查询
activeUsers = sql.querySQL("getActiveUsers", "
    SELECT * FROM t_user 
    WHERE f_status = {data.status}
    AND f_create_time >= '{data.startDate}'
    AND f_create_time <= '{data.endDate}'
    ORDER BY f_create_time DESC
    LIMIT {data.pageSize} OFFSET {data.offset}
"),
```

### 更新操作

#### execute - 执行更新SQL

```logic
// 批量更新操作
sql.execute("updateUserStatus", "
    UPDATE t_user 
    SET f_status = {data.newStatus},
        f_update_time = '{dateTools.getNow2()}',
        f_updater = {securityUtils.getCurrentUserId()}
    WHERE f_dept_id = {data.deptId}
    AND f_status = {data.oldStatus}
"),

// 条件删除
sql.execute("deleteInactiveUsers", "
    DELETE FROM t_user 
    WHERE f_status = 0 
    AND f_delete_time < '{data.beforeDate}'
"),
```

## 最佳实践

### 参数化查询（防止SQL注入）

```logic
// ✅ 正确
users = sql.querySQL("query", "SELECT * FROM t_user WHERE f_name = {data.name}"),

// ❌ 错误
// unsafeQuery = "SELECT * FROM t_user WHERE f_name = '" + data.name + "'",
```

### 错误处理

```logic
try {
    user = entity.getById("t_user", data.userId),
    user != null : null, (throw "用户不存在"),
    result = entity.partialSave("t_user", updateData)
} catch (Exception e) {
    log.error("操作失败: {e}"),
    throw "系统错误"
}
```
