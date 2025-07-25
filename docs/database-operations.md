# Logic数据库操作指南

Logic语言提供了强大且简洁的数据库操作能力，主要通过`entity`和`sql`两个内置对象实现。这些操作封装了复杂的数据库交互，让开发者能够专注于业务逻辑。

## entity对象 - 实体操作

`entity`对象提供了面向对象的数据库操作方式，适合处理单表的CRUD操作。

### 查询操作

#### getById - 根据ID查询

```javascript
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

#### 复杂查询场景

```javascript
// 查询多个相关实体
user = entity.getById("t_user", data.userId),
profile = entity.getById("t_user_profile", user.f_profile_id),
department = entity.getById("t_department", user.f_dept_id),

// 条件检查和数据组装
user != null && profile != null : (
    userInfo = {
        id: user.id,
        name: user.f_name,
        email: user.f_email,
        avatar: profile.f_avatar,
        bio: profile.f_bio,
        deptName: department != null ? department.f_name : "未分配"
    }
), (
    throw "用户信息不完整"
),
```

### 保存操作

#### partialSave - 保存或更新

`partialSave`是最常用的保存方法，当记录的`id`为null时执行插入，否则执行更新。

```javascript
// 新增用户
newUser = {
    f_name: data.name,
    f_email: data.email,
    f_phone: data.phone,
    f_status: 1,
    f_create_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss"),
    f_creator: securityUtils.getCurrentUserId()
},
result = entity.partialSave("t_user", newUser),

// 获取新生成的ID
newUserId = result.id,
log.info("新用户ID: " + newUserId),
```

```javascript
// 更新用户信息
updateData = {
    id: data.userId,              // 有ID表示更新操作
    f_name: data.name,
    f_email: data.email,
    f_update_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss"),
    f_updater: securityUtils.getCurrentUserId()
},
result = entity.partialSave("t_user", updateData),

log.info("更新用户成功: " + data.userId),
```

#### 批量保存

```javascript
// 批量保存用户
users = data.users,
savedUsers = [],

users.each(
    // 为每个用户添加系统字段
    userToSave = {
        f_name: row.name,
        f_email: row.email,
        f_status: 1,
        f_create_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss"),
        f_creator: securityUtils.getCurrentUserId()
    },
    
    // 保存用户
    savedUser = entity.partialSave("t_user", userToSave),
    savedUsers.push(savedUser),
    
    log.info("保存用户: " + row.name + ", ID: " + savedUser.id)
),
```

### 删除操作

#### deleteById - 根据ID删除

```javascript
// 基本删除操作
entity.deleteById("t_user", data.userId),
log.info("删除用户: " + data.userId),

// 带条件检查的删除
user = entity.getById("t_user", data.userId),
user != null : (
    // 检查是否可以删除
    user.f_status != 0 : (
        entity.deleteById("t_user", data.userId),
        log.info("删除用户成功: " + user.f_name)
    ), (
        throw "用户已被删除"
    )
), (
    throw "用户不存在"
),
```

#### 软删除模式

```javascript
// 软删除：更新状态而不是物理删除
softDeleteData = {
    id: data.userId,
    f_status: 0,                  // 状态设为0表示删除
    f_delete_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss"),
    f_deleter: securityUtils.getCurrentUserId()
},
result = entity.partialSave("t_user", softDeleteData),
log.info("软删除用户: " + data.userId),
```

## sql对象 - SQL查询

`sql`对象提供了直接执行SQL语句的能力，适合复杂查询、统计分析和跨表操作。

### 查询操作

#### querySQL - 执行查询SQL

```javascript
// 简单查询
users = sql.querySQL("getUserList", "
    SELECT id, f_name, f_email, f_status 
    FROM t_user 
    WHERE f_status = 1
    ORDER BY f_create_time DESC
"),

// 处理查询结果
users.each(
    log.info("用户: " + row.f_name + " (" + row.f_email + ")")
),
```

#### 参数化查询

```javascript
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

#### 复杂统计查询

```javascript
// 用户统计报表
userStats = sql.querySQL("getUserStats", "
    SELECT 
        d.f_name as dept_name,
        COUNT(*) as user_count,
        COUNT(CASE WHEN u.f_status = 1 THEN 1 END) as active_count,
        COUNT(CASE WHEN u.f_status = 0 THEN 1 END) as inactive_count
    FROM t_user u
    LEFT JOIN t_department d ON u.f_dept_id = d.id
    WHERE u.f_create_time >= '{data.startDate}'
    GROUP BY d.id, d.f_name
    ORDER BY user_count DESC
"),

// 处理统计结果
totalUsers = 0,
userStats.each(
    totalUsers = totalUsers + row.user_count,
    log.info("部门: " + row.dept_name + ", 用户数: " + row.user_count)
),
log.info("总用户数: " + totalUsers),
```

### 更新操作

#### execute - 执行更新SQL

```javascript
// 批量更新操作
sql.execute("updateUserStatus", "
    UPDATE t_user 
    SET f_status = {data.newStatus},
        f_update_time = '{dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss")}',
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

#### 事务性操作

```javascript
// 在try-catch中执行多个相关操作
try {
    // 1. 更新用户信息
    sql.execute("updateUser", "
        UPDATE t_user 
        SET f_name = '{data.name}', f_email = '{data.email}'
        WHERE id = {data.userId}
    "),
    
    // 2. 插入操作日志
    sql.execute("insertLog", "
        INSERT INTO t_operation_log (f_user_id, f_operation, f_create_time)
        VALUES ({data.userId}, '更新用户信息', '{dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss")}')
    "),
    
    log.info("用户信息更新成功")
    
} catch (Exception e) {
    log.error("更新用户信息失败: " + e),
    throw "系统错误，请稍后重试"
}
```

## 数据库操作最佳实践

### 1. 参数安全

```javascript
// ✅ 正确：使用参数化查询
users = sql.querySQL("safeQuery", "
    SELECT * FROM t_user 
    WHERE f_name LIKE '%{data.keyword}%'
    AND f_status = {data.status}
"),

// ❌ 错误：直接拼接SQL（存在注入风险）
// 不要这样写！
unsafeQuery = "SELECT * FROM t_user WHERE f_name = '" + data.name + "'",
```

### 2. 错误处理

```javascript
// 数据库操作应该包装在try-catch中
try {
    user = entity.getById("t_user", data.userId),
    user != null : null, (
        throw "用户不存在"
    ),
    
    result = entity.partialSave("t_user", updateData)
    
} catch (Exception e) {
    log.error("数据库操作失败: " + e),
    throw {
        msg: "数据保存失败，请稍后重试",
        status: 500
    }
}
```

### 3. 数据验证

```javascript
// 保存前进行数据验证
validate {
    name: {
        required: true,
        message: "姓名不能为空"
    },
    email: {
        required: true,
        message: "邮箱不能为空"
    }
},

// 业务逻辑验证
commonTools.isNotEmpty(data.name) : null, (
    throw "姓名不能为空"
),

// 邮箱格式验证（假设有相应插件）
isValidEmail(data.email) : null, (
    throw "邮箱格式不正确"
),
```

### 4. 性能优化

```javascript
// 避免在循环中进行数据库操作
users = sql.querySQL("getAllUsers", "SELECT * FROM t_user"),

// ❌ 错误：在循环中查询数据库
users.each(
    // 不要在这里执行entity.getById或sql.querySQL
),

// ✅ 正确：使用JOIN查询一次获取所有数据
usersWithProfiles = sql.querySQL("getUsersWithProfiles", "
    SELECT u.*, p.f_avatar, p.f_bio
    FROM t_user u
    LEFT JOIN t_user_profile p ON u.id = p.f_user_id
    WHERE u.f_status = 1
"),
```

### 5. 日志记录

```javascript
// 记录关键数据库操作
log.info("开始查询用户列表, 部门ID: " + data.deptId),

users = sql.querySQL("getUsersByDept", sqlQuery),

log.info("查询完成, 找到用户数: " + users.length),

// 记录重要的业务操作
user = entity.getById("t_user", data.userId),
user.f_status = 0,
result = entity.partialSave("t_user", user),

log.info("禁用用户: " + user.f_name + " (ID: " + user.id + ")"),
```

## 多数据源支持

### 数据源切换

```javascript
// 在Logic注册时指定数据源
// <logic alias="getUserFromSlaveDB" path="user/GetUser.logic" dataSource="slave-db"/>

// 运行时动态切换数据源
dynamicDataSource.withDataSource("readonly-db", (data) => {
    // 在只读数据库中执行查询
    users = sql.querySQL("heavyQuery", "
        SELECT * FROM t_user u
        LEFT JOIN t_department d ON u.f_dept_id = d.id
        WHERE u.f_create_time >= '{data.startDate}'
        ORDER BY u.f_create_time DESC
    ")
}, {
    startDate: data.startDate
}),
```

### 读写分离

```javascript
// 写操作使用主数据库
writeUser = {
    f_name: data.name,
    f_email: data.email,
    f_create_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss")
},
newUser = entity.partialSave("t_user", writeUser),

// 读操作可以使用从数据库
dynamicDataSource.withDataSource("slave-db", (queryData) => {
    userList = sql.querySQL("getUserList", "
        SELECT * FROM t_user 
        WHERE f_status = 1 
        ORDER BY f_create_time DESC
    ")
}, {}),
```

## 常见问题和解决方案

### 1. 主键生成

```javascript
// entity.partialSave会自动处理主键生成
newRecord = {
    f_name: "测试用户",
    f_create_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss")
},
savedRecord = entity.partialSave("t_user", newRecord),

// 获取自动生成的ID
generatedId = savedRecord.id,
log.info("生成的ID: " + generatedId),
```

### 2. 时间字段处理

```javascript
// 标准时间格式
currentTime = dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss"),

// 保存时间字段
record = {
    f_name: data.name,
    f_create_time: currentTime,
    f_update_time: currentTime
},
```

### 3. 空值处理

```javascript
// 检查查询结果是否为空
user = entity.getById("t_user", data.userId),
user != null : (
    userName = user.f_name != null ? user.f_name : "未知用户",
    userEmail = user.f_email != null ? user.f_email : ""
), (
    throw "用户不存在"
),
```

通过这些数据库操作方法，你可以在Logic中高效地处理各种数据持久化需求。记住始终使用参数化查询，合理处理异常，并添加适当的日志记录。