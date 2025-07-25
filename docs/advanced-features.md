# Logic高级特性

## Lambda表达式

### 定义和调用

```logic
// 定义Lambda
dataProcessor = (item) => {
    return {
        id: item.id,
        name: item.name.toUpperCase(),
        processedAt: dateTools.now()
    }
},

// 调用Lambda
result = dataProcessor.apply({id: 1, name: "test"}),
```

### 变量作用域

```logic
// 外部变量
multiplier = 3,

// Lambda可以读取外部变量，但不能修改
processor = (data) => {
    return data.value * multiplier  // 可以访问外部变量
},

result = processor.apply({value: 10}),  // 结果: 30
```

## Logic调用

### 同步调用

```logic
// 调用其他Logic
result = logic.run("targetLogicName", parameters),

// 示例
userInfo = logic.run("getUserById", {userId: data.userId}),
```

### 异步调用

```logic
// 异步调用Logic
logic.runAsync("processDataAsync", {
    data: largeDataSet,
    callback: "handleAsyncResult"
}),

// 回调处理函数
handleAsyncResult = (result) => {
    log.info("异步处理完成: " + result.status)
},
```

### 远程调用

```logic
// 调用远程服务的Logic
remoteResult = logic.remoteRun("userService", "getUserProfile", {
    userId: data.userId
}),
```

## 异步操作

### 异步任务

```logic
// 提交异步任务
taskId = asyncTask.submit("heavyProcessing", {
    dataSet: largeData,
    options: processingOptions
}),

// 检查任务状态
status = asyncTask.getStatus(taskId),

// 获取任务结果
status == "COMPLETED" : (
    result = asyncTask.getResult(taskId)
), null,
```

### 并发处理

```logic
// 并发执行多个任务
tasks = [
    {name: "task1", params: {type: "A"}},
    {name: "task2", params: {type: "B"}},
    {name: "task3", params: {type: "C"}}
],

results = [],
tasks.each(
    taskResult = logic.runAsync(row.name, row.params),
    results.push(taskResult)
),
```

## 多数据源操作

### 数据源切换

```logic
// 使用指定数据源
dynamicDataSource.withDataSource("readOnlyDB", (data) => {
    users = sql.querySQL("heavyQuery", "SELECT * FROM t_user"),
    return users
}, {}),
```

### 读写分离

```logic
// 写操作（主库）
user = entity.partialSave("t_user", userData),

// 读操作（从库）
dynamicDataSource.withDataSource("slaveDB", (params) => {
    userList = sql.querySQL("getUserList", "SELECT * FROM t_user")
}, {}),
```

## 事务管理

### 事务边界

```logic
// 事务性操作
try {
    // 多个相关操作在同一事务中
    user = entity.partialSave("t_user", userData),
    profile = entity.partialSave("t_user_profile", profileData),
    
    // 记录操作日志
    sql.execute("insertLog", "INSERT INTO t_log VALUES(...)"),
    
    log.info("事务提交成功")
    
} catch (Exception e) {
    log.error("事务回滚: " + e),
    throw "操作失败"
}
```

### 分布式事务

```logic
// 跨服务事务
distributedTransaction.begin("userRegistration"),

try {
    // 本地操作
    user = entity.partialSave("t_user", userData),
    
    // 远程服务调用
    profileResult = logic.remoteRun("profileService", "createProfile", {
        userId: user.id,
        profileData: data.profile
    }),
    
    // 确认事务
    distributedTransaction.commit("userRegistration")
    
} catch (Exception e) {
    distributedTransaction.rollback("userRegistration"),
    throw "分布式事务失败"
}
```
