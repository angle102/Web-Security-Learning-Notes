# Sqlmap

# sqlmap 使用笔记

> 本文仅用于授权安全测试、靶场练习、CTF 学习和 SQL 注入漏洞验证。
> 未经授权不得对真实网站、服务器、数据库或第三方资产使用 sqlmap 进行扫描、测试、枚举或数据导出。
> 真实项目中应遵循最小化验证原则，能证明漏洞存在即可，不应拖库、破坏数据或影响业务运行。

---

## 一、sqlmap 是什么

sqlmap 是一款开源的 SQL 注入自动化检测与验证工具。

它可以帮助安全测试人员自动化完成：

```text
SQL 注入检测
数据库类型识别
注入类型判断
当前数据库信息验证
当前数据库用户验证
请求包注入测试
Cookie 注入测试
POST 参数注入测试
JSON 参数注入测试
```

简单理解：

```text
sqlmap = SQL 注入自动化辅助验证工具
```

它不是替代手工分析的工具，而是用于辅助确认漏洞、提高测试效率。

---

## 二、sqlmap 使用前提

使用 sqlmap 前必须确认：

```text
1. 是否获得授权
2. 目标是否属于测试范围
3. 是否允许自动化扫描
4. 是否允许 SQL 注入验证
5. 是否允许读取数据库基础信息
6. 是否禁止导出数据
7. 是否限制测试时间
8. 是否需要控制扫描频率
```

安全原则：

```text
先手工判断，再用 sqlmap 辅助验证
先低强度检测，再根据授权提高强度
只验证漏洞存在，不随意枚举数据
不使用高风险参数影响业务
不对生产系统进行大规模自动化测试
```

---

## 三、适用场景

sqlmap 常用于以下场景：

```text
1. GET 参数 SQL 注入验证
2. POST 表单 SQL 注入验证
3. 登录框参数 SQL 注入验证
4. Cookie 参数 SQL 注入验证
5. Header 参数 SQL 注入验证
6. JSON 接口 SQL 注入验证
7. 搜索框 SQL 注入验证
8. 订单、用户、文章详情接口验证
9. 手工判断后辅助确认注入类型
10. CTF / 靶场 SQL 注入练习
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
username
password
sort
order
```

---

## 四、sqlmap 安装方式

### 4.1 Kali Linux

Kali 中通常自带 sqlmap。

查看版本：

```bash
sqlmap --version
```

查看帮助：

```bash
sqlmap -h
```

查看完整帮助：

```bash
sqlmap -hh
```

---

### 4.2 GitHub 安装

可以通过 Git 克隆：

```bash
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev
```

进入目录：

```bash
cd sqlmap-dev
```

运行：

```bash
python3 sqlmap.py -h
```

---

### 4.3 pip 安装

部分环境可使用：

```bash
pip install sqlmap
```

升级：

```bash
pip install --upgrade sqlmap
```

---

## 五、常用参数总览

| 参数                | 作用            |
| ----------------- | ------------- |
| `-u`              | 指定目标 URL      |
| `-r`              | 从请求包文件中读取请求   |
| `-p`              | 指定要测试的参数      |
| `--data`          | 指定 POST 数据    |
| `--cookie`        | 指定 Cookie     |
| `--headers`       | 指定请求头         |
| `--user-agent`    | 指定 User-Agent |
| `--batch`         | 自动使用默认选项      |
| `--level`         | 测试等级，范围 1-5   |
| `--risk`          | 风险等级，范围 1-3   |
| `--current-db`    | 获取当前数据库名      |
| `--current-user`  | 获取当前数据库用户     |
| `--banner`        | 获取数据库版本信息     |
| `--dbms`          | 指定数据库类型       |
| `--technique`     | 指定注入技术类型      |
| `--flush-session` | 清除本地缓存结果      |
| `--proxy`         | 指定代理          |
| `--random-agent`  | 随机 User-Agent |

---

## 六、基础使用

### 6.1 GET 参数检测

目标：

```text
http://target.com/news.php?id=1
```

命令：

```bash
sqlmap -u "http://target.com/news.php?id=1" --batch
```

参数说明：

```text
-u：指定目标 URL
--batch：自动选择默认选项，减少交互
```

适用场景：

```text
详情页
新闻页
商品页
用户查询接口
订单查询接口
```

---

### 6.2 指定参数检测

如果 URL 中有多个参数：

```text
http://target.com/news.php?id=1&type=2
```

只测试 `id` 参数：

```bash
sqlmap -u "http://target.com/news.php?id=1&type=2" -p id --batch
```

参数说明：

```text
-p id：只检测 id 参数
```

适用场景：

```text
URL 参数较多
只想验证某一个可疑参数
避免无意义扫描
减少请求数量
```

---

### 6.3 POST 参数检测

POST 请求示例：

```text
username=admin&password=123456
```

命令：

```bash
sqlmap -u "http://target.com/login.php" --data="username=admin&password=123456" --batch
```

指定参数：

```bash
sqlmap -u "http://target.com/login.php" --data="username=admin&password=123456" -p username --batch
```

适用场景：

```text
登录框
搜索框
表单提交
后台筛选
注册接口
```

---

## 七、使用 Burp 请求包检测

实际测试中，很多请求都带有 Cookie、Token、Header、JSON 数据或复杂参数。

这时推荐使用 Burp Suite 抓包，然后交给 sqlmap 检测。

---

### 7.1 Burp 保存请求包

在 Burp Suite 中：

```text
1. 抓取目标请求
2. 右键请求
3. Copy to file
4. 保存为 request.txt
```

示例请求包：

```http
POST /api/user/search HTTP/1.1
Host: target.com
Cookie: JSESSIONID=xxxx
Content-Type: application/x-www-form-urlencoded

keyword=test&page=1
```

---

### 7.2 sqlmap 读取请求包

命令：

```bash
sqlmap -r request.txt --batch
```

指定参数：

```bash
sqlmap -r request.txt -p keyword --batch
```

适用场景：

```text
需要登录态的后台接口
带 Cookie 的请求
带 CSRF Token 的请求
JSON 请求
复杂 Header 请求
API 接口测试
```

---

### 7.3 JSON 接口检测

JSON 请求示例：

```http
POST /api/search HTTP/1.1
Host: target.com
Content-Type: application/json
Cookie: token=xxxx

{"keyword":"test","page":1}
```

保存为 `request.txt` 后执行：

```bash
sqlmap -r request.txt -p keyword --batch
```

说明：

```text
JSON 接口建议优先使用 -r 请求包方式。
这样可以保留完整 Header、Cookie 和请求体格式。
```

---

## 八、Cookie 注入检测

部分系统可能把用户 ID、语言、排序字段等信息放在 Cookie 中。

示例：

```text
Cookie: uid=1; role=user
```

命令：

```bash
sqlmap -u "http://target.com/user.php" --cookie="uid=1; role=user" -p uid --batch
```

适用场景：

```text
Cookie 中存在 uid
Cookie 中存在 role
Cookie 中存在 trackingId
Cookie 中存在查询条件
```

注意：

```text
Cookie 注入测试必须在授权环境中进行。
不要测试不属于授权范围的真实业务系统。
```

---

## 九、控制扫描强度

sqlmap 的两个重要参数是：

```text
--level
--risk
```

---

### 9.1 level 参数

`--level` 表示测试等级，范围是：

```text
1 到 5
```

默认是：

```text
1
```

等级越高，测试点越多，请求数量也越多。

建议：

```text
真实授权项目中优先使用 level=1
必要时再逐步提高
```

示例：

```bash
sqlmap -u "http://target.com/news.php?id=1" --level=1 --risk=1 --batch
```

---

### 9.2 risk 参数

`--risk` 表示风险等级，范围是：

```text
1 到 3
```

默认是：

```text
1
```

风险等级越高，可能使用更激进的测试 payload。

建议：

```text
生产环境中优先使用 risk=1
除非明确授权，不要随意提高 risk
```

---

### 9.3 推荐安全检测命令

真实授权项目中建议先使用：

```bash
sqlmap -u "http://target.com/news.php?id=1" --level=1 --risk=1 --batch
```

请求包方式：

```bash
sqlmap -r request.txt -p id --level=1 --risk=1 --batch
```

说明：

```text
先低强度验证。
如果没有结果，再结合手工判断决定是否提高测试强度。
```

---

## 十、常见注入技术类型

sqlmap 支持多种 SQL 注入技术。

| 技术  | 含义                       |
| --- | ------------------------ |
| `B` | Boolean-based blind，布尔盲注 |
| `E` | Error-based，报错注入         |
| `U` | UNION query，联合查询注入       |
| `S` | Stacked queries，堆叠查询     |
| `T` | Time-based blind，时间盲注    |
| `Q` | Inline queries，内联查询      |

可以使用：

```bash
sqlmap -u "http://target.com/news.php?id=1" --technique=BEU --batch
```

说明：

```text
--technique=BEU 表示只测试布尔盲注、报错注入和 UNION 注入。
```

实际项目中常用思路：

```text
有回显：优先关注 UNION、报错注入
无回显但页面有差异：关注布尔盲注
无回显且页面无差异：关注时间盲注
```

---

## 十一、数据库基础信息验证

在授权靶场或项目中，通常只需要验证数据库基础信息即可证明风险。

### 11.1 获取当前数据库名

```bash
sqlmap -u "http://target.com/news.php?id=1" --current-db --batch
```

请求包方式：

```bash
sqlmap -r request.txt -p id --current-db --batch
```

---

### 11.2 获取当前数据库用户

```bash
sqlmap -u "http://target.com/news.php?id=1" --current-user --batch
```

---

### 11.3 获取数据库版本

```bash
sqlmap -u "http://target.com/news.php?id=1" --banner --batch
```

---

### 11.4 最小化验证原则

真实项目中建议只验证：

```text
当前数据库名
当前数据库用户
数据库版本
注入类型
存在漏洞的参数
```

不建议直接执行：

```text
全库枚举
全表枚举
数据导出
系统命令执行
文件读写
```

除非项目授权书明确允许。

---

## 十二、指定数据库类型

如果已经知道数据库类型，可以使用 `--dbms` 指定。

例如 MySQL：

```bash
sqlmap -u "http://target.com/news.php?id=1" --dbms=mysql --batch
```

SQL Server：

```bash
sqlmap -u "http://target.com/news.php?id=1" --dbms=mssql --batch
```

Oracle：

```bash
sqlmap -u "http://target.com/news.php?id=1" --dbms=oracle --batch
```

PostgreSQL：

```bash
sqlmap -u "http://target.com/news.php?id=1" --dbms=postgresql --batch
```

作用：

```text
减少无效测试
提高检测效率
降低请求数量
```

---

## 十三、代理到 Burp Suite

有时需要观察 sqlmap 发送了哪些请求，可以把 sqlmap 流量代理到 Burp。

命令：

```bash
sqlmap -u "http://target.com/news.php?id=1" --proxy="http://127.0.0.1:8080" --batch
```

作用：

```text
观察 sqlmap 请求
分析 payload
保存复现请求
判断是否被 WAF 拦截
方便写报告截图
```

注意：

```text
代理到 Burp 会增加测试时间。
```

---

## 十四、清除缓存重新检测

sqlmap 会缓存检测结果。

如果目标参数、Cookie、Token 或请求发生变化，可以清除缓存。

命令：

```bash
sqlmap -u "http://target.com/news.php?id=1" --flush-session --batch
```

请求包方式：

```bash
sqlmap -r request.txt --flush-session --batch
```

适用场景：

```text
更换了目标参数
更换了 Cookie
更换了请求包
前一次检测结果异常
需要重新验证
```

---

## 十五、常见结果怎么看

如果存在 SQL 注入，sqlmap 可能输出类似：

```text
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind
```

含义：

```text
Parameter：存在漏洞的参数
id：参数名
GET：参数位置
Type：注入类型
Title：检测到的具体注入方式
```

其他常见类型：

```text
boolean-based blind：布尔盲注
time-based blind：时间盲注
error-based：报错注入
UNION query：联合查询注入
stacked queries：堆叠查询
```

---

## 十六、常见使用场景示例

### 16.1 详情页参数

目标：

```text
/news.php?id=1
```

命令：

```bash
sqlmap -u "http://target.com/news.php?id=1" -p id --current-db --batch
```

---

### 16.2 搜索框 POST 参数

请求：

```text
keyword=test&page=1
```

命令：

```bash
sqlmap -u "http://target.com/search.php" --data="keyword=test&page=1" -p keyword --current-db --batch
```

---

### 16.3 登录框参数

请求：

```text
username=admin&password=123456
```

命令：

```bash
sqlmap -u "http://target.com/login.php" --data="username=admin&password=123456" -p username --batch
```

注意：

```text
登录框测试应控制频率。
不要进行弱口令爆破或高频请求。
```

---

### 16.4 后台接口请求包

保存 Burp 请求为：

```text
request.txt
```

命令：

```bash
sqlmap -r request.txt -p id --current-db --batch
```

适合：

```text
后台接口
带 Token 的请求
带 Cookie 的请求
JSON 请求
复杂 POST 请求
```

---

## 十七、sqlmap 与手工注入的关系

sqlmap 适合辅助验证，但不能完全代替手工判断。

推荐流程：

```text
1. 手工找参数
2. 手工判断参数是否影响页面
3. 手工测试单引号、布尔条件、时间延迟
4. 初步判断存在 SQL 注入可能
5. 使用 sqlmap 辅助验证
6. 输出漏洞参数、注入类型和最小化证明
7. 编写报告和修复建议
```

不要一上来就直接大范围扫描。

原因：

```text
容易产生大量请求
可能影响业务
可能被 WAF 拦截
可能造成误报
不利于理解漏洞原理
```

---

## 十八、报告中如何写 sqlmap 结果

报告中建议保留：

```text
目标 URL
请求方法
漏洞参数
参数位置
sqlmap 命令
sqlmap 识别出的注入类型
当前数据库名或数据库版本
关键截图
风险影响
修复建议
```

示例：

```text
漏洞名称：
SQL 注入漏洞

漏洞位置：
/news.php?id=1

漏洞参数：
id

验证方式：
使用 sqlmap 进行辅助验证。

验证命令：
sqlmap -u "http://target.com/news.php?id=1" -p id --current-db --batch

验证结果：
sqlmap 识别 id 参数存在 boolean-based blind 注入，并成功获取当前数据库名。

影响分析：
攻击者可通过构造 SQL 注入语句读取数据库信息，可能造成敏感数据泄露。

修复建议：
使用预编译 SQL；禁止字符串拼接；对参数进行白名单校验；关闭详细错误回显；数据库账号最小权限。
```

---

## 十九、注意事项

使用 sqlmap 时应注意：

```text
1. 必须确认授权范围
2. 不要对未授权目标使用
3. 不要直接高强度扫描
4. 不要随意导出数据库数据
5. 不要使用高风险功能影响业务
6. 不要频繁使用时间盲注造成服务压力
7. 扫描结果必须人工复核
8. sqlmap 结果不能直接等于漏洞结论
9. 报告中应写明测试时间、测试 IP 和测试范围
10. 真实项目中应优先做最小化证明
```

---

## 二十、常见问题

### 20.1 sqlmap 没有检测出来，是不是一定不存在漏洞？

不是。

可能原因：

```text
参数需要闭合方式特殊
请求缺少 Cookie 或 Token
需要登录态
WAF 拦截
参数经过加密
参数在 JSON 中
响应页面差异不明显
测试强度过低
目标存在二次注入
```

应结合手工测试继续分析。

---

### 20.2 为什么建议使用 Burp 请求包？

因为很多真实请求依赖：

```text
Cookie
Token
CSRF 参数
Authorization Header
Referer
JSON 请求体
特殊 Content-Type
```

直接用 `-u` 可能缺少上下文，导致检测失败。

---

### 20.3 level 和 risk 能不能直接开最高？

不建议。

原因：

```text
请求数量变多
测试时间变长
可能影响业务
可能触发 WAF
部分 payload 风险更高
```

推荐：

```text
先 level=1 risk=1
再根据授权范围逐步调整
```

---

### 20.4 sqlmap 可以直接用于报告吗？

可以作为辅助证据，但必须人工复核。

报告中最好同时包含：

```text
手工判断过程
sqlmap 验证结果
漏洞参数
响应截图
修复建议
```

---

## 二十一、学习靶场推荐

适合练习 sqlmap 的靶场：

```text
DVWA
sqli-labs
Pikachu
WebGoat
CTFHub SQL 注入模块
BUUOJ Web 题
PortSwigger Web Security Academy
```

建议练习顺序：

```text
1. GET 数字型注入
2. GET 字符型注入
3. POST 表单注入
4. Cookie 注入
5. Header 注入
6. JSON 参数注入
7. 布尔盲注
8. 时间盲注
9. 报错注入
10. UNION 查询注入
```

---

## 二十二、学习总结

sqlmap 的核心作用：

```text
辅助检测和验证 SQL 注入漏洞
```

常用输入方式：

```text
-u：直接指定 URL
-r：读取 Burp 请求包
--data：指定 POST 数据
--cookie：指定 Cookie
```

常用验证参数：

```text
--current-db
--current-user
--banner
-p
--level
--risk
--batch
```

推荐使用流程：

```text
手工判断
        ↓
Burp 抓包
        ↓
保存请求包
        ↓
sqlmap 指定参数低强度检测
        ↓
最小化验证数据库基础信息
        ↓
人工复核
        ↓
整理报告
```

一句话总结：

```text
sqlmap 是 SQL 注入辅助验证工具，真正重要的是知道什么时候用、测哪个参数、如何控制强度、如何复核结果以及如何写进报告。
```
