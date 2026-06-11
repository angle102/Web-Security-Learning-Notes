# Burp Suite

# 一、Burp Suite

## 1.1 工具定位

Burp Suite 是 Web 渗透测试中最常用的抓包与测试工具。

主要作用：

```text
拦截 HTTP / HTTPS 请求
修改请求参数
重放请求
测试登录逻辑
测试越权漏洞
测试 SQL 注入、XSS、文件上传
分析 API 接口
观察 Cookie、Token、Header
```

简单理解：

```text
Burp Suite = Web 请求分析与手工漏洞验证平台
```

---

## 1.2 使用场景

Burp Suite 常用于以下场景：

```text
1. 登录框测试
2. 文件上传测试
3. API 接口测试
4. 越权测试
5. SQL 注入手工验证
6. XSS 参数测试
7. CSRF 请求分析
8. JWT / Cookie 分析
9. 接口重放测试
10. 生成漏洞复现请求
```

---

## 1.3 常用模块

| 模块           | 作用                  |
| ------------ | ------------------- |
| Proxy        | 抓包、拦截、修改请求          |
| Repeater     | 手工重放请求              |
| Intruder     | 参数枚举、弱口令验证、批量测试     |
| Decoder      | 编码、解码、Base64、URL 编码 |
| Comparer     | 对比两个响应差异            |
| HTTP history | 查看所有请求记录            |
| Target       | 查看站点结构和接口路径         |

---

## 1.4 常见测试流程

### 第一步：设置代理

浏览器代理设置为：

```text
127.0.0.1:8080
```

Burp Suite 默认监听：

```text
127.0.0.1:8080
```

如果测试 HTTPS 网站，需要安装 Burp CA 证书。

---

### 第二步：访问目标网站

例如访问：

```text
http://example.com/login
```

在 Burp 的 HTTP history 中观察请求。

重点看：

```text
请求方法 GET / POST
请求路径
参数名
Cookie
Token
Referer
User-Agent
响应状态码
响应长度
返回内容
```

---

### 第三步：发送到 Repeater

右键请求：

```text
Send to Repeater
```

在 Repeater 中修改参数并重放。

常见用途：

```text
测试 SQL 注入参数
测试 XSS 回显
测试越权 userId
测试删除 Cookie 后是否还能访问
测试修改 role / isAdmin 是否生效
测试上传文件 MIME 和文件名
```

---

## 1.5 典型场景：登录框测试

登录请求示例：

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

username=admin&password=123456
```

测试点：

```text
是否存在弱口令
是否存在用户名枚举
是否限制登录失败次数
验证码是否可绕过
是否存在 SQL 注入
登录失败和成功响应是否明显不同
Cookie 是否设置 HttpOnly / Secure / SameSite
```

---

## 1.6 典型场景：越权测试

正常请求：

```http
GET /api/user/info?id=1001 HTTP/1.1
Cookie: JSESSIONID=xxx
```

修改参数：

```text
id=1002
```

判断：

```text
如果用户 A 能查看用户 B 的信息，可能存在水平越权。
```

越权测试重点：

```text
userId
orderId
fileId
tenantId
role
isAdmin
departmentId
```

---

## 1.7 Burp 输出内容

可以沉淀到报告中的内容：

```text
漏洞请求包
漏洞响应包
关键参数
Cookie / Token 状态
响应差异
复现截图
风险说明
修复建议
```

---

