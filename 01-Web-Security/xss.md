# XSS

# XSS 漏洞复现笔记

> 本文仅用于授权安全测试、靶场练习、CTF 学习和代码审计学习。
> 未经授权不得对真实网站进行 XSS 测试、攻击、数据窃取或绕过行为。
> 本文示例均以本地靶场、DVWA、Pikachu、WebGoat、PortSwigger Academy 或授权测试环境为前提。

---

## 一、XSS 漏洞是什么

XSS，全称是 **Cross-Site Scripting**，中文叫 **跨站脚本攻击**。

它是指 Web 应用在接收用户输入后，没有对输入内容进行正确过滤、转义或编码，导致攻击者输入的 HTML / JavaScript 代码被浏览器当作页面内容执行。

简单理解：

```text
用户输入的数据被浏览器当作脚本执行了。
```

例如一个搜索页面：

```text
/search?keyword=test
```

页面返回：

```html
搜索结果：test
```

如果用户输入：

```html
<script>alert(1)</script>
```

页面直接输出：

```html
搜索结果：<script>alert(1)</script>
```

浏览器就会执行其中的 JavaScript，从而触发 XSS。

---

## 二、XSS 产生原因

XSS 的根本原因是：

```text
用户可控输入被输出到页面时，没有进行正确的上下文编码或安全过滤。
```

危险代码示例：

```java
@GetMapping("/hello")
public void hello(String name, HttpServletResponse response) throws IOException {
    response.getWriter().write("<h1>Hello " + name + "</h1>");
}
```

如果用户访问：

```text
/hello?name=<script>alert(1)</script>
```

后端返回：

```html
<h1>Hello <script>alert(1)</script></h1>
```

浏览器会执行脚本。

---

## 三、XSS 常见类型

XSS 常见分为三类：

```text
1. 反射型 XSS
2. 存储型 XSS
3. DOM 型 XSS
```

---

## 四、反射型 XSS

### 4.1 原理

反射型 XSS 是指用户输入通过请求参数提交到服务器，服务器把输入内容直接返回到当前页面，浏览器执行了其中的脚本。

<img width="620" height="401" alt="image" src="https://github.com/user-attachments/assets/bc952d6d-5a30-4ef5-b358-a999549202f8" />


流程：

```text
用户输入恶意参数
        ↓
服务器接收参数
        ↓
服务器把参数原样返回页面
        ↓
浏览器执行脚本
```

---

### 4.2 触发场景

反射型 XSS 常见于：

```text
搜索框
登录失败提示
错误信息页面
跳转参数
URL 参数回显
留言预览
调试页面
```

常见参数名：

```text
keyword
search
q
msg
error
redirect
returnUrl
callback
name
title
```

---

### 4.3 判断方法

目标示例：

```text
http://target.com/search?keyword=test
```

第一步：输入普通字符，观察是否回显。

```text
/search?keyword=test123
```

如果页面显示：

```text
搜索结果：test123
```

说明参数存在回显。

第二步：输入 HTML 标签测试。

```html
<b>test</b>
```

如果页面显示加粗效果，说明 HTML 可能被解析。

第三步：输入安全测试 payload。

```html
<script>alert(1)</script>
```

如果弹窗触发，说明可能存在反射型 XSS。

---

### 4.4 复现示例

请求：

```text
http://target.com/search?keyword=<script>alert(1)</script>
```

页面返回：

```html
搜索结果：<script>alert(1)</script>
```

浏览器触发弹窗：

```text
alert(1)
```

结论：

```text
keyword 参数存在反射型 XSS。
```

---

## 五、存储型 XSS

### 5.1 原理

存储型 XSS 是指恶意输入被保存到数据库、日志、评论、用户资料等位置，之后其他用户访问相关页面时，恶意脚本被取出并执行。

<img width="515" height="473" alt="image" src="https://github.com/user-attachments/assets/8f2e76aa-5df6-4307-aba2-7a47503d9966" />


流程：

```text
攻击者提交恶意内容
        ↓
后端保存到数据库
        ↓
其他用户访问页面
        ↓
页面读取数据库内容并展示
        ↓
浏览器执行脚本
```

---

### 5.2 触发场景

存储型 XSS 常见于：

```text
留言板
评论区
用户昵称
个人简介
文章发布
商品评价
工单内容
后台富文本
文件名展示
站内信
日志查看页面
```

---

### 5.3 判断方法

以评论区为例。

第一步：提交普通内容。

```text
hello-xss-test
```

第二步：刷新页面，查看评论内容是否被保存并展示。

第三步：提交安全测试 payload。

```html
<script>alert(1)</script>
```

第四步：刷新页面或使用另一个账号访问该页面。

如果脚本被执行，说明存在存储型 XSS。

---

### 5.4 复现示例

评论内容：

```html
<script>alert(1)</script>
```

页面展示：

```html
<div class="comment">
    <script>alert(1)</script>
</div>
```

结果：

```text
其他用户访问评论页面时触发弹窗。
```

结论：

```text
评论功能存在存储型 XSS。
```

---

## 六、DOM 型 XSS

### 6.1 原理

DOM 型 XSS 是指漏洞发生在前端 JavaScript 代码中，攻击者控制的数据被前端脚本读取后，写入到 DOM 中并被浏览器执行。

这种情况下，恶意数据不一定经过服务器处理。

流程：

```text
用户控制 URL / Hash / 参数
        ↓
前端 JavaScript 读取数据
        ↓
使用 innerHTML / document.write 等危险方式写入页面
        ↓
浏览器执行脚本
```

---

### 6.2 触发场景

DOM 型 XSS 常见于：

```text
前端路由
URL hash
搜索参数
跳转参数
错误提示
前端模板渲染
单页面应用
```

常见危险 Source：

```javascript
location.href
location.search
location.hash
document.URL
document.referrer
window.name
localStorage
sessionStorage
```

常见危险 Sink：

```javascript
innerHTML
outerHTML
document.write()
eval()
setTimeout()
setInterval()
insertAdjacentHTML()
```

---

### 6.3 危险代码示例

```javascript
let msg = location.hash.substring(1);
document.getElementById("content").innerHTML = msg;
```

如果访问：

```text
http://target.com/#<img src=x onerror=alert(1)>
```

前端会把 hash 内容写入页面：

```html
<div id="content">
    <img src=x onerror=alert(1)>
</div>
```

从而触发脚本。

---

## 七、XSS 常见测试位置

测试 XSS 时重点关注用户输入和页面输出位置。

常见输入点：

```text
搜索框
登录框
注册框
评论框
留言框
昵称
签名
个人简介
文章标题
文章内容
商品评价
文件名
URL 参数
请求头
Cookie
富文本编辑器
```

常见输出点：

```text
HTML 标签内容
HTML 属性
JavaScript 字符串
URL 链接
CSS 样式
富文本区域
错误提示
后台列表
日志页面
```

---

## 八、不同上下文的 XSS

XSS 判断时不能只看是否过滤 `<script>`，还要看用户输入出现在什么上下文中。

---

### 8.1 HTML 标签内容中

示例：

```html
<div>用户输入</div>
```

测试：

```html
<script>alert(1)</script>
```

或：

```html
<img src=x onerror=alert(1)>
```

---

### 8.2 HTML 属性中

示例：

```html
<input value="用户输入">
```

如果输入：

```html
" onmouseover="alert(1)
```

可能变成：

```html
<input value="" onmouseover="alert(1)">
```

风险点：

```text
属性闭合
事件属性注入
引号过滤不完整
```

---

### 8.3 JavaScript 字符串中

示例：

```html
<script>
var name = "用户输入";
</script>
```

如果输入没有正确转义，可能破坏 JS 字符串结构。

风险点：

```text
引号
反斜杠
换行
</script>
```

---

### 8.4 URL 上下文中

示例：

```html
<a href="用户输入">点击</a>
```

风险点：

```text
javascript: 协议
data: 协议
跳转参数
开放重定向
```

修复时应限制协议为：

```text
http
https
```

---

## 九、XSS 的危害

XSS 的危害取决于触发位置、用户权限和系统功能。

常见危害包括：

```text
盗取用户会话信息
冒充用户执行操作
修改页面内容
伪造登录框
诱导用户钓鱼
读取页面敏感信息
调用用户权限下的接口
后台管理员被攻击后影响更大
传播蠕虫式攻击
```

在后台管理系统中，XSS 风险更高，因为管理员通常有更高权限。

例如：

```text
普通用户提交恶意昵称
        ↓
管理员后台查看用户列表
        ↓
触发存储型 XSS
        ↓
以管理员身份执行敏感操作
```

---

## 十、XSS 判断流程

完整判断流程：

```text
1. 找输入点
2. 判断输入是否回显
3. 判断回显位置
4. 判断是否被过滤
5. 判断是否被 HTML 编码
6. 根据上下文构造测试 payload
7. 观察是否触发脚本
8. 判断漏洞类型
9. 评估影响范围
10. 给出修复建议
```

---

## 十一、XSS 复现报告模板

```text
漏洞名称：XSS 跨站脚本漏洞

漏洞类型：
反射型 XSS / 存储型 XSS / DOM 型 XSS

漏洞位置：
/search?keyword=

漏洞参数：
keyword

漏洞等级：
中危 / 高危

漏洞描述：
目标系统在处理 keyword 参数时，未对用户输入进行正确的输出编码或安全过滤，导致用户输入的脚本内容被浏览器解析执行，存在跨站脚本攻击风险。

复现步骤：
1. 访问目标页面，确认 keyword 参数会在页面中回显。
2. 输入普通测试字符串，确认页面输出位置。
3. 输入测试 payload：<script>alert(1)</script>
4. 页面触发 JavaScript 弹窗，证明存在 XSS 漏洞。

影响分析：
攻击者可构造恶意链接或提交恶意内容，诱导用户访问后在其浏览器中执行脚本。若管理员访问触发页面，可能导致后台操作被冒用、敏感信息泄露或业务数据被篡改。

修复建议：
对用户输入进行上下文相关的输出编码；富文本使用白名单过滤；避免使用 innerHTML、document.write 等危险 API；Cookie 设置 HttpOnly、Secure、SameSite；配置 CSP；后端与前端共同进行安全处理。
```

---

## 十二、修复方案

### 12.1 输出编码

XSS 防护的核心是：

```text
在输出位置进行上下文相关编码。
```

不同上下文需要不同编码方式：

| 输出位置           | 防护方式          |
| -------------- | ------------- |
| HTML 标签内容      | HTML 实体编码     |
| HTML 属性        | 属性编码          |
| JavaScript 字符串 | JS 字符串转义      |
| URL 参数         | URL 编码        |
| CSS 内容         | CSS 编码或避免用户可控 |

例如 HTML 中应将特殊字符编码：

| 字符  | 编码后      |
| --- | -------- |
| `<` | `&lt;`   |
| `>` | `&gt;`   |
| `"` | `&quot;` |
| `'` | `&#x27;` |
| `&` | `&amp;`  |

---

### 12.2 输入校验

输入校验可以降低风险，但不能完全替代输出编码。

建议：

```text
限制长度
限制字符集
限制格式
过滤明显危险内容
业务字段使用白名单
```

例如手机号字段只允许数字：

```text
^1[3-9]\d{9}$
```

年龄字段只允许数字范围。

---

### 12.3 富文本白名单过滤

如果业务允许用户输入富文本，例如文章内容、评论、公告，需要使用白名单过滤。

允许：

```text
p
br
strong
em
ul
ol
li
a
img
table
```

禁止：

```text
script
iframe
object
embed
onerror
onclick
javascript:
data:
```

注意：

```text
富文本不能简单用黑名单过滤，推荐使用成熟的 HTML Sanitizer。
```

---

### 12.4 避免危险前端 API

前端开发中应避免直接把用户输入写入 HTML。

危险写法：

```javascript
element.innerHTML = userInput;
document.write(userInput);
```

更安全的方式：

```javascript
element.textContent = userInput;
```

如果必须渲染 HTML，应使用安全的白名单清洗库。

---

### 12.5 Cookie 安全属性

为了降低 XSS 后的影响，应设置 Cookie 安全属性。

```text
HttpOnly：禁止 JavaScript 读取 Cookie
Secure：仅 HTTPS 传输 Cookie
SameSite：降低 CSRF 风险
```

示例：

```http
Set-Cookie: JSESSIONID=abc123; HttpOnly; Secure; SameSite=Lax
```

注意：

```text
HttpOnly 不能修复 XSS，但可以降低 Cookie 被脚本读取的风险。
```

---

### 12.6 配置 CSP

CSP，全称 Content Security Policy，即内容安全策略。

它可以限制页面允许加载和执行的脚本来源。

示例：

```http
Content-Security-Policy: default-src 'self'; script-src 'self'
```

作用：

```text
限制外部脚本加载
减少内联脚本执行
降低 XSS 利用成功率
```

注意：

```text
CSP 是防御增强措施，不应替代输出编码。
```

---

## 十三、代码审计中的 XSS 判断

代码审计时可以按照：

```text
Source → 数据流 → Sink
```

### 13.1 Source 点

常见用户输入：

```java
@RequestParam
@PathVariable
@RequestBody
request.getParameter()
request.getHeader()
request.getCookies()
```

前端 Source：

```javascript
location.href
location.search
location.hash
document.URL
localStorage
sessionStorage
```

---

### 13.2 Sink 点

后端输出 Sink：

```java
response.getWriter().write()
model.addAttribute()
return "view"
```

前端危险 Sink：

```javascript
innerHTML
outerHTML
document.write()
insertAdjacentHTML()
eval()
setTimeout()
```

---

### 13.3 Java 危险示例

```java
@GetMapping("/msg")
public void msg(String content, HttpServletResponse response) throws IOException {
    response.getWriter().write("<div>" + content + "</div>");
}
```

分析：

```text
Source：content 参数
Sink：response.getWriter().write()
问题：用户输入直接拼接到 HTML 输出
结论：疑似反射型 XSS
```

---

### 13.4 前端危险示例

```javascript
const msg = new URLSearchParams(location.search).get("msg");
document.getElementById("result").innerHTML = msg;
```

分析：

```text
Source：location.search
Sink：innerHTML
问题：URL 参数直接写入 DOM
结论：疑似 DOM 型 XSS
```

---

## 十四、XSS 学习靶场推荐

适合练习 XSS 的靶场：

```text
DVWA
Pikachu
WebGoat
PortSwigger Web Security Academy
XSS-Labs
CTFHub
BUUOJ Web 题
攻防世界 Web 题
```

建议练习顺序：

```text
1. 反射型 XSS
2. 存储型 XSS
3. DOM 型 XSS
4. HTML 标签上下文
5. HTML 属性上下文
6. JavaScript 字符串上下文
7. URL 上下文
8. 富文本过滤绕过
9. CSP 防护场景
10. 代码审计定位 XSS
```

---

## 十五、总结

XSS 判断核心：

```text
用户输入是否被输出到页面
输出位置是否会被浏览器解析
是否进行了正确的上下文编码
是否存在危险 DOM 操作
```

常见触发场景：

```text
搜索框
评论区
留言板
昵称
个人简介
后台富文本
错误提示
URL 参数
前端路由
文件名展示
```

主要危害：

```text
盗取会话
冒充用户操作
钓鱼
页面篡改
后台权限滥用
敏感信息泄露
```

修复核心：

```text
上下文相关输出编码
富文本白名单过滤
避免危险前端 API
Cookie 设置 HttpOnly / Secure / SameSite
配置 CSP
输入校验作为辅助
```

一句话总结：

```text
XSS 的本质是用户输入进入页面后被浏览器当作脚本执行；防护关键是让用户输入永远作为普通文本展示，而不是作为 HTML 或 JavaScript 执行。
```
