# Logic开发最佳实践

本指南基于大量的Logic开发经验，总结了企业级Logic开发的最佳实践，帮助开发者编写高质量、可维护、高性能的Logic代码。

## 代码结构和组织

### 标准Logic文件结构

```javascript
// 1. 文件头注释
// 功能：用户信息查询和更新
// 作者：开发者姓名
// 创建时间：2023-12-01
// 最后修改：2023-12-01

// 2. 参数验证
validate {
    userId: {
        required: true,
        message: "用户ID不能为空"
    },
    updateData: {
        required: false
    }
},

// 3. 参数获取和预处理
userId = data.userId,
updateData = data.updateData,
currentTime = dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss"),
currentUser = securityUtils.getCurrentUser(),

// 4. 业务逻辑验证
commonTools.isNotEmpty(userId) : null, (
    throw "用户ID不能为空"
),

// 5. 核心业务处理
user = entity.getById("t_user", userId),
user != null : null, (
    throw "用户不存在"
),

// 6. 数据操作
updateData != null : (
    // 更新操作
    updateRecord = {
        id: userId,
        f_name: updateData.name,
        f_email: updateData.email,
        f_update_time: currentTime,
        f_updater: currentUser.id
    },
    result = entity.partialSave("t_user", updateRecord),
    log.info("更新用户信息成功: " + userId)
), (
    // 查询操作
    result = user,
    log.info("查询用户信息: " + user.f_name)
),

// 7. 返回结果
return {
    success: true,
    data: result,
    message: updateData != null ? "更新成功" : "查询成功"
}
```

### 模块化设计

```javascript
// ✅ 推荐：将复杂逻辑拆分为多个Logic
// UserService.logic - 用户基础服务
validateUser = logic.run("validateUserData", {
    userData: data.user
}),

userInfo = logic.run("getUserProfile", {
    userId: data.userId
}),

// ❌ 避免：在单个Logic中处理所有逻辑
// 避免创建过长的Logic文件（超过200行）
```

## 命名规范

### Logic文件命名

```javascript
// ✅ 推荐的命名模式
// Get + 业务对象 + [具体操作]
GetUserById.logic
GetUsersByDepartment.logic
GetUserProfile.logic

// Save + 业务对象 + [具体操作]
SaveUserInfo.logic
SaveUserProfile.logic
SaveUserPassword.logic

// Delete + 业务对象 + [具体操作]
DeleteUserById.logic
DeleteUserBatch.logic

// 业务动作 + 业务对象
ValidateUserData.logic
SendUserNotification.logic
```

### 变量命名

```javascript
// ✅ 推荐：有意义的变量名
currentUser = securityUtils.getCurrentUser(),
userProfile = entity.getById("t_user_profile", userId),
updateData = data.userInfo,
validationResult = validateUserData(userData),

// ❌ 避免：无意义的变量名
u = securityUtils.getCurrentUser(),
p = entity.getById("t_user_profile", userId),
d = data.userInfo,
r = validateUserData(userData),
```

### 常量定义

```javascript
// 在Logic开头定义常量
USER_STATUS_ACTIVE = 1,
USER_STATUS_INACTIVE = 0,
USER_STATUS_DELETED = -1,

DEFAULT_PAGE_SIZE = 20,
MAX_PAGE_SIZE = 100,

// 使用常量
user.f_status = USER_STATUS_ACTIVE,
pageSize = data.pageSize != null ? data.pageSize : DEFAULT_PAGE_SIZE,
```

## 错误处理策略

### 参数验证

```javascript
// 多层次参数验证
validate {
    // 1. 框架级别验证
    userId: {
        required: true,
        message: "用户ID是必需的"
    }
},

// 2. 业务级别验证
commonTools.isNotEmpty(data.userId) : null, (
    throw "用户ID不能为空"
),

convertTools.toNumber(data.userId) != null : null, (
    throw "用户ID必须是数字"
),

data.userId > 0 : null, (
    throw "用户ID必须大于0"
),
```

### 异常处理模式

```javascript
// ✅ 推荐：统一异常处理
try {
    // 业务逻辑
    user = entity.getById("t_user", data.userId),
    user != null : null, (
        throw "用户不存在"
    ),
    
    result = processUserData(user)
    
} catch (Exception e) {
    // 记录详细错误信息
    log.error("处理用户数据失败, userId: " + data.userId + ", 错误: " + e),
    
    // 返回用户友好的错误信息
    throw {
        msg: "用户数据处理失败，请稍后重试",
        code: "USER_PROCESS_ERROR",
        details: e.message
    }
}
```

### 错误分类处理

```javascript
// 业务错误 - 返回具体错误信息
validateBusinessRule(data) : null, (
    throw {
        type: "BUSINESS_ERROR",
        msg: "业务规则验证失败",
        code: "VALIDATION_FAILED"
    }
),

// 系统错误 - 记录日志并返回通用错误
try {
    result = externalApiCall(data)
} catch (Exception e) {
    log.error("外部API调用失败: " + e),
    throw {
        type: "SYSTEM_ERROR",
        msg: "系统繁忙，请稍后重试",
        code: "EXTERNAL_API_ERROR"
    }
}
```

## 数据库操作规范

### 查询优化

```javascript
// ✅ 推荐：使用参数化查询
users = sql.querySQL("getUsersByCondition", "
    SELECT u.id, u.f_name, u.f_email, d.f_name as dept_name
    FROM t_user u
    LEFT JOIN t_department d ON u.f_dept_id = d.id
    WHERE u.f_status = {data.status}
    AND u.f_create_time >= '{data.startDate}'
    ORDER BY u.f_create_time DESC
    LIMIT {data.pageSize} OFFSET {data.offset}
"),

// ❌ 避免：在循环中执行查询
users.each(
    // 不要在这里执行数据库查询
    profile = entity.getById("t_user_profile", row.id)  // 避免
),
```

### 事务处理

```javascript
// 复杂事务操作的最佳实践
try {
    // 1. 数据准备和验证
    user = entity.getById("t_user", data.userId),
    user != null : null, (
        throw "用户不存在"
    ),
    
    // 2. 业务规则检查
    validateBusinessRules(user, data.operation),
    
    // 3. 执行数据库操作
    // 更新用户状态
    updateUser = {
        id: data.userId,
        f_status: data.newStatus,
        f_update_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss"),
        f_updater: securityUtils.getCurrentUserId()
    },
    entity.partialSave("t_user", updateUser),
    
    // 记录操作日志
    operationLog = {
        f_user_id: data.userId,
        f_operation: data.operation,
        f_operator: securityUtils.getCurrentUserId(),
        f_create_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss")
    },
    entity.partialSave("t_operation_log", operationLog),
    
    log.info("用户状态更新成功: " + data.userId)
    
} catch (Exception e) {
    log.error("用户状态更新失败: " + e),
    throw "操作失败，请稍后重试"
}
```

## 性能优化

### 查询优化

```javascript
// ✅ 一次查询获取所有需要的数据
userDetails = sql.querySQL("getUserWithProfile", "
    SELECT 
        u.id, u.f_name, u.f_email, u.f_status,
        p.f_avatar, p.f_bio, p.f_phone,
        d.f_name as dept_name
    FROM t_user u
    LEFT JOIN t_user_profile p ON u.id = p.f_user_id
    LEFT JOIN t_department d ON u.f_dept_id = d.id
    WHERE u.id = {data.userId}
"),

// ❌ 避免：多次查询
user = entity.getById("t_user", data.userId),
profile = entity.getById("t_user_profile", user.f_profile_id),
department = entity.getById("t_department", user.f_dept_id),
```

### 批量操作

```javascript
// ✅ 批量插入优化
users = data.users,
batchSize = 100,
currentBatch = [],
processedCount = 0,

users.each(
    currentBatch.push({
        f_name: row.name,
        f_email: row.email,
        f_create_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss")
    }),
    
    // 达到批次大小或最后一批时执行插入
    (currentBatch.length >= batchSize || rowIndex == users.length - 1) : (
        // 构建批量SQL
        valuesList = [],
        currentBatch.each(
            valuesList.push("('" + row.f_name + "', '" + row.f_email + "', '" + row.f_create_time + "')")
        ),
        
        sql.execute("batchInsertUsers", "
            INSERT INTO t_user (f_name, f_email, f_create_time) 
            VALUES " + valuesList.join(", ")
        ),
        
        processedCount = processedCount + currentBatch.length,
        log.info("已处理用户数: " + processedCount),
        
        // 清空当前批次
        currentBatch = []
    ), null
),
```

### 缓存策略

```javascript
// 实现简单的内存缓存
CACHE_PREFIX = "user_cache_",
CACHE_EXPIRY = 1800,  // 30分钟

getUserFromCache = (userId) => {
    cacheKey = CACHE_PREFIX + userId,
    cachedData = redis.get(cacheKey),
    
    cachedData != null : (
        log.info("从缓存获取用户: " + userId),
        return jsonTools.fromJson(cachedData)
    ), (
        // 从数据库查询
        user = entity.getById("t_user", userId),
        user != null : (
            // 写入缓存
            redis.setex(cacheKey, CACHE_EXPIRY, jsonTools.toJson(user)),
            log.info("用户数据写入缓存: " + userId)
        ), null,
        return user
    )
},

// 使用缓存
user = getUserFromCache(data.userId),
```

## 日志记录规范

### 日志级别使用

```javascript
// debug - 详细的调试信息
log.debug("开始处理用户数据, 参数: " + jsonTools.toJson(data)),

// info - 重要的业务流程信息
log.info("用户登录成功, userId: " + user.id + ", IP: " + request.clientIp),

// warn - 警告信息，不影响正常流程
log.warn("用户密码即将过期, userId: " + user.id + ", 过期时间: " + user.f_password_expire),

// error - 错误信息，影响正常流程
log.error("数据库连接失败: " + e.message),
```

### 结构化日志

```javascript
// ✅ 推荐：结构化日志信息
logData = {
    action: "USER_LOGIN",
    userId: user.id,
    userName: user.f_name,
    clientIp: request.clientIp,
    userAgent: request.userAgent,
    timestamp: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss"),
    success: true
},
log.info("用户操作日志: " + jsonTools.toJson(logData)),

// ❌ 避免：不规范的日志信息
log.info("用户" + user.f_name + "在" + dateTools.now() + "登录了"),
```

## 安全最佳实践

### 输入验证和清理

```javascript
// SQL注入防护
userInput = data.searchKeyword,
// 使用参数化查询，不要直接拼接SQL
users = sql.querySQL("searchUsers", "
    SELECT * FROM t_user 
    WHERE f_name LIKE '%{userInput}%'
    AND f_status = 1
"),

// XSS防护（在返回前端前）
safeUserName = escapeHtml(user.f_name),
safeUserBio = escapeHtml(user.f_bio),
```

### 权限控制

```javascript
// 统一的权限检查
checkPermission = (userId, permission) => {
    userPermissions = logic.run("getUserPermissions", {
        userId: userId
    }),
    
    return userPermissions.permissions.indexOf(permission) >= 0
},

// 在业务逻辑前检查权限
currentUserId = securityUtils.getCurrentUserId(),
checkPermission(currentUserId, "USER_MANAGE") : null, (
    throw {
        msg: "权限不足",
        code: "ACCESS_DENIED",
        status: 403
    }
),
```

### 敏感数据处理

```javascript
// 密码处理
plainPassword = data.password,
encryptedPassword = secureTools.encrypt(plainPassword),

// 保存用户时不包含敏感信息
userToSave = {
    f_name: data.name,
    f_email: data.email,
    f_password: encryptedPassword,  // 加密后的密码
    // 不保存原始密码
},

// 返回给前端时移除敏感字段
userResponse = {
    id: user.id,
    name: user.f_name,
    email: user.f_email,
    // 不返回密码字段
},
```

## 测试和调试

### 调试技巧

```javascript
// 调试信息输出
DEBUG_MODE = true,  // 生产环境设为false

DEBUG_MODE : (
    log.info("调试信息 - 用户数据: " + jsonTools.toJson(user)),
    log.info("调试信息 - 处理参数: " + jsonTools.toJson(data)),
    log.info("调试信息 - 当前步骤: 开始处理用户验证")
), null,

// 条件断点
user.f_status == 0 : (
    log.warn("发现非活跃用户: " + user.id),
    // 可以在这里设置特殊处理逻辑用于调试
), null,
```

### 单元测试模拟

```javascript
// 创建测试数据
TEST_MODE = data.testMode || false,

TEST_MODE : (
    // 测试模式下使用模拟数据
    mockUser = {
        id: 1,
        f_name: "测试用户",
        f_email: "test@example.com",
        f_status: 1
    },
    user = mockUser,
    log.info("使用测试数据")
), (
    // 正常模式下查询数据库
    user = entity.getById("t_user", data.userId)
),
```

## 代码复用和模块化

### 公共Logic封装

```javascript
// CommonUtils.logic - 公共工具Logic
validateRequired = (value, fieldName) => {
    return commonTools.isNotEmpty(value) ? null : (fieldName + "不能为空")
},

formatDate = (date, pattern) => {
    return dateTools.format(date, pattern || "yyyy-MM-dd HH:mm:ss")
},

getCurrentUser = () => {
    return securityUtils.getCurrentUser()
},

// 在其他Logic中调用
validation = logic.run("CommonUtils", {
    action: "validateRequired",
    value: data.userName,
    fieldName: "用户名"
}),

validation.error : (
    throw validation.error
), null,
```

### 配置管理

```javascript
// 在Logic开头定义配置
CONFIG = {
    maxRetries: 3,
    timeoutMs: 5000,
    batchSize: 100,
    cacheExpiry: 1800
},

// 使用配置
retryCount = 0,
(retryCount < CONFIG.maxRetries).each(
    // 重试逻辑
),
```

## 版本控制和部署

### 版本标记

```javascript
// 在Logic文件头部标记版本信息
/*
 * Version: 1.2.0
 * Last Updated: 2023-12-01
 * Changes: 添加批量处理功能，优化查询性能
 * Author: 张三
 */
```

### 向后兼容

```javascript
// 处理API版本兼容
apiVersion = data.version || "1.0",

apiVersion == "2.0" : (
    // 新版本逻辑
    result = processV2(data)
), apiVersion == "1.1" : (
    // 1.1版本逻辑
    result = processV1_1(data)
), (
    // 默认版本逻辑
    result = processV1(data)
),
```

### 灰度发布支持

```javascript
// 功能开关
FEATURE_FLAGS = {
    enableNewUserFlow: true,
    enableAdvancedSearch: false
},

// 根据功能开关选择逻辑
FEATURE_FLAGS.enableNewUserFlow : (
    result = logic.run("NewUserRegistration", data)
), (
    result = logic.run("LegacyUserRegistration", data)
),
```

遵循这些最佳实践，可以确保Logic代码的质量、可维护性和性能，同时提高开发效率和系统稳定性。