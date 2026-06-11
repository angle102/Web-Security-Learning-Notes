# Penetration Testing Process


```text
Web-Security-Learning-Notes/Penetration-Testing/渗透测试完整流程笔记.md
```

# 渗透测试完整流程笔记

> 本笔记仅用于授权渗透测试、靶场练习、CTF、安全竞赛和安全学习。
> 未经授权不得对真实网站、服务器或内网系统进行扫描、测试、爆破、攻击或数据访问。

---

## 目录

* [一、渗透测试前提](#一渗透测试前提)
* [二、整体流程](#二整体流程)
* [三、第一阶段：授权与范围确认](#三第一阶段授权与范围确认)
* [四、第二阶段：信息收集](#四第二阶段信息收集)
* [五、第三阶段：扫描与枚举](#五第三阶段扫描与枚举)
* [六、第四阶段：Web 漏洞测试](#六第四阶段web-漏洞测试)
* [七、第五阶段：登录框测试](#七第五阶段登录框测试)
* [八、第六阶段：文件上传测试](#八第六阶段文件上传测试)
* [九、第七阶段：API 接口测试](#九第七阶段api-接口测试)
* [十、第八阶段：权限获取后的安全验证](#十第八阶段权限获取后的安全验证)
* [十一、第九阶段：内网渗透基础流程](#十一第九阶段内网渗透基础流程)
* [十二、第十阶段：代码审计辅助分析](#十二第十阶段代码审计辅助分析)
* [十三、第十一阶段：日志与痕迹分析](#十三第十一阶段日志与痕迹分析)
* [十四、第十二阶段：报告编写](#十四第十二阶段报告编写)
* [十五、面试回答模板](#十五面试回答模板)

---

# 一、渗透测试前提

渗透测试必须建立在合法授权的基础上。

测试前需要明确：

```text
1. 测试目标
2. 授权范围
3. 测试时间
4. 允许测试的系统
5. 禁止测试的系统
6. 是否允许端口扫描
7. 是否允许弱口令验证
8. 是否允许上传测试
9. 是否允许内网探测
10. 是否允许权限提升验证
11. 是否允许读取少量脱敏数据作为证明
12. 报告交付形式
```

必须遵守原则：

```text
不破坏数据
不拖库
不清除日志
不影响业务
不进行未授权测试
敏感信息脱敏
只做最小化漏洞证明
```

---

# 二、整体流程

标准渗透测试流程可以分为：

```text
1. 授权与范围确认
2. 信息收集
3. 扫描与枚举
4. Web 漏洞测试
5. 登录认证测试
6. 文件上传测试
7. API 接口测试
8. 权限获取后的安全验证
9. 内网渗透基础分析
10. 代码审计辅助分析
11. 日志与痕迹分析
12. 报告编写与复测
```

整体思路：

```text
信息收集 → 资产发现 → 服务识别 → 入口定位 → 漏洞验证 → 权限分析 → 风险证明 → 修复建议
```

---

# 三、第一阶段：授权与范围确认

## 3.1 明确目标

示例：

```text
目标域名：example.com
目标 IP：192.168.1.100
测试范围：www.example.com、api.example.com、admin.example.com
测试类型：Web 渗透测试 / 内网渗透测试 / 代码审计 / 应急排查
```

## 3.2 明确禁止事项

例如：

```text
禁止大流量扫描
禁止 DoS 测试
禁止真实数据导出
禁止删除、修改业务数据
禁止清理日志
禁止测试非授权子域名
禁止访问核心生产数据库
```

## 3.3 测试记录模板

```text
项目名称：
测试人员：
测试时间：
授权联系人：
目标范围：
测试账号：
测试 IP：
禁止事项：
交付物：
```

---

# 四、第二阶段：信息收集

信息收集的目标是了解目标的外部暴露面，包括域名、子域名、IP、端口、技术栈、后台入口、接口、敏感文件和历史资产。

---

## 4.1 域名基础信息

常用命令：

```bash
whois example.com
```

```bash
dig example.com
dig example.com A
dig example.com MX
dig example.com NS
dig example.com TXT
```

```bash
nslookup example.com
```

重点关注：

```text
A 记录：主站 IP
MX 记录：邮件服务器
NS 记录：DNS 服务商
TXT 记录：SPF、DKIM、第三方验证信息
CNAME：是否接入 CDN 或云服务
```

---

## 4.2 CDN 判断

判断方式：

```bash
ping example.com
nslookup example.com
```

如果不同地区解析出不同 IP，可能存在 CDN。

CDN 场景下要关注：

```text
历史解析 IP
子域名真实 IP
邮件服务器 IP
测试环境 IP
接口服务器 IP
旧系统 IP
```

---

## 4.3 子域名收集

常用工具：

```bash
subfinder -d example.com -o subfinder.txt
```

```bash
amass enum -passive -d example.com -o amass.txt
```

```bash
assetfinder --subs-only example.com > assetfinder.txt
```

合并去重：

```bash
cat subfinder.txt amass.txt assetfinder.txt | sort -u > subdomains.txt
```

重点关注子域名：

```text
www.example.com
api.example.com
admin.example.com
test.example.com
dev.example.com
oa.example.com
vpn.example.com
mail.example.com
git.example.com
jenkins.example.com
```

高价值子域名通常包括：

```text
admin：后台
api：接口
test：测试环境
dev：开发环境
oa：办公系统
vpn：远程接入
git：代码仓库
jenkins：持续集成
mail：邮件系统
```

---

## 4.4 证书透明度收集

可以通过证书透明度记录发现历史子域名。

常见平台：

```text
crt.sh
Censys
FOFA
Hunter
ZoomEye
```

搜索格式：

```text
%.example.com
```

价值：

```text
发现历史子域名
发现测试环境
发现旧系统
发现内部命名规则
```

---

## 4.5 搜索引擎语法

常用搜索语法：

```text
site:example.com
site:example.com admin
site:example.com login
site:example.com filetype:pdf
site:example.com filetype:xls
site:example.com inurl:admin
site:example.com inurl:login
site:example.com inurl:api
```

搜索敏感信息：

```text
site:example.com password
site:example.com username
site:example.com "后台"
site:example.com "管理系统"
site:example.com "数据库"
```

价值：

```text
发现后台入口
发现泄露文档
发现员工邮箱
发现接口说明
发现历史页面
发现敏感文件
```

---

## 4.6 GitHub / 代码平台泄露检查

搜索关键词：

```text
example.com
"example.com" "password"
"example.com" "api_key"
"example.com" "secret"
"example.com" "access_token"
```

重点关注：

```text
数据库连接字符串
AK/SK
接口密钥
JWT secret
测试账号
内部接口地址
CI/CD 配置
```

---

## 4.7 favicon 与 robots.txt

### favicon

路径：

```text
/favicon.ico
```

作用：

```text
网站标签页小图标
可用于辅助识别 CMS、框架、后台系统
```

价值：

```text
识别 WordPress、Jenkins、phpMyAdmin、宝塔、Tomcat 等系统指纹
判断多个子域是否属于同一套系统
```

### robots.txt

路径：

```text
/robots.txt
```

示例：

```text
User-agent: *
Disallow: /admin/
Disallow: /backup/
Disallow: /test/
```

价值：

```text
发现后台路径
发现备份目录
发现测试目录
发现隐藏接口
```

---

# 五、第三阶段：扫描与枚举

扫描枚举的目标是确认哪些资产真实存活、开放哪些端口、运行哪些服务、存在什么目录和接口。

---

## 5.1 子域名存活探测

使用 httpx：

```bash
httpx -l subdomains.txt -title -status-code -tech-detect -o alive.txt
```

重点看：

```text
状态码
页面标题
技术栈
跳转地址
响应长度
登录页
后台页
403 / 401 / 500
```

状态码含义：

| 状态码         | 含义    | 渗透价值            |
| ----------- | ----- | --------------- |
| 200         | 正常访问  | 页面或接口存在         |
| 301/302     | 跳转    | 可能是登录页、后台、SSO   |
| 401         | 需要认证  | 资源存在，需要身份认证     |
| 403         | 禁止访问  | 目录或文件可能真实存在     |
| 404         | 不存在   | 用于判断伪 404       |
| 500         | 服务器异常 | 可能暴露框架、路径、数据库错误 |
| 502/503/504 | 网关异常  | 可能存在后端服务或反向代理   |

---

## 5.2 IP 提取与去重

```bash
dnsx -l subdomains.txt -a -resp-only -o ips.txt
sort -u ips.txt > unique_ips.txt
```

目的：

```text
整理真实 IP
发现同一服务器承载多个子域
发现源站
识别云主机或内网地址
```

---

## 5.3 端口扫描

基础扫描：

```bash
nmap -sS -Pn -T3 example.com
```

服务识别：

```bash
nmap -sV -sC -Pn example.com
```

指定端口：

```bash
nmap -sV -sC -Pn -p 80,443,8080,8443 example.com
```

常见端口价值：

| 端口        | 服务            | 价值               |
| --------- | ------------- | ---------------- |
| 21        | FTP           | 文件服务、匿名访问        |
| 22        | SSH           | 远程登录             |
| 80/443    | HTTP/HTTPS    | Web 服务           |
| 3306      | MySQL         | 数据库              |
| 5432      | PostgreSQL    | 数据库              |
| 6379      | Redis         | 缓存、Session、Token |
| 8080/8443 | Web 服务        | 后台、中间件           |
| 8888      | 宝塔面板          | 运维管理             |
| 9200      | Elasticsearch | 日志、数据检索          |
| 27017     | MongoDB       | 数据库              |

---

## 5.4 Web 指纹识别

常用方式：

```bash
whatweb https://example.com
```

```bash
curl -I https://example.com
```

关注：

```text
Server：nginx / Apache / IIS
X-Powered-By：PHP / ASP.NET / Express
Cookie：JSESSIONID / PHPSESSID / Laravel_session
框架：Spring Boot / ThinkPHP / Laravel / Django / Flask
CMS：WordPress / Discuz / Drupal / Dedecms
WAF：Cloudflare / 阿里云盾 / 腾讯云 WAF
```

---

## 5.5 目录扫描

常用工具：

```text
dirsearch
ffuf
gobuster
dirb
```

dirsearch 示例：

```bash
dirsearch -u https://example.com -e php,html,js,json,txt,bak,zip -t 5
```

低速扫描：

```bash
dirsearch -u https://example.com -e php,js,json,txt -t 2 --delay 1
```

ffuf 示例：

```bash
ffuf -u https://example.com/FUZZ -w wordlist.txt
```

带后缀：

```bash
ffuf -u https://example.com/FUZZ -w wordlist.txt -e .php,.jsp,.html,.bak,.zip
```

目录扫描重点路径：

```text
/admin
/login
/backend
/manage
/console
/api
/api/v1
/swagger
/swagger-ui.html
/v2/api-docs
/openapi.json
/actuator
/upload
/uploads
/static
/assets
/backup
/backup.zip
/www.zip
/db.sql
/.git/
/.env
/config.php.bak
/WEB-INF/web.xml
```

---

## 5.6 高价值路径含义

| 路径                 | 可能内容           | 价值                  |
| ------------------ | -------------- | ------------------- |
| `/admin`           | 后台管理入口         | 登录、弱口令、越权           |
| `/login`           | 登录页面           | 认证测试                |
| `/backend`         | 后台管理目录         | 后台入口                |
| `/manage`          | 管理系统           | 用户、权限、配置            |
| `/console`         | 控制台            | 运维、中间件、管理端          |
| `/api`             | 接口根路径          | API 测试              |
| `/api/v1`          | 第一版接口          | 旧接口可能存在权限缺陷         |
| `/swagger`         | Swagger 文档     | 暴露接口和参数             |
| `/swagger-ui.html` | Swagger UI 页面  | API 文档              |
| `/v2/api-docs`     | Swagger JSON   | 接口路径、参数             |
| `/openapi.json`    | OpenAPI 文档     | 接口说明                |
| `/actuator`        | Spring Boot 监控 | 配置、环境、接口泄露          |
| `/upload`          | 上传目录           | 文件上传风险              |
| `/uploads`         | 上传文件目录         | 用户文件泄露              |
| `/static`          | 静态资源           | JS、CSS、接口地址         |
| `/assets`          | 前端资源           | JS、隐藏路由             |
| `/backup`          | 备份目录           | 源码、数据库备份            |
| `/backup.zip`      | 备份压缩包          | 源码泄露                |
| `/www.zip`         | 网站源码包          | 配置、源码               |
| `/db.sql`          | 数据库导出          | 用户、管理员、密码哈希         |
| `/.git/`           | Git 目录         | 源码、提交历史             |
| `/.env`            | 环境变量           | 密码、密钥、数据库           |
| `/config.php.bak`  | PHP 配置备份       | 数据库账号密码             |
| `/WEB-INF/web.xml` | Java Web 配置    | Servlet、Filter、权限规则 |

---

# 六、第四阶段：Web 漏洞测试

---

## 6.1 SQL 注入

常见参数：

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
```

初步判断：

```text
参数加单引号是否报错
and 1=1 与 and 1=2 页面是否不同
响应时间是否异常
是否出现 SQL 错误
```

常见风险位置：

```text
登录框
搜索框
详情页
列表页
订单查询
后台筛选
API 参数
```

修复建议：

```text
使用预编译
禁止字符串拼接 SQL
MyBatis 使用 #{}
避免使用 ${}
参数白名单校验
数据库最小权限
```

---

## 6.2 XSS

测试位置：

```text
搜索框
留言板
昵称
个人资料
URL 参数
错误提示
后台富文本
```

类型：

```text
反射型 XSS
存储型 XSS
DOM 型 XSS
```

审计点：

```text
用户输入是否回显
是否做 HTML 编码
富文本是否过滤危险标签
Cookie 是否设置 HttpOnly
是否配置 CSP
```

修复建议：

```text
输出编码
输入过滤
富文本白名单
设置 HttpOnly
设置 CSP
```

---

## 6.3 文件读取 / 路径穿越

常见参数：

```text
file
path
download
filename
template
page
```

风险：

```text
读取配置文件
读取日志
读取源码
读取系统文件
下载非授权文件
```

安全写法思路：

```java
Path base = Paths.get("/data/files").toRealPath();
Path target = base.resolve(filename).normalize();

if (!target.startsWith(base)) {
    throw new SecurityException("Invalid path");
}
```

核心逻辑：

```text
先定义允许访问的基础目录
再拼接用户传入文件名
再规范化路径
最后判断最终路径是否仍在基础目录下
```

---

## 6.4 SSRF

常见参数：

```text
url
link
target
callback
redirect
image
avatar
webhook
```

常见 Sink：

```text
服务端请求用户指定 URL
远程图片抓取
Webhook 回调
文件下载
接口代理
```

风险：

```text
访问内网服务
访问本机服务
访问云元数据
探测内网端口
读取内部接口返回
```

修复建议：

```text
限制协议为 http/https
使用域名白名单
禁止访问 localhost、127.0.0.1、内网 IP
禁止访问云元数据地址
禁止重定向到内网
DNS 解析后校验 IP
```

---

## 6.5 命令执行

常见功能：

```text
ping
nslookup
traceroute
图片处理
文件转换
压缩解压
日志分析
```

危险函数：

```text
Runtime.getRuntime().exec()
ProcessBuilder()
system()
exec()
shell_exec()
```

修复建议：

```text
避免系统命令拼接
使用语言原生 API 替代命令
参数白名单校验
最小权限运行服务
限制可执行命令范围
```

---

## 6.6 认证绕过

常见问题：

```text
前端判断登录状态，后端不校验
修改返回包即可进入后台
直接访问后台接口不需要登录
空密码登录
只校验用户名不校验密码
Token 校验不完整
```

检查点：

```text
直接访问 /admin
直接调用 /api/admin
删除 Cookie 后访问接口
修改返回包字段
修改 role / isAdmin 参数
```

---

## 6.7 越权漏洞

类型：

```text
水平越权：用户 A 访问用户 B 的数据
垂直越权：普通用户访问管理员功能
未授权访问：不登录也能访问接口
```

常见接口：

```text
/api/user/info?id=1001
/api/order/detail?id=2001
/api/admin/user/list
/api/file/download?id=3001
```

修复建议：

```text
后端校验当前用户身份
后端校验资源归属
管理员接口强制权限校验
不要只依赖前端隐藏按钮
不要信任前端传入的 userId、role
```

---

# 七、第五阶段：登录框测试

即使只有一个登录框，也可能存在很多漏洞。

---

## 7.1 弱口令 / 默认口令

关注：

```text
默认账号
默认密码
弱密码
测试账号
初始密码
```

常见系统：

```text
CMS
后台管理系统
Tomcat
Jenkins
宝塔
phpMyAdmin
运维平台
```

---

## 7.2 用户名枚举

判断方式：

```text
不存在用户：用户不存在
存在用户但密码错误：密码错误
```

如果错误提示不同，就可能泄露用户名是否存在。

其他判断点：

```text
响应长度不同
响应状态码不同
响应时间不同
验证码触发策略不同
```

安全建议：

```text
统一提示：用户名或密码错误
限制失败次数
增加登录告警
```

---

## 7.3 暴力破解风险

检查：

```text
是否有验证码
是否限制错误次数
是否锁定账号
是否限制 IP
是否限制设备
是否有 MFA
是否有登录告警
```

MFA：Multi-Factor Authentication，多因素认证，例如短信验证码、邮箱验证码、动态令牌、指纹、人脸、企业微信确认等。

---

## 7.4 验证码绕过

常见问题：

```text
验证码只在前端校验
验证码可以复用
验证码不刷新
验证码为空也能登录
验证码和 Session 未绑定
验证码可预测
验证码接口无频率限制
```

测试思路：

```text
删除验证码参数
验证码填错
验证码重复使用
更换账号复用验证码
更换 Session 复用验证码
```

---

## 7.5 SQL 注入

登录框参数：

```text
username
password
account
mobile
email
```

如果后端拼接 SQL，可能存在注入。

修复建议：

```text
预编译 SQL
密码哈希校验
错误信息统一
数据库最小权限
```

---

## 7.6 XSS

登录失败时如果回显用户名，可能存在反射型 XSS。

常见位置：

```text
用户名
错误提示
redirect
returnUrl
msg
```

---

## 7.7 Session / Cookie 安全

登录成功后检查 Cookie：

```text
SessionID 登录前后是否变化
Cookie 是否设置 HttpOnly
Cookie 是否设置 Secure
Cookie 是否设置 SameSite
退出登录后 Session 是否失效
修改密码后旧 Session 是否失效
```

---

## 7.8 JWT 安全

如果登录后返回 JWT，重点看：

```text
是否有过期时间
secret 是否弱
是否允许 alg=none
是否只在前端判断 role
是否能修改 userId / role
退出登录后 Token 是否仍有效
```

---

## 7.9 开放重定向

常见参数：

```text
redirect
returnUrl
callback
next
url
```

风险：

```text
钓鱼
SSO 绕过辅助
Token 泄露
```

---

# 八、第六阶段：文件上传测试

文件上传常见于：

```text
头像上传
附件上传
工单上传
图片上传
富文本编辑器上传
后台模板上传
```

重点检查：

```text
是否限制后缀
是否校验 MIME
是否校验文件内容
是否重命名
是否保存到 Web 可执行目录
是否允许访问上传路径
是否限制文件大小
是否存在路径穿越
```

常见富文本编辑器：

```text
UEditor
KindEditor
CKEditor
TinyMCE
wangEditor
Quill
```

富文本风险：

```text
XSS
任意文件上传
SSRF
权限绕过
```

修复建议：

```text
白名单后缀
校验文件魔数
随机文件名
保存到非 Web 执行目录
禁止脚本类型
限制大小
上传目录禁止执行权限
```

---

# 九、第七阶段：API 接口测试

API 接口常见路径：

```text
/api
/api/v1
/api/v2
/swagger
/swagger-ui.html
/v2/api-docs
/openapi.json
/graphql
```

重点检查：

```text
未授权访问
水平越权
垂直越权
敏感信息泄露
接口参数可控
批量导出
任意文件下载
接口文档暴露
调试接口暴露
```

Swagger 泄露可能暴露：

```text
接口路径
请求方法
参数名
参数类型
认证方式
返回字段
后台接口
测试接口
```

GraphQL 重点关注：

```text
是否允许 introspection
是否能查询敏感字段
是否存在越权
是否存在复杂查询导致资源消耗
```

---

# 十、第八阶段：权限获取后的安全验证

如果在授权范围内获得了后台权限、WebShell、服务器普通权限或数据库权限，需要做最小化证明。

---

## 10.1 Linux 基础信息收集

```bash
whoami
id
hostname
uname -a
ip addr
ip route
arp -a
ss -ntlp
ps aux
```

命令含义：

| 命令         | 作用             |
| ---------- | -------------- |
| `whoami`   | 查看当前用户         |
| `id`       | 查看 UID、GID、用户组 |
| `hostname` | 查看主机名          |
| `uname -a` | 查看系统和内核版本      |
| `ip addr`  | 查看网卡和 IP       |
| `ip route` | 查看路由表          |
| `arp -a`   | 查看 ARP 缓存      |
| `ss -ntlp` | 查看监听端口和进程      |
| `ps aux`   | 查看运行进程         |

---

## 10.2 Windows 基础信息收集

```cmd
whoami
whoami /all
hostname
ipconfig /all
route print
arp -a
netstat -ano
tasklist
systeminfo
```

命令含义：

| 命令              | 作用           |
| --------------- | ------------ |
| `whoami`        | 查看当前用户       |
| `whoami /all`   | 查看用户权限和组     |
| `ipconfig /all` | 查看网卡、DNS、域信息 |
| `route print`   | 查看路由表        |
| `arp -a`        | 查看 ARP 缓存    |
| `netstat -ano`  | 查看网络连接       |
| `tasklist`      | 查看进程         |
| `systeminfo`    | 查看系统信息       |

---

## 10.3 配置文件排查

常见配置文件：

```text
application.yml
application.properties
.env
config.php
database.php
web.xml
settings.py
config.json
```

重点搜索字段：

```text
password
passwd
pwd
secret
token
apikey
accessKey
secretKey
jdbc
redis
mysql
ldap
mail
bucket
```

可能发现：

```text
数据库账号密码
Redis 密码
JWT secret
第三方接口 Key
OSS / S3 AK/SK
邮件服务器账号
内部接口地址
```

---

# 十一、第九阶段：内网渗透基础流程

内网渗透是指从一个入口点进入目标内网后，继续识别内网资产、账号凭据、权限关系和关键系统风险。

---

## 11.1 内网渗透前提

必须确认：

```text
授权网段
禁止测试系统
是否允许扫描
是否允许弱口令验证
是否允许横向移动
是否允许提权验证
是否允许域环境测试
```

---

## 11.2 初始入口

可能入口：

```text
WebShell
普通服务器权限
VPN 账号
办公主机权限
数据库账号
后台账号
堡垒机账号
```

---

## 11.3 本机信息收集

Linux：

```bash
whoami
id
hostname
uname -a
ip addr
ip route
arp -a
ss -ntlp
ps aux
```

Windows：

```cmd
whoami /all
hostname
ipconfig /all
route print
arp -a
netstat -ano
tasklist
systeminfo
```

目的：

```text
确认当前权限
确认主机角色
确认内网网段
确认开放服务
确认运行进程
确认是否在域环境
```

---

## 11.4 内网资产发现

高价值资产：

```text
域控
文件服务器
数据库服务器
Web 系统
运维系统
监控系统
代码仓库
Jenkins
GitLab
Nexus
Harbor
堡垒机
OA
邮件系统
VPN
财务系统
日志平台
```

常见端口：

| 端口        | 服务                | 价值           |
| --------- | ----------------- | ------------ |
| 53        | DNS               | 可能是域控或内网 DNS |
| 80/443    | Web               | 内网业务系统       |
| 445       | SMB               | Windows 文件共享 |
| 3389      | RDP               | 远程桌面         |
| 3306      | MySQL             | 数据库          |
| 5432      | PostgreSQL        | 数据库          |
| 6379      | Redis             | 缓存、Session   |
| 9200      | Elasticsearch     | 日志平台         |
| 27017     | MongoDB           | 数据库          |
| 8080/8081 | Web/Jenkins/Nexus | 运维系统         |
| 9090      | Prometheus        | 监控           |
| 5601      | Kibana            | 日志平台         |

---

## 11.5 凭据与权限分析

可能来源：

```text
配置文件
数据库连接字符串
脚本文件
历史命令
环境变量
浏览器保存密码
远程连接记录
共享目录
备份文件
应用日志
CI/CD 配置
```

Linux 常见位置：

```bash
history
env
cat ~/.bash_history
```

Windows 常见命令：

```cmd
set
cmdkey /list
net use
```

重点判断：

```text
密码是否复用
数据库密码是否可登录后台
本地管理员密码是否复用
域账号是否可访问其他主机
```

---

## 11.6 Windows 域环境基础

### 什么是域？

域是企业内网中统一管理用户、主机和权限的逻辑范围。

### 什么是域控？

域控是 Domain Controller，负责管理域用户、域主机、组策略和身份认证。

常见判断命令：

```cmd
whoami /fqdn
echo %USERDOMAIN%
nltest /domain_trusts
net config workstation
ipconfig /all
```

重点看：

```text
当前用户是否是域用户
登录域是什么
DNS 是否指向域控
是否存在域信任关系
工作站是否加入域
```

域环境核心资产：

```text
域控
域管理员
域用户
文件服务器
组策略
Kerberos 认证
```

---

# 十二、第十阶段：代码审计辅助分析

代码审计的核心是：

```text
Source → 数据流 → Sink
```

也就是：

```text
用户可控输入 → 中间业务逻辑 → 危险函数
```

---

## 12.1 Source 点

Source 是用户可控输入，例如：

```java
@RequestParam
@PathVariable
@RequestBody
request.getParameter()
request.getHeader()
request.getCookies()
request.getInputStream()
MultipartFile
```

---

## 12.2 Sink 点

Sink 是危险操作点，例如：

```text
SQL 执行
命令执行
文件读写
URL 请求
XML 解析
反序列化
页面输出
模板渲染
```

常见 Sink：

```java
Statement.executeQuery()
Runtime.getRuntime().exec()
ProcessBuilder.start()
new FileInputStream()
Files.readAllBytes()
new URL(url).openConnection()
ObjectInputStream.readObject()
DocumentBuilder.parse()
response.getWriter().write()
```

---

## 12.3 Java 项目重点文件

| 文件/目录                    | 作用             | 审计重点                   |
| ------------------------ | -------------- | ---------------------- |
| `pom.xml`                | Maven 依赖配置     | 第三方组件版本                |
| `build.gradle`           | Gradle 依赖配置    | 依赖版本                   |
| `application.yml`        | Spring Boot 配置 | 数据库、Redis、JWT、Actuator |
| `application.properties` | Spring Boot 配置 | 敏感配置                   |
| `web.xml`                | 传统 Java Web 配置 | Servlet、Filter、路径映射    |
| `Controller`             | 请求入口           | Source 点               |
| `Service`                | 业务逻辑           | 权限判断                   |
| `Mapper`                 | MyBatis SQL 层  | SQL 注入                 |
| `DAO`                    | 数据访问层          | SQL 拼接                 |
| `Filter`                 | 请求过滤器          | 登录校验                   |
| `Interceptor`            | Spring 拦截器     | 权限控制                   |
| `SecurityConfig`         | 安全配置           | 认证授权                   |

---

## 12.4 SQL 注入审计

危险写法：

```java
String sql = "select * from users where name = '" + name + "'";
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(sql);
```

MyBatis 危险写法：

```java
@Select("select * from user where name = '${name}'")
```

安全写法：

```java
@Select("select * from user where name = #{name}")
```

区别：

```text
${}：字符串拼接，容易注入
#{}：预编译参数，相对安全
```

---

## 12.5 反序列化审计

常见关键词：

```text
ObjectInputStream
readObject
readUnshared
Serializable
Externalizable
Fastjson
Jackson
XStream
Hessian
Shiro rememberMe
```

典型危险代码：

```java
ObjectInputStream ois = new ObjectInputStream(request.getInputStream());
Object obj = ois.readObject();
```

判断逻辑：

```text
用户可控数据是否进入 readObject
是否有类白名单
是否反序列化任意 Object
依赖中是否存在危险 gadget
```

---

## 12.6 Fastjson 反序列化

Fastjson 是 Java JSON 序列化和反序列化库。

常见方法：

```java
JSON.toJSONString(user);
JSON.parseObject(json);
```

重点搜索：

```text
com.alibaba.fastjson
JSON.parseObject
JSON.parse
parseArray
setAutoTypeSupport(true)
Feature.SupportAutoType
```

高风险组合：

```text
用户可控 JSON
+
JSON.parseObject
+
AutoType 开启
+
Object / Map / JSONObject 宽泛类型
```

---

## 12.7 XXE 漏洞审计

XXE 是 XML 外部实体注入漏洞。

判断逻辑：

```text
用户可控 XML
+
XML 解析器
+
没有禁用外部实体
=
疑似 XXE
```

常见 XML 解析器：

```text
DocumentBuilderFactory
SAXParserFactory
SAXReader
XMLInputFactory
TransformerFactory
SchemaFactory
```

危险代码：

```java
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = factory.newDocumentBuilder();
Document doc = builder.parse(request.getInputStream());
```

安全配置示例：

```java
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
factory.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
factory.setXIncludeAware(false);
factory.setExpandEntityReferences(false);
```

---

## 12.8 Tabby 与 CodeQL

### Tabby

Tabby 偏 Java 调用链分析，适合快速梳理：

```text
类
方法
调用关系
继承关系
Source 到 Sink 调用链
```

需要资源：

```text
jar
war
class
项目依赖
JDK 环境
配置文件
```

### CodeQL

CodeQL 是语义代码分析工具，可以把代码构建成数据库，再用 QL 查询漏洞模式。

需要资源：

```text
完整源码
pom.xml / build.gradle
可成功构建
JDK
Maven / Gradle
CodeQL CLI
查询规则
```

创建数据库示例：

```bash
codeql database create java-db --language=java --command="mvn clean package -DskipTests"
```

运行分析：

```bash
codeql database analyze java-db codeql/java-queries --format=csv --output=result.csv
```

---

# 十三、第十一阶段：日志与痕迹分析

渗透测试中不能清理日志，但需要记录测试行为并帮助甲方排查。

常见日志：

```text
Web 访问日志
应用日志
数据库日志
系统登录日志
中间件日志
安全设备告警
WAF 日志
```

Linux 常见日志：

```text
/var/log/auth.log
/var/log/secure
/var/log/messages
/var/log/nginx/access.log
/var/log/nginx/error.log
Tomcat logs
```

Windows 常见日志：

```text
安全日志
系统日志
应用程序日志
PowerShell 日志
远程登录日志
```

应提供给甲方：

```text
测试 IP
测试时间
测试路径
测试账号
触发漏洞的请求
风险说明
修复建议
```

---

# 十四、第十二阶段：报告编写

渗透测试报告应包括：

```text
1. 项目概述
2. 测试范围
3. 测试时间
4. 测试方法
5. 资产清单
6. 漏洞清单
7. 漏洞复现步骤
8. 影响分析
9. 风险等级
10. 修复建议
11. 复测结果
```

---

## 14.1 漏洞表模板

| 编号 | 漏洞类型   | 位置                  | 风险等级 | 影响      | 修复建议      |
| -- | ------ | ------------------- | ---- | ------- | --------- |
| 1  | 未授权访问  | `/api/user/list`    | 高    | 可查看用户信息 | 增加鉴权      |
| 2  | 敏感文件泄露 | `/.env`             | 高    | 泄露数据库密码 | 禁止访问敏感文件  |
| 3  | 弱口令    | `/admin`            | 高    | 后台被接管   | 强密码和 MFA  |
| 4  | 水平越权   | `/api/order/detail` | 高    | 越权查看订单  | 服务端校验资源归属 |

---

## 14.2 漏洞描述模板

```text
漏洞名称：
漏洞等级：
漏洞位置：
影响范围：
漏洞描述：
复现步骤：
证明截图：
影响分析：
修复建议：
复测结果：
```

---

## 14.3 修复建议常见方向

```text
加强身份认证
增加权限校验
使用预编译 SQL
关闭敏感接口
限制后台访问 IP
禁用目录浏览
删除备份文件
禁止敏感文件公网访问
升级存在漏洞的组件
配置 WAF / IDS / EDR
日志监控与告警
密码策略和 MFA
最小权限原则
```

---

# 十五、面试回答模板

如果面试官问：“给你一个目标，如何开展渗透测试？”

可以回答：

```text
在获得授权后，我会先确认测试范围、目标系统、允许操作和禁止事项。

第一步是信息收集，包括 Whois、DNS、子域名、证书透明度、搜索引擎语法、GitHub 泄露、robots.txt 和 favicon 指纹，目标是整理域名、子域名、IP、后台入口、接口和技术栈线索。

第二步是扫描枚举，我会对子域名做存活探测，记录状态码、标题、技术栈和跳转地址；再对 IP 做端口扫描和服务识别，确认开放端口、中间件和版本；随后对 Web 资产进行目录扫描、敏感文件枚举、JS 接口提取、API 文档探测和登录入口枚举。

第三步是漏洞验证，根据发现的入口测试 SQL 注入、XSS、文件上传、任意文件读取、SSRF、命令执行、认证绕过、越权、敏感文件泄露和组件配置问题。

如果拿到权限，我会进行最小化安全验证，例如确认当前权限、查看配置文件是否泄露数据库或 Redis 密码、判断是否存在内网访问风险，但不会破坏数据、不拖库、不清除日志。

如果授权包含内网，我会进一步做本机信息收集、内网资产识别、凭据分析、权限提升验证和横向移动风险评估，重点关注域控、数据库、代码仓库、运维平台、文件服务器和监控平台。

最后输出完整报告，包括资产清单、漏洞清单、复现步骤、风险影响、证据截图、修复建议和复测结果。
```

---

# 总结

渗透测试不是简单地“扫工具”或“打漏洞”，而是一个完整的方法论：

```text
授权边界明确
信息收集充分
资产梳理完整
漏洞验证克制
权限分析清晰
风险证明最小化
报告输出规范
修复建议可落地
```

核心能力：

```text
知道测什么
知道为什么测
知道怎么证明风险
知道如何避免影响业务
知道如何给出修复建议
```
