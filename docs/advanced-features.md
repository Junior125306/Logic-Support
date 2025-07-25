# Logic高级特性

Logic语言不仅提供了基础的脚本功能，还包含了许多高级特性，如Lambda表达式、异步操作、Logic调用、多数据源支持等。这些特性让Logic能够应对复杂的企业级业务场景。

## Lambda表达式

Lambda表达式让Logic支持函数式编程，可以创建可重用的代码块和回调函数。

### 基本语法

```javascript
// Lambda表达式定义语法
lambdaName = (parameters) => {
    // 函数体
    return result
},
```

### 简单Lambda示例

```javascript
// 数据处理Lambda
dataProcessor = (item) => {
    return {
        id: item.id,
        name: item.name.toUpperCase(),
        processedAt: dateTools.now()
    }
},

// 数学计算Lambda
calculator = (a, b) => {
    return a * b + 10
},

// 调用Lambda
processedItem = dataProcessor.apply({id: 1, name: "test"}),
calcResult = calculator.apply(5, 3),  // 结果: 25

log.info("处理结果: " + jsonTools.toJson(processedItem)),
log.info("计算结果: " + calcResult),
```

### Lambda变量作用域

```javascript
// 外部变量
multiplier = 3,
prefix = "处理后的:",

// Lambda可以读取外部变量，但不能修改
processor = (data) => {
    // ✅ 可以读取外部变量
    result = data.value * multiplier,
    name = prefix + data.name,
    
    // ❌ 不能修改外部变量
    // multiplier = 5,  // 这样做无效
    
    return {
        name: name,
        value: result,
        timestamp: dateTools.now()
    }
},

// 使用Lambda
inputData = {name: "测试", value: 10},
output = processor.apply(inputData),
```

### 复杂Lambda应用

```javascript
// 数据验证Lambda
validator = (user) => {
    // 姓名验证
    nameValid = commonTools.isNotEmpty(user.name),
    // 邮箱验证
    emailValid = commonTools.isNotEmpty(user.email) && user.email.indexOf("@") > 0,
    // 年龄验证
    ageValid = user.age != null && user.age > 0 && user.age < 120,
    
    return {
        isValid: nameValid && emailValid && ageValid,
        errors: [
            !nameValid ? "姓名不能为空" : null,
            !emailValid ? "邮箱格式不正确" : null,
            !ageValid ? "年龄必须在1-119之间" : null
        ].filter(item => item != null)
    }
},

// 批量验证用户数据
users = data.users,
validationResults = [],

users.each(
    validation = validator.apply(row),
    validationResults.push({
        user: row,
        validation: validation
    }),
    
    validation.isValid : (
        log.info("用户验证通过: " + row.name)
    ), (
        log.warn("用户验证失败: " + row.name + ", 错误: " + validation.errors.join(", "))
    )
),
```

### Lambda作为回调函数

```javascript
// 异步操作回调
successCallback = (response) => {
    log.info("请求成功: " + response),
    data = jsonTools.fromJson(response),
    // 处理成功响应
    return data
},

errorCallback = (error) => {
    log.error("请求失败: " + error),
    throw "网络请求失败"
},

// 使用回调（假设的异步API）
// asyncRequest(url, successCallback, errorCallback),
```

## Logic调用系统

Logic可以调用其他Logic脚本，实现模块化和代码复用。

### 基本调用语法

```javascript
// 调用其他Logic
result = logic.run("targetLogicName", parameters),

// 示例：调用用户查询Logic
userInfo = logic.run("getUserById", {
    userId: data.userId
}),

// 检查调用结果
userInfo.success : (
    log.info("获取用户信息成功: " + userInfo.data.name)
), (
    throw "获取用户信息失败: " + userInfo.message
),
```

### 复杂Logic调用链

```javascript
// 1. 验证用户权限
authResult = logic.run("checkUserPermission", {
    userId: data.userId,
    permission: "USER_MANAGE"
}),

authResult.success : null, (
    throw "权限不足: " + authResult.message
),

// 2. 获取用户详细信息
userDetail = logic.run("getUserDetailById", {
    userId: data.targetUserId,
    includeProfile: true,
    includeDepartment: true
}),

// 3. 更新用户信息
updateResult = logic.run("updateUserInfo", {
    userId: data.targetUserId,
    updateData: {
        name: data.newName,
        email: data.newEmail,
        updater: data.userId
    }
}),

// 4. 记录操作日志
logic.run("logUserOperation", {
    operatorId: data.userId,
    targetUserId: data.targetUserId,
    operation: "UPDATE_USER",
    details: "更新用户信息"
}),
```

### 异步Logic调用

```javascript
// 异步调用Logic（不等待返回结果）
logic.runAsync("sendEmailNotification", {
    userId: data.userId,
    emailType: "ACCOUNT_UPDATE",
    emailData: {
        userName: userInfo.name,
        updateTime: dateTools.now()
    }
}),

// 继续执行其他逻辑，不等待邮件发送完成
log.info("用户更新完成，邮件通知已发送"),
```

### 远程Logic调用

```javascript
// 调用其他服务的Logic
remoteResult = logic.remoteRun("user-service", "validateUser", {
    username: data.username,
    password: data.password
}),

remoteResult.success : (
    log.info("远程用户验证成功"),
    userToken = remoteResult.data.token
), (
    throw "用户验证失败: " + remoteResult.message
),
```

## 异步操作处理

Logic支持异步操作，可以处理不阻塞主流程的后台任务。

### 异步任务处理

```javascript
// 定义异步任务处理函数
asyncProcessor = (taskData) => {
    log.info("开始处理异步任务: " + taskData.taskId),
    
    // 模拟耗时操作
    try {
        // 处理业务逻辑
        result = processLongRunningTask(taskData),
        
        // 更新任务状态
        logic.run("updateTaskStatus", {
            taskId: taskData.taskId,
            status: "COMPLETED",
            result: result
        })
        
    } catch (Exception e) {
        log.error("异步任务处理失败: " + e),
        logic.run("updateTaskStatus", {
            taskId: taskData.taskId,
            status: "FAILED",
            error: e.message
        })
    }
},

// 启动异步任务
taskData = {
    taskId: commonTools.generateUUID(),
    type: data.taskType,
    parameters: data.taskParams
},

// 异步执行任务
logic.runAsync("processAsyncTask", taskData),

// 立即返回任务ID，不等待处理完成
return {
    success: true,
    taskId: taskData.taskId,
    message: "任务已提交，正在后台处理"
}
```

### 批量异步处理

```javascript
// 批量数据异步处理
batchData = data.items,
batchId = commonTools.generateUUID(),
totalItems = batchData.length,

log.info("开始批量处理，批次ID: " + batchId + ", 数据量: " + totalItems),

// 为每个数据项创建异步任务
processedCount = 0,
batchData.each(
    taskData = {
        batchId: batchId,
        itemId: row.id,
        itemData: row,
        itemIndex: rowIndex,
        totalItems: totalItems
    },
    
    // 异步处理每个项目
    logic.runAsync("processBatchItem", taskData),
    processedCount = processedCount + 1
),

// 记录批次状态
logic.run("createBatchRecord", {
    batchId: batchId,
    totalItems: totalItems,
    status: "PROCESSING",
    createTime: dateTools.now()
}),
```

## 多数据源支持

Logic支持多数据源操作，可以同时访问不同的数据库。

### 数据源配置

```xml
<!-- 在logic.xml中配置数据源 -->
<logic alias="getUserFromMaster" path="user/GetUser.logic" dataSource="master-db"/>
<logic alias="getUserFromSlave" path="user/GetUser.logic" dataSource="slave-db"/>
<logic alias="getUserFromCache" path="user/GetUser.logic" dataSource="redis-cache"/>
```

### 动态数据源切换

```javascript
// 使用只读数据库进行查询
dynamicDataSource.withDataSource("readonly-db", (queryData) => {
    // 在只读数据库中执行重查询
    heavyQueryResult = sql.querySQL("complexReport", "
        SELECT 
            u.id, u.f_name, u.f_email,
            d.f_name as dept_name,
            COUNT(o.id) as order_count,
            SUM(o.f_amount) as total_amount
        FROM t_user u
        LEFT JOIN t_department d ON u.f_dept_id = d.id
        LEFT JOIN t_order o ON u.id = o.f_user_id
        WHERE u.f_create_time >= '{queryData.startDate}'
        AND u.f_create_time <= '{queryData.endDate}'
        GROUP BY u.id, u.f_name, u.f_email, d.f_name
        ORDER BY total_amount DESC
    ")
}, {
    startDate: data.startDate,
    endDate: data.endDate
}),

// 主数据库写入操作
newUser = {
    f_name: data.name,
    f_email: data.email,
    f_create_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss")
},
savedUser = entity.partialSave("t_user", newUser),
```

### 读写分离模式

```javascript
// 写操作使用主数据库
writeToMaster = (userData) => {
    return entity.partialSave("t_user", userData)
},

// 读操作使用从数据库
readFromSlave = (userId) => {
    return dynamicDataSource.withDataSource("slave-db", (params) => {
        return entity.getById("t_user", params.userId)
    }, {userId: userId})
},

// 实际应用
newUser = {
    f_name: data.name,
    f_email: data.email,
    f_create_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss")
},

// 写入主库
savedUser = writeToMaster(newUser),
log.info("用户保存成功，ID: " + savedUser.id),

// 从从库读取（可能需要等待主从同步）
setTimeout(() => {
    userFromSlave = readFromSlave(savedUser.id),
    log.info("从从库读取用户: " + userFromSlave.f_name)
}, 1000),
```

### 跨数据源事务处理

```javascript
// 跨数据源的分布式事务（需要特别注意一致性）
try {
    // 1. 在主业务库创建订单
    order = {
        f_order_no: OrderNumberGenerator.generate(),
        f_user_id: data.userId,
        f_amount: data.amount,
        f_status: "PENDING",
        f_create_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss")
    },
    savedOrder = entity.partialSave("t_order", order),
    
    // 2. 在财务库创建支付记录
    dynamicDataSource.withDataSource("finance-db", (financeData) => {
        paymentRecord = {
            f_order_id: financeData.orderId,
            f_amount: financeData.amount,
            f_status: "CREATED",
            f_create_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss")
        },
        return entity.partialSave("t_payment", paymentRecord)
    }, {
        orderId: savedOrder.id,
        amount: data.amount
    }),
    
    // 3. 在库存库扣减库存
    dynamicDataSource.withDataSource("inventory-db", (inventoryData) => {
        sql.execute("updateInventory", "
            UPDATE t_inventory 
            SET f_quantity = f_quantity - {inventoryData.quantity}
            WHERE f_product_id = {inventoryData.productId}
            AND f_quantity >= {inventoryData.quantity}
        ")
    }, {
        productId: data.productId,
        quantity: data.quantity
    }),
    
    log.info("跨数据源事务执行成功")
    
} catch (Exception e) {
    log.error("跨数据源事务失败，需要补偿操作: " + e),
    
    // 实现补偿逻辑
    logic.runAsync("compensateFailedTransaction", {
        orderId: savedOrder.id,
        error: e.message
    }),
    
    throw "订单创建失败，请稍后重试"
}
```

## 性能优化技巧

### 批量操作优化

```javascript
// ❌ 低效：循环中的数据库操作
users = data.users,
users.each(
    // 避免在循环中进行数据库操作
    entity.partialSave("t_user", row)
),

// ✅ 高效：批量SQL操作
userValues = [],
users.each(
    userValues.push("('" + row.name + "', '" + row.email + "', '" + dateTools.now() + "')")
),

valuesString = userValues.join(", "),
sql.execute("batchInsertUsers", "
    INSERT INTO t_user (f_name, f_email, f_create_time) 
    VALUES " + valuesString
),
```

### 缓存策略

```javascript
// 使用Redis缓存提高性能
cacheKey = "user_profile_" + data.userId,

// 先尝试从缓存获取
cachedData = redis.get(cacheKey),
cachedData != null : (
    userProfile = jsonTools.fromJson(cachedData),
    log.info("从缓存获取用户资料")
), (
    // 缓存未命中，从数据库查询
    userProfile = entity.getById("t_user_profile", data.userId),
    
    // 写入缓存，设置过期时间（30分钟）
    redis.setex(cacheKey, 1800, jsonTools.toJson(userProfile)),
    log.info("从数据库查询用户资料并写入缓存")
),
```

### 异步优化

```javascript
// 将耗时操作异步化
// ❌ 同步处理耗时任务
processLargeFile(data.fileContent),  // 阻塞主流程

// ✅ 异步处理耗时任务
taskId = commonTools.generateUUID(),
logic.runAsync("processLargeFileAsync", {
    taskId: taskId,
    fileContent: data.fileContent,
    callback: data.callbackUrl
}),

// 立即返回结果
return {
    success: true,
    taskId: taskId,
    message: "文件正在后台处理，处理完成后将通知您"
}
```

## 错误处理和重试机制

### 自动重试

```javascript
// 带重试的网络请求
maxRetries = 3,
retryCount = 0,
success = false,

(retryCount < maxRetries && !success).each(
    try {
        response = restTools.get(data.apiUrl),
        success = true,
        log.info("API调用成功")
    } catch (Exception e) {
        retryCount = retryCount + 1,
        log.warn("API调用失败，重试次数: " + retryCount + ", 错误: " + e),
        
        retryCount < maxRetries : (
            // 等待一段时间再重试
            sleep(1000 * retryCount)  // 递增等待时间
        ), null
    }
),

success : null, (
    throw "API调用失败，已重试" + maxRetries + "次"
),
```

### 熔断器模式

```javascript
// 简单的熔断器实现
circuitBreakerKey = "api_" + data.serviceId,
failureCount = redis.get(circuitBreakerKey + "_failures") || 0,
lastFailureTime = redis.get(circuitBreakerKey + "_last_failure"),

// 检查熔断状态
maxFailures = 5,
breakerTimeout = 60000,  // 1分钟

failureCount >= maxFailures : (
    currentTime = dateTools.currentTimeMillis(),
    timeSinceLastFailure = currentTime - (lastFailureTime || 0),
    
    timeSinceLastFailure < breakerTimeout : (
        throw "服务熔断中，请稍后重试"
    ), (
        // 重置熔断器
        redis.del(circuitBreakerKey + "_failures"),
        redis.del(circuitBreakerKey + "_last_failure"),
        log.info("熔断器已重置")
    )
), null,

// 正常调用服务
try {
    result = restTools.get(data.apiUrl),
    // 成功时重置失败计数
    redis.del(circuitBreakerKey + "_failures")
} catch (Exception e) {
    // 记录失败
    newFailureCount = failureCount + 1,
    redis.set(circuitBreakerKey + "_failures", newFailureCount),
    redis.set(circuitBreakerKey + "_last_failure", dateTools.currentTimeMillis()),
    
    throw "服务调用失败: " + e
}
```

这些高级特性让Logic能够构建复杂、可靠、高性能的企业级应用。合理运用这些特性，可以大大提升Logic脚本的功能和可维护性。