# Logic插件系统详解

Logic语言的强大之处在于其丰富的插件系统。插件让Logic能够调用Java方法，实现复杂的业务功能。插件分为系统通用插件和项目专用插件两大类。

## 插件系统概述

### 插件工作原理

Logic插件本质上是Java类的封装，通过XML配置文件注册到Logic引擎中。当Logic脚本调用插件时，引擎会自动转换参数类型并调用相应的Java方法。

```logic
// Logic中调用插件的语法
result = pluginName.methodName(param1, param2),

// 示例：获取当前时间
currentTime = dateTools.getNow(),

// 示例：获取一个用"-"分隔的UUID
UUID = commonTools.getUUID(data.name),
```

### 插件配置结构

插件配置位于以下位置：

- **项目级插件**：`src/main/resources/plugins.xml` (全局)
- **模块级插件**：`src/main/resources/{模块名}/plugins.xml` (模块专用)
- **系统级插件**：`../systemv4/af-common/***/plugins.xml` (系统通用)

#### 跨平台路径说明

**项目结构示例：**

```umi
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

- **Linux/Mac**: `../systemv4/af-common/**/plugins.xml`
- **Windows**: `..\systemv4\af-common\**\plugins.xml`

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

```logic
// UUID生成（标准带分隔符）
uniqueId = commonTools.getUUID(),
// UUID生成（简化版不带分隔符）
simpleUUID = commonTools.getUUID(true),

// MD5加密
password = commonTools.md5(data.password),
// MD5加密（带盐值）
securePassword = commonTools.md5(data.password, "salt123"),

// 随机数生成
randomNum = commonTools.getRandomNumber(1, 100),

// 数字判断
isNum = commonTools.isNumeric(data.input),

// 字符串操作
isContained = commonTools.isContains(data.text, "search"),
parts = commonTools.split(data.text, ","),

// 精确数学运算
sum = commonTools.add(data.num1, data.num2),
difference = commonTools.sub(data.num1, data.num2),
product = commonTools.mul(data.num1, data.num2),
quotient = commonTools.div(data.num1, data.num2),

// 示例用法
orderId = commonTools.getUUID(),
log.info("生成订单ID: {orderId}"),
```

#### convertTools - 数据类型转换

```logic
// 字符串转数字
age = convertTools.stringToInt(data.ageStr),
score = convertTools.stringToDouble(data.scoreStr),
price = convertTools.stringToBigDecimal(data.priceStr),

// 数字转字符串
priceStr = convertTools.bigDecimalToString(data.price),

// Base64编解码
encoded = convertTools.base64Encode(data.bytes),
decoded = convertTools.base64Decode(data.encodedBytes),

// 数值取整操作
roundUp = convertTools.ceil(data.value),      // 向上取整
roundDown = convertTools.floor(data.value),   // 向下取整
rounded = convertTools.round(data.value),     // 四舍五入

// 十六进制转换
hexStr = convertTools.byteToHexStr(data.bytes),
bytes = convertTools.hexStrToByte(data.hexString),
intVal = convertTools.hexStrToInt(data.hexString),

// 示例：安全的类型转换
try {
    userAge = convertTools.stringToInt(data.age),
    userAge > 0 : null, (
        throw "年龄必须大于0"
    )
} catch (Exception e) {
    throw "年龄格式不正确"
}
```

### 日期时间插件

#### dateTools - 日期处理工具

```logic
// 获取当前时间
now = dateTools.getNow(),                    // 返回Date对象
nowStr = dateTools.getNow2(),               // 返回"yyyy-MM-dd HH:mm:ss"格式字符串
currentTimestamp = dateTools.getCurrentTimeMillis(),      // 毫秒时间戳
currentSeconds = dateTools.getCurrentTimeSeconds(),       // 秒时间戳

// 自定义格式获取当前时间
dateStr = dateTools.getNow("yyyy-MM-dd"),
timeStr = dateTools.getNow("yyyy-MM-dd HH:mm:ss"),
customFormat = dateTools.getNow("yyyy年MM月dd日"),

// 获取当前时间的年月日
currentYear = dateTools.getNowYear(),
currentMonth = dateTools.getNowMonth(),
currentDay = dateTools.getNowDay(),
dayOfWeek = dateTools.getNowDayOfWeek(),

// 日期格式化
formattedDate = dateTools.format("2023-12-01 10:30:00", "yyyy-MM-dd"),
standardDateTime = dateTools.formatDateTime("2023-12-01"),

// 日期比较
isLater = dateTools.compareDate("2023-12-02", "2023-12-01"),  // true表示第一个日期>=第二个

// 日期计算
tomorrow = dateTools.getDelayDate(nowStr, "DAY", "1"),
lastWeek = dateTools.getDelayDate(nowStr, "DAY", "-7"),
nextMonth = dateTools.getDelayDate(nowStr, "MONTH", "1"),

// 获取时间间隔
timeDiff = dateTools.getDateBetween("2023-12-01 10:00:00", "2023-12-01 15:30:00"),
dayDiff = dateTools.getDateDayBetween("2023-12-01", "2023-12-05", true),

// 时间戳转换
timestampToDate = dateTools.timestampToDate(1701417600000),
dateToTimestamp = dateTools.dateToTimestamp("2023-12-01 10:00:00"),

// 实际应用示例
createTime = dateTools.getNow2(),
expireTime = dateTools.getDelayDate(createTime, "DAY", "30"),

userRecord = {
    f_name: data.name,
    f_create_time: createTime,
    f_expire_time: expireTime
},
```

### 数据处理插件

#### jsonTools - JSON处理工具

```logic
// 字符串转JSON对象
jsonData = "{\"name\":\"李四\",\"age\":25}",
userObj = jsonTools.convertToJson(jsonData),
userName = userObj.name,

// 字符串转JSON数组
arrayData = "[{\"name\":\"张三\"}, {\"name\":\"李四\"}]",
userArray = jsonTools.parseArray(arrayData),

// JSON对象合并
baseObj = {name: "张三", age: 30},
addObj = {email: "test@example.com", phone: "123456"},
mergedObj = jsonTools.addJSON(baseObj, addObj),

// JSON对象替换
oldObj = {name: "张三", age: 30},
newObj = {name: "李四", age: 25},
replacedObj = jsonTools.replaceJSON(oldObj, newObj),

// 读取JSON文件
fileData = jsonTools.readJsonFile("path/to/file.json"),
arrayFromFile = jsonTools.readJsonArrayFile("path/to/array.json"),

// 创建新的JSON对象和数组
newJson = jsonTools.getJson(),
newArray = jsonTools.getArray(),
```

#### toTree - 树形数据转换

```logic
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

```logic
// GET请求
response = restTools.get("http://api.example.com/users", null),
users = jsonTools.convertToJson(response),

// GET请求（带请求头）
headers = {"Authorization":"Bearer token"}
headersStr = "{headers}",
response = restTools.get("http://api.example.com/users", headersStr),

// POST请求（JSON数据）
requestData = {
    name: data.name,
    email: data.email
},
postResponse = restTools.post("http://api.example.com/users", requestData),

// POST请求（字符串数据，带请求头）
jsonStr = '{"name":"张三","age":30}',
headers = '{"Content-Type":"application/json"}',
postResponse = restTools.post("http://api.example.com/users", jsonStr, headers),

// POST表单数据
formData = "name=张三&age=30",
formResponse = restTools.postByFormData("http://api.example.com/form", formData, headers),

// PUT和DELETE请求
putResponse = restTools.put("http://api.example.com/users/1", requestData),
deleteResponse = restTools.delete("http://api.example.com/users/1", headers),

// 错误处理
try {
    apiResult = restTools.get("http://external-api.com/data", null),
    data = jsonTools.convertToJson(apiResult)
} catch (Exception e) {
    log.error("API调用失败: {e}"),
    throw "外部服务暂时不可用"
}
```

#### restAsyncTools - 异步HTTP请求

```logic
// 异步GET请求
callback = (response) => {
    log.info("异步请求完成: {response}"),
    // 处理响应数据
    result = jsonTools.convertToJson(response)
},

restAsyncTools.getAsync("http://api.example.com/async-data", callback),

// 异步POST请求
asyncData = {
    message: "异步数据",
    timestamp: dateTools.getNow2()
},
restAsyncTools.postAsync("http://api.example.com/async", asyncData, callback),
```

### 编码加密插件

#### Base64Tools - Base64编解码

```logic
// Base64编码
originalText = "Hello, Logic!",
encoded = Base64Tools.encode(originalText),
log.info("编码结果: {encoded}"),

// Base64解码
decoded = Base64Tools.decode(encoded),
log.info("解码结果: {decoded}"),

// 文件数据编码
fileData = data.fileContent,
encodedFile = Base64Tools.encode(fileData),
```

#### secureTools - 加密工具

```logic
// AES加密解密
key = "your-base64-encoded-key",
plainText = data.password,
encryptedText = secureTools.AESEncrypt(plainText, key),
decryptedText = secureTools.AESDecrypt(encryptedText, key),

// AES CBC模式加密解密
hexKey = "your-hex-key",
encryptedCBC = secureTools.AESEncryptCBC(plainText, hexKey),
decryptedCBC = secureTools.AESDecryptCBC(encryptedCBC, hexKey),

// RSA密钥对生成
keyPair = secureTools.generateRSAKeyPair(),
publicKey = keyPair.publicKey,
privateKey = keyPair.privateKey,

// RSA加密解密
rsaEncrypted = secureTools.RSAEncrypt(plainText, publicKey),
rsaDecrypted = secureTools.RSADecrypt(rsaEncrypted, privateKey),

// RSA数字签名
signature = secureTools.sign(data.content, privateKey),
isValid = secureTools.verify(data.content, signature, publicKey),

// 实际应用示例
userRecord = {
    f_username: data.username,
    f_password: secureTools.AESEncrypt(data.password, key),
    f_create_time: dateTools.getNow2()
},
```

#### sha1Tools - SHA1哈希

```logic
// 生成哈希值
inputData = data.username + data.email,
hashValue = sha1Tools.hash(inputData),
log.info("哈希值: {hashValue}"),

// 文件完整性校验
fileContent = data.fileData,
fileHash = sha1Tools.hash(fileContent),
```

## 项目专用插件

除了系统内置插件外，Logic项目还可以自定义业务专用插件。这些插件通常在项目的 `src/main/resources/plugins.xml` 中配置。

### 自定义插件示例

#### 用户上下文插件

```logic
// 示例：假设项目中有用户上下文插件
currentUser = userContext.getCurrentUser(),
currentUserId = userContext.getCurrentUserId(),

// 在数据保存时记录操作者
saveData = {
    f_name: data.name,
    f_creator: currentUserId,
    f_create_time: dateTools.getNow2()
},
```

#### 文件处理插件

```logic
// 示例：假设项目中有文件保存插件
fileData = data.fileContent,
fileName = data.fileName,
savedPath = fileManager.saveFile(fileData, fileName),

log.info("文件保存路径: {savedPath}"),
```

#### 业务工具插件

```logic
// 示例：假设项目中有编号生成插件
orderNumber = numberGenerator.generateOrderNumber(),
ticketNumber = numberGenerator.generateTicketNumber(),

log.info("生成订单号: {orderNumber}"),
```

## 插件发现

### 查找可用插件

#### Linux/Mac 环境

```bash
# 查找项目中的所有插件配置
find . -name "plugins.xml" -path "*/resources/*"

# 查看systemv4系统插件
find ../systemv4 -name "plugins.xml" -type f 2>/dev/null

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
