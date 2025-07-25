# Logic插件系统详解

Logic语言的强大之处在于其丰富的插件系统。插件让Logic能够调用Java方法，实现复杂的业务功能。插件分为系统通用插件和项目专用插件两大类。

## 插件系统概述

### 插件工作原理

Logic插件本质上是Java类的封装，通过XML配置文件注册到Logic引擎中。当Logic脚本调用插件时，引擎会自动转换参数类型并调用相应的Java方法。

```javascript
// Logic中调用插件的语法
result = pluginName.methodName(param1, param2),

// 示例：获取当前时间
currentTime = dateTools.now(),

// 示例：检查字符串是否非空
isValid = commonTools.isNotEmpty(data.name),
```

### 插件配置结构

插件配置位于以下位置：

- **项目级插件**：`src/main/resources/plugins.xml` (全局)
- **模块级插件**：`src/main/resources/{模块名}/plugins.xml` (模块专用)
- **系统级插件**：`../systemv4/af-common/*/src/main/resources/*/plugins.xml` (系统通用)

#### 跨平台路径说明

**项目结构示例：**
```
your-workspace/
├── your-logic-project/          # 当前Logic项目
│   ├── src/main/resources/
│   │   ├── plugins.xml         # 项目级插件
│   │   └── {module}/plugins.xml # 模块级插件
│   └── ...
└── systemv4/                    # SystemV4框架 (同级目录)
    └── af-common/
        ├── af-common-plugins/
        │   └── src/main/resources/plugins/plugins.xml
        ├── af-common-jpa/
        │   └── src/main/resources/jpaSupportForLogic/plugins.xml
        └── ...
```

**不同操作系统的路径表示：**
- **Linux/Mac**: `../systemv4/af-common/af-common-plugins/`
- **Windows**: `..\systemv4\af-common\af-common-plugins\`

#### 插件注册格式

```xml
<cfg>
    <plugin alias='插件别名' class='Java类全限定名'/>
    <plugin alias='dateTools' class='com.af.v4.system.common.plugins.date.DateTools'/>
    <plugin alias='commonTools' class='com.af.v4.system.common.plugins.core.CommonTools'/>
</cfg>
```

## 系统内置插件详解

### 基础工具插件

#### commonTools - 通用工具类

```javascript
// 字符串和集合检查
isEmpty = commonTools.isEmpty(data.name),
isNotEmpty = commonTools.isNotEmpty(data.email),
hasContent = commonTools.isNotBlank(data.description),

// UUID生成
uniqueId = commonTools.generateUUID(),

// 示例用法
commonTools.isNotEmpty(data.userName) : null, (
    throw "用户名不能为空"
),

// 生成唯一标识符
orderId = commonTools.generateUUID(),
log.info("生成订单ID: " + orderId),
```

#### convertTools - 数据类型转换

```javascript
// 字符串转数字
age = convertTools.toNumber(data.ageStr),
score = convertTools.toDouble(data.scoreStr),

// 数字转字符串
ageStr = convertTools.toString(user.age),

// 布尔值转换
isActive = convertTools.toBoolean(data.activeFlag),

// 示例：安全的类型转换
try {
    userAge = convertTools.toNumber(data.age),
    userAge > 0 : null, (
        throw "年龄必须大于0"
    )
} catch (Exception e) {
    throw "年龄格式不正确"
}
```

### 日期时间插件

#### dateTools - 日期处理工具

```javascript
// 获取当前时间
now = dateTools.now(),
currentTimestamp = dateTools.currentTimeMillis(),

// 日期格式化
dateStr = dateTools.format(now, "yyyy-MM-dd"),
timeStr = dateTools.format(now, "yyyy-MM-dd HH:mm:ss"),
customFormat = dateTools.format(now, "yyyy年MM月dd日"),

// 日期解析
birthday = dateTools.parse("1990-01-01", "yyyy-MM-dd"),
datetime = dateTools.parse("2023-12-01 10:30:00", "yyyy-MM-dd HH:mm:ss"),

// 日期计算
tomorrow = dateTools.addDays(now, 1),
lastWeek = dateTools.addDays(now, -7),
nextMonth = dateTools.addMonths(now, 1),

// 实际应用示例
createTime = dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss"),
expireTime = dateTools.format(dateTools.addDays(dateTools.now(), 30), "yyyy-MM-dd HH:mm:ss"),

userRecord = {
    f_name: data.name,
    f_create_time: createTime,
    f_expire_time: expireTime
},
```

### 数据处理插件

#### jsonTools - JSON处理工具

```javascript
// 对象转JSON字符串
userObj = {name: "张三", age: 30},
jsonStr = jsonTools.toJson(userObj),
log.info("JSON字符串: " + jsonStr),

// JSON字符串转对象
jsonData = '{"name":"李四","age":25}',
userObj = jsonTools.fromJson(jsonData),
userName = userObj.name,

// 复杂对象处理
complexData = {
    users: [
        {name: "张三", age: 30},
        {name: "李四", age: 25}
    ],
    meta: {
        total: 2,
        page: 1
    }
},
jsonOutput = jsonTools.toJson(complexData),
```

#### toTree - 树形数据转换

```javascript
// 扁平数据转树形结构
flatData = [
    {id: 1, name: "根节点", f_parent_id: null},
    {id: 2, name: "子节点1", f_parent_id: 1},
    {id: 3, name: "子节点2", f_parent_id: 1},
    {id: 4, name: "孙节点1", f_parent_id: 2}
],

treeData = toTree.convert(flatData, "id", "f_parent_id", "children"),

// 遍历树形结构
treeData.each(
    log.info("根节点: " + row.name),
    row.children != null : (
        row.children.each(
            log.info("  子节点: " + row.name)
        )
    ), null
),
```

### 网络请求插件

#### restTools - HTTP请求工具

```javascript
// GET请求
response = restTools.get("http://api.example.com/users"),
users = jsonTools.fromJson(response),

// POST请求
requestData = {
    name: data.name,
    email: data.email
},
postResponse = restTools.post("http://api.example.com/users", requestData),

// 带参数的GET请求
params = {
    page: 1,
    size: 10,
    keyword: data.keyword
},
searchResult = restTools.get("http://api.example.com/search", params),

// 错误处理
try {
    apiResult = restTools.get("http://external-api.com/data"),
    data = jsonTools.fromJson(apiResult)
} catch (Exception e) {
    log.error("API调用失败: " + e),
    throw "外部服务暂时不可用"
}
```

#### restAsyncTools - 异步HTTP请求

```javascript
// 异步GET请求
callback = (response) => {
    log.info("异步请求完成: " + response),
    // 处理响应数据
    result = jsonTools.fromJson(response)
},

restAsyncTools.getAsync("http://api.example.com/async-data", callback),

// 异步POST请求
asyncData = {
    message: "异步数据",
    timestamp: dateTools.now()
},
restAsyncTools.postAsync("http://api.example.com/async", asyncData, callback),
```

### 编码加密插件

#### Base64Tools - Base64编解码

```javascript
// Base64编码
originalText = "Hello, Logic!",
encoded = Base64Tools.encode(originalText),
log.info("编码结果: " + encoded),

// Base64解码
decoded = Base64Tools.decode(encoded),
log.info("解码结果: " + decoded),

// 文件数据编码
fileData = data.fileContent,
encodedFile = Base64Tools.encode(fileData),
```

#### secureTools - 加密工具

```javascript
// 密码加密
plainPassword = data.password,
encryptedPassword = secureTools.encrypt(plainPassword),

// 保存加密后的密码
userRecord = {
    f_username: data.username,
    f_password: encryptedPassword,
    f_create_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss")
},
```

#### sha1Tools - SHA1哈希

```javascript
// 生成哈希值
inputData = data.username + data.email,
hashValue = sha1Tools.hash(inputData),
log.info("哈希值: " + hashValue),

// 文件完整性校验
fileContent = data.fileData,
fileHash = sha1Tools.hash(fileContent),
```

## 项目专用插件

### 用户和安全相关

#### securityUtils - 当前用户信息

```javascript
// 获取当前登录用户
currentUser = securityUtils.getCurrentUser(),
currentUserId = securityUtils.getCurrentUserId(),

// 使用用户信息
log.info("当前用户: " + currentUser.username),

// 在数据保存时记录操作者
saveData = {
    f_name: data.name,
    f_creator: currentUserId,
    f_create_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss")
},
```

### 文件处理插件

#### FileSaveTools - 文件保存工具

```javascript
// 保存上传的文件
fileData = data.fileContent,
fileName = data.fileName,
savedPath = FileSaveTools.saveFile(fileData),

log.info("文件保存路径: " + savedPath),

// 保存并记录文件信息
fileRecord = {
    f_original_name: fileName,
    f_save_path: savedPath,
    f_file_size: fileData.length,
    f_upload_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss"),
    f_uploader: securityUtils.getCurrentUserId()
},
```

### 业务工具插件

#### OrderNumberGenerator - 工单编号生成

```javascript
// 生成唯一工单号
orderNumber = OrderNumberGenerator.generate(),
log.info("生成工单号: " + orderNumber),

// 创建工单记录
orderRecord = {
    f_order_no: orderNumber,
    f_title: data.title,
    f_description: data.description,
    f_creator: securityUtils.getCurrentUserId(),
    f_create_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss")
},
```

## 插件开发指南

### 自定义插件开发

如果需要开发自定义插件，需要：

1. **创建Java类**

```java
public class CustomTools {
    public static String processData(String input) {
        // 处理逻辑
        return "processed: " + input;
    }
    
    public static boolean validateData(Object data) {
        // 验证逻辑
        return data != null;
    }
}
```

2. **注册插件**

在相应的 `plugins.xml` 中添加：

```xml
<plugin alias='customTools' class='com.your.package.CustomTools'/>
```

3. **在Logic中使用**

```javascript
// 调用自定义插件
result = customTools.processData(data.input),
isValid = customTools.validateData(data.payload),
```

### 插件开发最佳实践

#### 1. 方法设计

```java
// ✅ 推荐：静态方法，简单参数
public static String formatName(String firstName, String lastName) {
    return firstName + " " + lastName;
}

// ✅ 推荐：返回具体类型
public static boolean isValidEmail(String email) {
    return email != null && email.contains("@");
}

// ❌ 避免：复杂对象参数（Logic转换困难）
public static void complexMethod(ComplexObject obj) {
    // 避免这种设计
}
```

#### 2. 异常处理

```java
public static String safeOperation(String input) {
    try {
        // 业务逻辑
        return processInput(input);
    } catch (Exception e) {
        // 记录日志并返回友好信息
        logger.error("处理失败", e);
        throw new RuntimeException("操作失败：" + e.getMessage());
    }
}
```

#### 3. 参数验证

```java
public static String validateAndProcess(String input) {
    if (input == null || input.trim().isEmpty()) {
        throw new IllegalArgumentException("输入参数不能为空");
    }
    return input.trim().toLowerCase();
}
```

## 插件使用最佳实践

### 1. 错误处理

```javascript
// 包装插件调用在try-catch中
try {
    result = restTools.get("http://external-api.com/data"),
    processedData = jsonTools.fromJson(result)
} catch (Exception e) {
    log.error("插件调用失败: " + e),
    throw "外部服务调用失败"
}
```

### 2. 参数验证

```javascript
// 调用插件前验证参数
commonTools.isNotEmpty(data.url) : null, (
    throw "URL不能为空"
),

response = restTools.get(data.url),
```

### 3. 性能考虑

```javascript
// 避免在循环中频繁调用耗时插件
items = data.items,
processedItems = [],

items.each(
    // ❌ 避免：在循环中调用HTTP请求
    // result = restTools.get("http://api.com/process/" + row.id),
    
    // ✅ 推荐：批量处理或缓存结果
    processedItems.push({
        id: row.id,
        processed: true,
        timestamp: dateTools.now()
    })
),
```

### 4. 日志记录

```javascript
// 记录重要插件调用
log.info("开始调用外部API: " + apiUrl),
response = restTools.get(apiUrl),
log.info("API调用完成，响应长度: " + response.length),
```

## 插件发现和调试

### 查找可用插件

#### Linux/Mac 环境
```bash
# 查找项目中的所有插件配置
find . -name "plugins.xml" -path "*/resources/*"

# 查看systemv4系统插件
find ../systemv4 -name "plugins.xml" -type f 2>/dev/null

# 或者使用环境变量
export SYSTEMV4_HOME="../systemv4"
find $SYSTEMV4_HOME -name "plugins.xml" -type f
```

#### Windows 环境
```cmd
# 查找项目中的所有插件配置
dir /s /b *plugins.xml | findstr resources

# 查看systemv4系统插件
dir /s /b ..\systemv4\*plugins.xml

# 或者使用环境变量
set SYSTEMV4_HOME=..\systemv4
dir /s /b %SYSTEMV4_HOME%\*plugins.xml
```

#### 通用查找方法
如果不确定systemv4的位置，可以使用以下方法：

**Linux/Mac:**
```bash
# 在上级目录中查找systemv4
find .. -name "systemv4" -type d 2>/dev/null
```

**Windows:**
```cmd
# 在上级目录中查找systemv4
dir /s /b /ad ..\systemv4
```

### 插件调试技巧

```javascript
// 1. 打印插件返回值类型和内容
result = dateTools.now(),
log.info("dateTools.now()返回类型: " + typeof(result)),
log.info("dateTools.now()返回值: " + result),

// 2. 测试插件方法是否存在
try {
    testResult = unknownPlugin.unknownMethod(),
    log.info("方法调用成功")
} catch (Exception e) {
    log.error("方法不存在或调用失败: " + e)
}

// 3. 参数类型验证
log.info("传入参数类型: " + typeof(data.input)),
result = customTools.processData(data.input),
```

## 环境配置建议

### 设置SystemV4路径环境变量

为了便于跨项目使用，建议设置环境变量：

#### Linux/Mac
```bash
# 在 ~/.bashrc 或 ~/.zshrc 中添加
export SYSTEMV4_HOME="../systemv4"

# 或者设置为绝对路径
export SYSTEMV4_HOME="/path/to/your/systemv4"
```

#### Windows
```cmd
# 设置系统环境变量
setx SYSTEMV4_HOME "..\systemv4"

# 或者在项目中设置
set SYSTEMV4_HOME=..\systemv4
```

### IDE配置

#### VS Code
在项目的 `.vscode/settings.json` 中添加：
```json
{
  "systemv4.home": "../systemv4",
  "files.associations": {
    "*.logic": "javascript"
  }
}
```

#### IntelliJ IDEA
在项目设置中添加路径变量：
- Name: `SYSTEMV4_HOME`
- Value: `../systemv4`

### 项目模板建议

创建项目时推荐的目录结构：
```
workspace/
├── systemv4/                    # SystemV4框架
│   └── af-common/
│       ├── af-common-plugins/
│       ├── af-common-jpa/
│       └── ...
├── project1/                    # Logic项目1
│   ├── src/main/resources/
│   └── ...
├── project2/                    # Logic项目2
│   ├── src/main/resources/
│   └── ...
└── shared-resources/            # 共享资源
    └── plugins/
```

通过合理使用插件系统，Logic可以实现几乎任何Java能实现的功能，大大扩展了脚本的能力边界。记住始终处理异常，验证参数，并添加适当的日志记录。